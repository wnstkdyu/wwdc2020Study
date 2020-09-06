# Safely manage pointers in Swift

WWDC 2020의 [Unsafe Swift](https://developer.apple.com/videos/play/wwdc2020/10648/) 에서 이어지는 내용으로 Unsafe한 오퍼레이션을 어떻게 다뤄야 할 지에 대해 다룬다. 대부분의 시간을 **Pointer Type Safety** 에 관해 다루는데 이는 

## Pointer Type Safety

대부분의 시간을 이 주제에 대해 다룰 것이며 C로 된 API와 같은 기능을 하지만 Swift로 써진 API를 잘못 다룰 때 발생하는 이슈에 관한 것으로 어떻게 안전하게 해당 오퍼레이션을 사용할 수 있는 지에 대해 다룬다.

### Levels of safety

Pointer Type Safety는 Safety 별로 몇 가지 레벨로 나눌 수 있으며 레벨이 낮아질수록 Safety에 관해  개발자가 더 많은 책임을 지니게 된다.

- **Safe**
  - Swift에서 제공하는 기본적인 API들로 Unsafe한 오퍼레이션을 사용하지 않기 때문에 Unsafe한 요소들을 걱정하지 않아도 됨.
- **Unsafe**
  - Safe함을 포기하면서도 성능 최적화를 위해 또는 사용해야만 하는 경우 Unsafe 오퍼레이션을 사용하게 된다.
  - Unsafe 접두사를 붙인 `UnsafePointer<T>` 가 이에 해당되며 이 경우 **Type Safety에 관해서는 걱정하지 않아도** 된다.
- **Raw**
  - Byte 별 접근이 필요한 경우 사용하는 API로 `UnsafeRawPointer` 가 이에 해당된다.
  - 해당 Byte가 어떤 타입에 해당되는 지는 포인터가 모르기 때문에 개발자가 직접 신경을 써줘야 한다. 즉, Type Safety가  보장되지 않는다.
- **Mutable type**
  - 가장 깊은 단에 있는 Memory-binding API로 타입에 메모리를 할당할 때 사용하는 API.
  - 포인터 타입을 안전하게 다루기 위한 모든 책임을 Swift가 아닌 개발자가 지게 된다.

### Safe vs Unsafe Swift Code

Safe한 코드는 예측 가능한 행동을 보여준다. 컴파일 시에 에러가 날 경우 IDE에서 에러를 알려주고 런타임 시에 에러가 날 경우 크래시를 발생시켜 에러가 났음을 알려준다.

반면, Unsafe한 코드의 경우 행동이 예측 가능하지 않기 때문에 코드 동작에 대한 책임을 개발자가 직접 챙겨야 한다.

물론 테스트 케이스를 작성하고 Sanitizer를 사용하는 하여 에러를 발견하고 해결하는 데 도움을 주지만 런타임 시에 발생할 수 있는 모든 에러를 방지해주지는 못한다. 크래시는 사용자에게 분명 나쁜 경험이지만 예측할 수  없는 행동은 더 나쁠 수 있다. 예를 들어 고객의 데이터가 잘못된 정보로 오염될 수도 있다.

때문에 Unsafe한 API를 사용 시에는 여러 측면을 고려해야 한다.

#### Unsafe한 여러 상황

- 객체가 할당된 메모리는 이미 유효하지 않은데 포인터 객체는 살아 있어 해당 메모리에 접근하는 경우. 이 예는 포인터 사용 시에 Unsafe한 측면의 대부분을 차지함.

<img width="637" alt="스크린샷 2020-09-06 오후 3 31 52" src="https://user-images.githubusercontent.com/22453984/92319705-23669300-f056-11ea-9136-b5e4d804ab12.png" style="zoom: 67%;" >

- 포인터가 Buffer로 된 객체를 가리키고 있을 때 Buffer 범위 밖을 가리키게 되는 경우

<img width="642" alt="스크린샷 2020-09-06 오후 3 34 42" src="https://user-images.githubusercontent.com/22453984/92319751-7dffef00-f056-11ea-9ade-548cc5410e0a.png" style="zoom:67%;" >

이런 상황들도 있지만 이번 세션에서는 쉽게 간과할 수 있는 상황에 대해 집중해보고자 한다.

### Pointer types

포인터는 자신만의 타입을 갖고 있으며 이는 메모리에 할당된 타입과는 구분된다. 이 두 타입이 일치하는지 어떻게 알  수 있을까? 또  일치하지 않을 경우에는 어떻게 될까?

```swift
struct Image {
    let url: String
}

struct Collage {
    var imageData: UnsafeMutablePointer<Image>?
    // Int 타입으로 초기화
    var imageCount: Int = 0
}

func addImages(_ countPtr: UnsafeMutablePointer<UInt32>) -> UnsafeMutablePointer<Image> {
    let images = [Image(url: "첫번째 이미지"), Image(url: "두번째 이미지"), Image(url: "세번째 이미지")]
    let imageData = UnsafeMutablePointer<Image>.allocate(capacity: images.count)
    imageData.initialize(from: images, count: images.count)
    
    // UInt32 타입으로 쓰기 작업 수행
    countPtr.pointee += 3
    return imageData
}

func saveImages(_ imageData: UnsafeMutablePointer<Image>, _ count: Int) {
    print("imageData: \(imageData), count: \(count)")
}

var collage = Collage()
collage.imageData = withUnsafeMutablePointer(to: &collage.imageCount) {
    addImages(UnsafeMutableRawPointer($0).assumingMemoryBound(to: UInt32.self))
}
// Int 타입으로 읽기
saveImages(collage.imageData!, collage.imageCount)
```

위의 경우 객체 초기화 시의 타입과 해당 객체에 접근하는 포인터가 알고  있는 타입이 다르다. 때문에 `imageCount`를 pointee를 통해 갯수를 올렸어도 컴파일러는 원래 객체의 프로퍼티가 변경되지 않은 것으로 인식하여 0으로 읽을 수 있다.

실제로 위의 코드를 실행해보면 타입이 불일치함에도 불구하고 컴파일러가 알아서 쓰기 작업을 수행해 준다. 하지만 중요한 것은 우리가 항상 결과를 예측할 수 없다는 것이다. 컴파일러는 타입 정보를 통해 가정을 세우고 동작하기 때문에 잘못된 가정을 세울 경우 예측하지 못한 결과를 만들어낼 수도 있다.

#### Pointer type bugs

포인터 타입으로 일어날 수 있는 버그는 크게 두 가지로 나눌 수 있다.

- 의도치 않았던 행동을 일으킬 수도 있음
- 오랫동안 숨겨져 감춰져 있을 수도 있음

- 하지만 어느 순간 갑자기 드러날 수가 있음
  - 보다 안전한 코드로의 변경을 통해
  - 컴파일러 업데이트를 통해

#### Pointer type rules for Swift and C

C 언어는 포인터 타입을 안전하게 사용하기 위해 "strict aliasing"과 "type punning"을 지키기 위한 규칙을 가지고 있다.

> type-punning이란?
>
> type punning은 어떤 변수가 있을 때, 그 변수의 실제 값을 무시하고 형 변환 없이 다른 자료형으로 변환하는 걸 말합니다. 변환된 값은 메모리 상에서 원래 값과 똑같은 비트열로 표현되게 됩니다.
>
> https://kldp.org/node/56205

하지만 Swift의 포인터를 사용할 경우 해당 규칙을 몰라도 되며 안전하게 C와 호환되는 오퍼레이션을 사용할 수 있다.

## Swift type-safe pointers

Swift에서는 type-safe하게 개발자가 포인터를 사용할 수 있도록 한다.

> 대신 포인터 객체의 생명 주기와 범위(ex. Buffer 범위를 벗어나는 경우)를 고려하여 다뤄야 한다.

#### UnsafePointer<T> is a *typed pointer*

- 타입이 정해진 메모리 주소를 의미.
- Typed pointer는 메모리에 정해진 타입에 대해서만 값을 읽거나 쓸 수 있다.

<img src="https://user-images.githubusercontent.com/22453984/92327707-8d9f2800-f096-11ea-85ce-8449610a5c9a.png" alt="스크린샷 2020-09-06 오후 11 13 15" style="zoom:67%;" />

#### Swift typed pointers are strict

일반적으로 다른 언어에서 포인터 타입은 그리 중요하지 않으며 해당 포인터를 다른 타입으로 캐스팅하는 것은 C에서 허용되는 작업이다.

그러나 Swift에서는 다른 타입의 포인터로 다른 포인터의 메모리에 접근하는 것은 Undefined 된 행동으로 간주된다. 따라서 C 언어에서와 같이 포인터를 다른 타입의 포인터로 캐스팅하는 행위는 금지된다. 이러한 방법으로 포인터 타입은 컴파일 시간에 Swift의 타입 시스템에 속하게 됨으로써 런타임 시에 타입 정보를 저장하는 비용이나 타입 정보를 확인하는 비용이 필요하지 않게 된다.

<img src="https://user-images.githubusercontent.com/22453984/92327727-c5a66b00-f096-11ea-9fc2-b06ec90c1f19.png" alt="스크린샷 2020-09-06 오후 11 14 50" style="zoom:67%;" />

#### Typed pointer 예시

```swift
withUnsafePointer(to: i) { (intPtr: UnsafePointer<Int>) in ... }
array.withUnsafeBufferPointer { (elementPtr: UnsafeBufferPointer<Element>) in ... }

let tPtr = UnsafeMutablePointer<T>.allocate(capacity: count)
tPtr.initialize(repeating: t, count: count)
tPtr.assign(repeating: t, count: count)
tPtr.deinitialize(count: count)
tPtr.deallocate()
```

만약, typed pointer가 아니라 상황에 따라 다른 타입으로 해석되어야 하는 경우에는 어떻게 해야할까?

## Swift *raw* pointers

Swift가 제공하는 Raw 포인터 타입을 사용하면 타입을 지정하지 않은 채 메모리에 접근이 가능하다.

### Loading bytes with UnsafeRawPointer

Raw 포인터를 사용하면 메모리에 들어있는 타입을 무시할 수 있다.

UnsafeRawPointer를 사용하여 다음과 같이 Int64 타입이 들어있는 메모리에서 UInt32 타입의 값으로 새로 로드할 수 있다.

<img src="https://user-images.githubusercontent.com/22453984/92328444-d0173380-f09b-11ea-9aa4-f8b5e707731d.png" alt="스크린샷 2020-09-06 오후 11 50 56" style="zoom:67%;" />

또한, UnsafeMutableRawPointer를 통해 메모리에 새로운 타입으로 쓰기 작업을 할 수도 있다. 메모리에 쓰기 작업이 완료되면 해당 값들은 다시 Int64로 재해석이 되어 기존의 포인터로도 접근이 가능하다. 하지만 RawPointer에서 UInt32 타입의 포인터로 캐스팅하는 작업은 메모리에 정해진 타입을 위반하기 때문에 여전히 불가능하다.

![스크린샷 2020-09-06 오후 11 53 40](https://user-images.githubusercontent.com/22453984/92328497-31d79d80-f09c-11ea-97ce-7f4651f33ce0.png)

#### Raw pointers 사용 예시

Typed pointer와 마찬가지로 클로저를 통해 RawPointer를 다룰 수 있도록 API를 제공하고 있다.

```swift
withUnsafeBytes(of: i) { (iBytes: UnsafeRawBufferPointer) in ... }
withUnsafeMutableBytes(of: &x) { (xBytes: UnsafeMutableRawBufferPointer) in ... }
array.withUnsafeBytes { (elementBytes: UnsafeRawBufferPointer) in ... }

func readUInt32(data: Data) -> UInt32 {
    data.withUnsafeBytes { (buffer: UnsafeRawBufferPointer) in
    	buffer.load(fromByteOffset: 4, as: UInt32.self)     
    }
}

// 클로저와 달리 직접 byteCount와 memory 크기 등을 정해줘야 한다.
let rawPtr = UnsafeMutableRawPointer.allocate(
	byteCount: MemoryLayout<T>.stride * numValues,
	alignment: MemoryLayout<T>.alignment)
let tPtr = rawPtr.initializeMemory(as: T.self, repeating: t, count: numValues)
// rawPtr의 deinit은 Typed pointer인 tPtr의 deinit을 통해 이룰 수 있음.
```



## Memory-binding APIs

![스크린샷 2020-09-07 오전 12 20 21](https://user-images.githubusercontent.com/22453984/92329056-ec1cd400-f09f-11ea-81fd-80f7c7b92ab1.png)

- `assumingMemoryBound(to:)`
  - 타입이 없는 포인터를 Typed 포인터로 만들어주는 API로 사전에 메모리에 어떤 타입 정보가 들어갔는지에 대해 알고 사용해야 한다.
- `bindMemory(to:capacity:)`
  - 메모리에 할당된 타입 정보를 변경하는 API. 기존에 사용되던 Typed 포인터를 유효하지 않게 하기 때문에 사용에 주의해야 함.
- `withMemoryRebound(to:capacity:)`
  - 메모리에 할당된 타입 정보를 임시로 변경하는 것으로 클로저를 통해 한정된 스코프 내에서 Typed 포인터를 다룰 수 잇게 한다.
  - 타입이 맞지 않는 C API를 다룰 때 유용함.

### Safely reinterpreting bytes

``` swift
// X
let uint32Ptr = rawPtr.bindMemory(to: UInt32.self)
return uint32Ptr.pointee

// O
return rawPtr.load(as: UInt32.self)
```

첫번째 방법처럼 메모리에 이미 바인딩된 타입을 변경한다면 이미 해당 메모리에 접근하여 사용하고 있는 포인터들은 모두 유효하지 않게 되어 의도치 않은 결과를 가져올 것이다.

때문에 Raw 포인터를 얻어(임시로 생성하거나) 원하는 타입으로 해당 바이트를 새로 해석하자.

## 결론

- 포인터를 사용하는 것을 피하자.
- Typed 포인터를 다른 타입으로 재해석하는 데 사용하지 말자.
- `UnsafeRawBufferPointer`를 사용하자.
  - Raw bytes를 다른 타입으로 재해석 해야할 때
  - Swift 타입을 바이트 스트림으로 복호화 해야할 때
  - 연속된 메모리에 다른 타입들을 포함하는 Container를 구현할 때

