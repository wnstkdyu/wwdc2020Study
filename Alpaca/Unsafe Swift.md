# Unsafe Swift

Unsafe가 접두어로 붙은 타입들은 인터페이스로부터 어떠한 점이 'Unsafe'한 지 명확히 알 수 없다. 그보다는 해당 타입들을 구현 시에 유효하지 않은 입력 값이 들어왔을 때 깨닫는 경우가 많다.

옵셔널 타입을 Force Unwrapping하여 사용하는 경우, 해당 변수에 `nil` 이 들어오는 경우 Fatal error를 맞게 된다.

```swift
let value: Int? = nil
print(value!) // Fatal error: Unexpectedly found nil while unwrapping an Optional value
```

다만, 우리는 여기서 어떤 값이 들어올 때 에러가 나는지와 어떤 에러가 나는지 둘 다 알 수 있기 때문에 쉽게 고칠 수 있다.

따라서 오퍼레이션이 Safe하냐, Unsafe하냐는 다음과 같이 구분할 수 있다.

- **Safe** oprations have **well-defined** behaviour on **all** input
- **Unsafe** operations have **undefined** behaviour on **some** input

예를 들어 ! 대신 Unsafe하게 강제 언래핑을 하는 방법은  [`unsafelyUnwrapped`](https://developer.apple.com/documentation/swift/optional/1641793-unsafelyunwrapped) 를 사용하는 것이다. 이 프로퍼티는 접미사로 붙이는 !와 같이 해당 값이 nil이 아닐 것이라고 생각하고 사용할 수 있지만 차이점은 optimized builds (-0) 시에 입력 값이 실제로 있는 지에 대해 검사를 하지 않는다. 호출 시에 언제나 값이 있을 것이라고 믿고 값을 읽어내게 된다. 때문에 어떤 결과를 초래할 지 예상할 수 없다. 아무 의미없는 쓰레기 값을 반환할 수도, 바로 크래시를 유발할 수도, 또는 제 3의 결과를 일으킬 수 있다. 또한 이러한 결과는 에러 발생 때마다 달라질 것이다.

> 다만, debug builds (-0none) 시에는 똑같이 동작한다.

결과적으로 이 프로퍼티를 사용하는 것은 잠재적으로 제어할 수 없는 이슈를 내포하며 디버깅 또한 무척이나 어려울 것이다. 이 점이 Swift standard library에 속한 Unsafe 타입들의 핵심 속성이다.

## Unsafe 사용 시에는 주의를 기울이자.

때때로 작업중에는 Unsafe 타입을 사용해야만 하는 것들이 있다. 다만, 해당 타입을 사용할 때는 그 상황과 조건을 전부 이해하고 주의를 기울여 사용해야 한다.

#### Benefits of unsafe interfaces

- C나 Objective-C로 작성된 코드의 상호 호환성
- 런타임 성능에 대한 제어

알아두어야 할 중요한 점 하나는 Safe code가 No crashes를 의미하지 않는다는 것이다. Safe code는 기대와는 다른 입력값이 왔을 때 에러를 던져 이를 알릴 수 있는 코드(여기서는 API라고 말함.)이다. 따라서 Swift가 안전한 언어라고 말하는 것은 언어 차원에서 입력값을 검증하는 과정을 거치기 때문이고 이 과정을 거지치 않는 API들이 Unsafe 접두사를 달고 나오는 것이다.

### Unsafe Pointers

C 언어의 포인터를 추상화한 개념으로 Swift가 자동으로 처리해주는 메모리 관련 처리를 직접 할 수 있게 강력한 오퍼레이션을 제공한다. 그와 동시에 메모리 관련 처리를 직접 해줘야 한다는 책임 또한 같이 따라오게 된다.

다음의 코드를 보자.

```swift
let ptr = UnsafeMutablePointer<Int>.allocate(capacity: 1)
// 42로 초기화
ptr.initialize(to: 42)
print(ptr.pointee)
ptr.deallocate()
// 이미 dealloc된 메모리에 할당하려고 한다면 문제가 생기게 된다.
ptr.pointee = 23 // X
```

<img src="https://user-images.githubusercontent.com/22453984/91451500-71da9b80-e8b8-11ea-9409-1906bf333568.png" alt="스크린샷 2020-08-27 오후 10 55 42" style="zoom:50%;" />

위와 같이 UnsafePointer 타입을 사용한다는 것은 메모리 관리에 대한 책임까지 같이 한다는 의미이며 `ptr` 변수가 dealloc 되었음에도 코드 상으로는 해당 변수가 유효한 지에 대해 알 수가 없다. 따라서 크래시 발생할 수도 있고, `ptr` 이 위치하고 있던 메모리 위치에 다른 값이 저장되어 해당 값이 23으로 set되는 등 예측할 수 없는 결과가 일어나게 된다.

그렇다면 UnsafePointer를 사용하는 이유는 무엇일까?

기본적으로 Unsafe한 언어 타입인 C나 Objective-C와의 호환성 때문이다.

![스크린샷 2020-08-27 오후 11 09 47](https://user-images.githubusercontent.com/22453984/91453201-6a1bf680-e8ba-11ea-8adc-9358839a75cc.png)

```Swift
// C
void process_integers(const int *start, size_t count);

// Swift
func process_integer s(_ start: UnsafePointer<CInt>!, _ count: Int) { ... }

// 1) start 버퍼를 만듦
let start = UnsafeMutablePointer<CInt>.allocate(capacity: 4)
// 2) 버퍼 안의 값을 CInt 값으로 초기화
start.initialize(to: 0)
(start + 1).initialize(to: 2)
(start + 2).initialize(to: 4)
(start + 3).initialize(to: 6)

// 3)
process_integers(start, 4)

// 4)
start.deinitialize(count: 4)
// 5)
start.deallocate()
```

위의 코드와 같이 UnsafePointer를 통해 C의 함수를 사용할 수 있지만 각 단계는 기본적으로 Unsafe함을 내포하고 있다.

1) start 변수에 UnsafePointer를 담아두었다고 해서 start 변수가 해당 포인터의 메모리를 관리하는 것은 아니다. 개발자가 직접 코드 상으로 적절한 타이밍에 deinit 및 dealloc을 해줘야 한다. 그렇지 않을 경우 메모리 상에서 계속 남아 메모리 릭을 유발할 것.

2) 초기화 과정 자체가 해당 값들의 주소가 버퍼 안에 있다는 것을 증명해주지는 않는다.

3) 메서드의 반환값으로 포인터를 주는지, 또 그 값을 참조로 잡고 있을지에 대해 함수 문서를 통해 알아야 한다.

4) Deinit 과정은 언제나 초기화 과정이 선행되고 난 후에 이루어져야 한다.

5) dealloc 과정은 메모리 할당이 되었으며 deinit 상태에 들어간 메모리에 대해서만 이루어져야 한다.

이러한 모든 과정 각각은 과정에 맞는 가정들을 기반으로 하고 이루어지고 있으며 가정이 틀렸을 경우 예상치 못한 결과를 초래할 수도 있다.

### UnsafeBufferPointer

위의 코드에서는 포인터의 시작점만을 가지고 포인터를 관리하고 있다. 하지만 실제로 해당 포인터에 들어가는 값의 형태는 버퍼 형태이기 때문에 length를 통해 버퍼의 범위를 명시해주는 것은 관리 차원에서 크게 이득일 것이다.

이것이 Swift 표준 라이브러리에서 BufferPointer를 제공해주는 이유이다.

## 그 밖의 Swift 표준 라이브러리가 제공해주는 것

- BufferPointer를 통해 Collection에서 메모리에 접근할 수 있는 기회 제공
- 좀 더 작은 크기에서 Unsafe한 오퍼레이션을 관리하도록 임시 포인터를 클로저의 형태로 제공
  - 아래와 같이 메모리 관리에 대한 부담을 줄일 수 있음

```swift
// C
void process_integers(const int *start, size_t count);

// Swift
let values: [CInt] = [0, 2, 4, 6]
values.withUnsafeBufferPointer { buffer in
	print_integers(buffer.baseAddress!, buffer.count)
}
// 컴파일러가 자동으로 해당 Array에 대한 임시 버퍼 포인터를 생성하여 전달해주기도 함.
print_integers(values, values.count)
```

아래 그림과 같이 Swift의 표준 타입을 전달하면 컴파일러가 자동으로 임시 포인터를 생성하여 C 언어의 함수로 전달해준다.

![스크린샷 2020-08-27 오후 11 36 52](https://user-images.githubusercontent.com/22453984/91456411-317e1c00-e8be-11ea-95f6-b528e862b6d5.png)

중요한 것은 **클로저로 받든, 암시적으로 컴파일러가 전달하든 UnsafePointer를 전달하는 메서드 호출 시점에만 해당 포인터가 유효**하다는 점이다. 때문에 영상에서는 클로저로 포인터의 생명 주기를 명확히 알 수 있기 때문에 클로저 타입의 API를 사용하기를 좀 더 권장하고 있다.

![스크린샷 2020-08-27 오후 11 45 15](https://user-images.githubusercontent.com/22453984/91457490-5d4dd180-e8bf-11ea-8622-cf4dd824b36d.png)

## 결론

- 각각의 Unsafe 인터페이스 요구사항을 따르라
- Unsafe API의 사용을 고립화해라 => 클로저 타입의 API처럼...!
- 여러 값을 다루는 버퍼를 포인터로 다룰 경우, BufferPointer 타입을 사용하여 바운더리를 통제하자.
- Sanitizer를 통해 테스트하자.