# Explore logging in Swift

### Logs help fix complex bugs

사용자들은 믿을 만한 퀄리티를 가진 앱을 기대하지만 때때로 재현이 잘 되지 않는 버그는 고치기 힘들다. 이러한 상황에서 로그는 버그를 이해하는 데 도움이 될 수 있다.

## New logging APIs in Xcode 12

- 이벤트 발생 시 해당 이벤트를 기록
- 기기 내에 아카이빙되어 이후에 가져올 수 있음
- 성능 상의 오버헤드는 적어 광범위하게 사용이 가능

### 사용 방법

```swift
// os 모듈을 import하여 로깅 API를 사용할 준비를 함.
import os

// subsystem과 category를 통해 로그를 식별할 수 있도록 함.
// subsystem은 bundle id를 주로 사용하며 category는 앱 내의 세부 분류를 의미.
let logger = Logger(subsystem: "com.example.Fruta", category: "giftcards")

func beginTask(url: URL, handler: (Data) -> Void) {
    launchTask(with: url) {
        handler($0)
    }
    logger.log("Started a task \(taskId)")
}
```

Log 메시지는 String Interpolation의 사용이 가능한데, print 문처럼 전부 문자열로 변환되지는 않는다. 왜냐하면 해당 작업은 너무 느리기 때문이다.

대신, 컴파일러와 로깅 라이브러리가 로그 메시지의 최적화된 표현 방식을 만들어내며 단지, 실제로 보여질 때만 문자열로 변환되어지는 비용이 들 뿐이다.

### Built in logging for a wide range of data types

- Numeric 타입
- description이 정의된 Objective-C 객체
- `CustomStringConvertible` 을 준수하는 모든 타입

#### 민감 정보의 로깅 제한

기본적으로 배포된 앱에서 Numeric 값을 로깅하려고 하면 해당 값은 개인 정보 보호를 위해 찍히지 않고 대신 `<private>` 으로 나오게 된다.

하지만 다음과 같이 privacy 보호 정도를 명시해 주면 해당 정보가 정상적으로 로그에 찍히게 된다.

```swift
logger.log("Paid with bank account \(accountNumber)")
logger.log("Ordered smoothie \(smoothieName, privacy: .public)")
```

### View and filter log archives in Console.app

- .logarchive 확장자를 가진 파일을 열어 확인
- 검색과 필터링이 쉬움

기기 연결 후 Console 앱을 통해 로그 확인 시에는 모든 프로세스에 대한 로그가 쌓이기 때문에 Logger 생성 시에 전달한 `subsystem` 값으로 앱을 필터링하여 맞는 로그를 찾자.

### Stream logs while app is running

- Mac에 연결된 기기가 있다면 Console 앱을 통해 확인이 가능
- Xcode를 통해 Run을 했다면 Xcode의 Console 창에 로그가 찍힐 것임

## Log levels

중요도에 따라 다섯 개로 나누고 있으며 레벨에 따라 콘솔 창에서 로그가 분류되며 로그 레벨이 중요해짐에 따라 로그가 프로세스 종료 후에도 유지되느냐, 그렇지 않느냐가 결정됨.

따라서 레벨이 높아짐에 따라 로깅의 성능은 저하됨.

- **Debug**
  - 디버깅 시에만 유용한 로그
  - 유지되지 않음.
- **Info**
  - 유용하지만 문제 해결에는 크게 도움이 되지 않는 로그
  - `collect` 명령 전에만 조금 유지.
- **Notice (Default)**
  - 문제 해결에 필수적인 로그
  - 저장소 용량이 다 찰 때까지 유지.
- **Error**
  - 실행 시에 보이는 에러를 기록
  - 저장소 용량이 다 찰 때까지 유지.
- **Fault**
  - 좀 더 심각한 버그를 의미
  - 저장소 용량이 다 찰 때까지 유지.

Error와 Fault 레벨은 각각 노랑, 빨강 색으로 콘솔 창에 표시가 됨.

Logger는 각각의 레벨에 따라 쉽게 로그를 찍을 수 있도록 메서드를 제공하고 있음.

``` swift
Logger.notice(_ message:)
Logger.fault(_ message:)
```

## Formatted Log

로그를 읽는 데에 가독성을 높이기 위해 Raw 데이터는 가공을 거쳐야 하는데 Logging API는 이를 런타임 시의 비용없이 가능케 한다.

```swift
logger.log("\(data, format: .hex, align: .right(columns: width))")
```

### Personal data must not be public

개인 정보가 담긴 로깅은 privacy 설정을 `.public` 을 하지 말아야 한다.

## API availability

iOS 14부터 사용이 가능
