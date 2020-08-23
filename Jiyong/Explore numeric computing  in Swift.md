- # Explore numeric computing in Swift

  ### Swift Numerics Package Overview

  - Provides building blocks for `generic` numerical computing in Swift.
  - Includes a protocol called [`Real`](https://github.com/apple/swift-numerics/blob/c634b5ddcea8445244b7cde1e5deddb8adf58e37/Sources/RealModule/Real.swift#L30) and a [`Complex`](https://github.com/apple/swift-numerics/blob/c634b5ddcea8445244b7cde1e5deddb8adf58e37/Sources/ComplexModule/Complex.swift#L48) number type.
  - High-performance `Float16` type.

  

  ### Example : The logit function 

  ```swift
  import Darwin
  
  func logit(_ p: Double) -> Double { 
  	log(p) - log1p(-p)
  }
  ```

  - 이 함수는 Double 타입에 대해서만 지원하기 때문에 문제가 됩니다. 
  - 다른 타입으로도 사용하고 싶다면 제네릭으로 시도해 볼 수 있습니다.

  ```swift
  func logit<T>(_ p: T) -> T { 
  	log(p) - log1p(-p)	
  }
  ```

  - 그러나 log 및 log1p 함수는 소수의 부동 소수점 유형에만 의미가 있기 때문에 컴파일되지 않습니다.

  ```swift
  import Numerics
  
  func logit<NumberType: Real>(_ p: NumberType) -> NumberType {
    log(p) - log1p(-p)
  }
  ```

  - `Numerics` 모듈은 `Real` 프로토콜을 제공하여 코드가  표준 부동 소수점 유형에 대해 작동 할 수 있도록합니다. 

  

  ### [The Real Protocol](https://github.com/apple/swift-numerics/tree/5dfc460876510988560170cee3702ab01b89587a/Sources/RealModule)

  - Part of Swift Numerics package
  - Provides generic access to all standard floating-point capabilities.

  ```swift
  public protocol Real: FloatingPoint, AlgebraicField, RealFunctions { ... }
  ```

  - [AlgebraicField](https://github.com/apple/swift-numerics/blob/5dfc460876510988560170cee3702ab01b89587a/Sources/RealModule/AlgebraicField.swift#L48) and [`RealFunctions`](https://github.com/apple/swift-numerics/blob/5dfc460876510988560170cee3702ab01b89587a/Sources/RealModule/RealFunctions.swift#L12) are also new protocols defined in the Numerics package

  

  ### Numeric Protocols in the Swift standard library

  <img src="/Users/user/wwdc2020Study/Jiyong/images/numerics_swift_standard.png" alt="numerics_swift_standard" style="zoom:50%;" />

  This topic focuses on [`AdditiveArithmetic`](https://developer.apple.com/documentation/swift/additivearithmetic), [`SignedNumeric`](https://developer.apple.com/documentation/swift/signednumeric), and [`FloatingPoint`](https://developer.apple.com/documentation/swift/floatingpoint)

  - `AdditiveArithmetic` 덧셈과 뺄셈 유형에 적용되는 타입
  - `SignedNumeric` 곱셈 타입
  - `FloatingPoint` 
    - 부동 소수점 계산을위한 다른 많은 연산을 추가합니다.
    - 비교 함수, 숫자를 지수 및 유효 숫자로 분해하는 방법, 무한대 또는 파이와 같은 유용한 상수가 포함됩니다.

  

  

  ### Protocols in the Numerics package

  기존 구현타입을 보강하여 하나의 Real 프로토콜로 조합함.

<img src="/Users/user/wwdc2020Study/Jiyong/images/real.png" alt="real" style="zoom:50%;" />

 ### Implications of the Real protocol

- Real 프로토콜로 제한된 제네릭은 모든 표준 부동 소수점 유형을 지원합니다.
- 중복코드를 줄여준다.
- 유지보수를 단순화시킴
- numerical 타입을 쉽게 생성 시킴



### The Complex Type

- Swift Numerics package의 일부이다.
- `Double`이 기본형 이다.

```swift
import Numerics
let z: Complex<Double> = Complex(1.0, 2.0) // z = 1 + 2i
```

### Generic Numerical Programming

Complex는 그 자체만으로 타입이지만, 제네릭 numerical프로그래밍 또한 이용할 수 있따.

```swift
public struct Complex<NumberType> where NumberType: Real {
    /// The real component
    public var real: NumberType
  
    /// The imaginary component
    public var imaginary: NumberType
  
    /// Construct a complex number with specified real and imaginary parts
    public init(_ real: NumberType, _ imaginary: NumberType) {
        self.real = real
        self.imaginary = imaginary
    }
}
```

복소수가 완전히 기능하도록하려면 SignedNumeric 프로토콜을 사용하여 확장해야합니다.

```swift
extension Complex: SignedNumeric {
    /// The sum of 'z' and 'w'
    public static func +(z: Complex, w: Complex) -> Complex {
        return Complex(z.real + w.real, z.imaginary + w.imaginary)
    }

    /// The difference of 'z' and 'w'
    public static func -(z: Complex, w: Complex) -> Complex {
        return Complex(z.real - w.real, z.imaginary - w.imaginary)
    }

    /// The product of 'z' and 'w'
    public static func *(z: Complex, w: Complex) -> Complex {
        return Complex(z.real * w.real - z.imaginary * w.imaginary,
                       z.real * w.imaginary + z.imaginary * w.real)
    }
}
```



### Compatibility with C and C++

**2 개의 부동 소수점 값이있는 일반 구조체이므로 메모리 레이아웃이 정확히 C 및 C ++의 복소수 유형과 일치합니다.**

The following are all the same in memory:

- `Complex` (Swift)
- `_Complex double` (C)
- `std::complex` (C++)

결과적으로 Swift 코드는 C / C ++ API로 복소수의 버퍼를 교환 할 수 있습니다.
복잡한 유형을 예상하는 C / C ++ 라이브러리에 Swift `Complex` 구조체에 대한 포인터를 전달할 수 있습니다.



### 성능 비교

Numerics 패키지는 복잡한 무한대 및 NaN을 C 또는 C ++ 방식과 다르게 처리합니다.
이는 C 또는 C ++에서 포팅 된 코드에 영향을 줄 수 있습니다.

그러나 이러한 값에 대한 Swift의 처리는 곱셈과 나눗셈에 대해 더 간단하고 훨씬 더 성능이 좋습니다.
 아래는 Swift와 C를 비교 한 벤치 마크입니다.

<img src="/Users/user/wwdc2020Study/Jiyong/images/complex_benchmark.png" alt="complex_benchmark" style="zoom:50%;" />

### Float16

a new floating-point type in the Swift standard library.

- IEEE 754 standard format.
- Already available on iOS, iPadOS, tvOS, watchOS (ARM-based platforms)
- Calling convention for x86 is not yet finalized (working with Intel to fix this).

a normal floating-point type

- Conforms to `Real` from Swift Numerics.
- Conforms to `BinaryFloatingPoint` and `SIMDScalar`.
- If you implement the `Real` protocol in your code, you will automatically get support for `Float16`.

**Pros**

- SIMD 레지스터 또는 메모리 페이지에 더 많은 것을 넣을 수 있기 때문에 성능이 크게 향상됩니다.
- C / Objective-C __fp16 유형과 상호 운용됩니다.

**Cons**

- 낮은 정밀도, 작은 범위.

### Hardware Support

- Supported (and preferred) by Apple's GPUs.
- Supported by Apple's CPUs starting with A11 Bionic.

 <img src="/Users/user/wwdc2020Study/Jiyong/images/float16_benchmark.png" alt="float16_benchmark" style="zoom:50%;" />