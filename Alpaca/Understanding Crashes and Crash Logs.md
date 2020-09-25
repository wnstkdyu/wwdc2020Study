# Understanding Crashes and Crash Logs

#### 크래시는 왜 일어날까?

- CPU에서의 코드 실행이 불가능할 때 (예: 0으로 나누는 경우)
- OS에서 정책을 강제할 때 (예: 앱 런치 타임이 너무 긴 경우)
- 언어 차원에서 정의되지 않은 행동을 막을 때 (예: 배열 범위를 넘어가서 접근할 때)

크래시 로그 자체는 Unsymbolicated 된 상태로 저장되지만 Xcode에서는 이것을 사람이 읽을 수 있도록 Symbolicated 된 형태로 만들어준다.

<img src="https://user-images.githubusercontent.com/22453984/94219797-990da280-ff22-11ea-8314-a3ca881411f2.png" alt="스크린샷 2020-09-25 오전 11 30 24" style="zoom: 67%;" />

## 크래시 로그에 접근하기

크래시 로그를 살펴볼 수 있는 방법에는 여러가지가 있지만 먼저 Release 버전이나 TestFlight 버전의 크래시 로그를 확인할 수 있는 **Crashes Organizer**가 있다.

<img src="https://user-images.githubusercontent.com/22453984/94220376-c018a400-ff23-11ea-84b5-1d41325276b3.png" alt="스크린샷 2020-09-25 오전 11 39 16" style="zoom:70%;" />

Crashes Organizer를 통해 크래시 최근 통계, 영향을 받는 기기 정보, 크래시 위치 정보를 알 수 있다.

하지만 만약 TestFlight나 AppStore를 통해 앱을 배포하지 않았다면 다른 선택지를 사용하여 크래시 로그를 봐야 한다.

- Devices window
  - ![스크린샷 2020-09-25 오전 11 54 02](https://user-images.githubusercontent.com/22453984/94221315-d0ca1980-ff25-11ea-8e30-03f2ed577005.png)
- Automated testing
  - Xcode 서버나 Xcode를 통한 테스트 시 크래시가 일어날 경우에도 크래시 로그를 Symbolicated 된 형태로 보여줌.
- Console app
- Sharing from device
  - 설정 -> 개인 정보 보호 -> 분석 및 향상 -> 분석 데이터
  - <img src="https://user-images.githubusercontent.com/22453984/94230511-10e8c680-ff3d-11ea-9399-5f7c1464c529.jpeg" alt="IMG_0E04D426DC16-1" style="zoom:25%;" />

#### Symbolication

Symbolication은 개발자가 크래시를 추적할 단서를 얻는 데 큰 도움이 되기 때문에 매우 중요하다. 다음은 애플이 권장하는 Best practices이다.

- 서버 사이드에서의 Symbolication을 위해 앱 Symbol을 업로드한다.
- 로컬 Symbolication을 위해 앱 아카이브를 보관한다.
  - 앱 아카이브에는 dSym(Debug symbol)을 가지고 있기 때문에 이를 통해 로컬에서 Symbolication이 가능하다.
- *bitcode* 앱은 dSYM을 다운받아서 Symbolication을 하자.
  - bitcode를 사용하면 개발자가 올린 바이너리와 사용자가 사용하는 바이너리가 달라지기 때문에 따로 다운받아야 함.



## 크래시 로그 분석하기

크래시 로그에는 기기 정보, 스택 트레이스, 크래시가 일어났을 당시의 다른 스레드의 모습 등 다양한 정보들을 가지고 있다.

먼저, 처음에는 어떤 종류의 크래시가 일어났는지 알아야 한다.

> 참고 문서: https://developer.apple.com/documentation/xcode/diagnosing_issues_using_crash_reports_and_device_logs/identifying_the_cause_of_common_crashes

<img src="https://user-images.githubusercontent.com/22453984/94247966-cd9c5100-ff58-11ea-9fb5-2778118184b7.png" alt="스크린샷 2020-09-25 오후 5 58 59"  />

다음으로, 크래시 당시에 어떤 메서드가 호출되었는지와 어떤 에러 메시지가 나왔는 지를 본다.

<img src="https://user-images.githubusercontent.com/22453984/94248138-00dee000-ff59-11ea-9328-5075fe42c457.png" alt="스크린샷 2020-09-25 오후 6 00 29" style="zoom:80%;" />

#### Assertions and Preconditions

위의 경우와 같이 어떠한 조건을 만족시키지 못할 때 런타임에서는 프로세스를 멈춘다.

이와 같은 경우에는,

- nil을 갖고 있는 옵셔널 값의 강제 언래핑
- Swift 배열에 범위를 벗어난 접근
- Swift 연산에서의 Overflow
- Uncaught exception
- Custom assertion in your code

하지만, 이와 달리 OS 상으로 멈추는 경우도 존재한다.

#### Terminations by the Operating System

- Watchdog events
  - 앱을 Launch 시키는 데 너무 많은 시간이 들 경우
- Device overheated
- Memory exhaustion
- Invalid code signature

> OS 상으로 일어나는 크래시 로그는 Crash Organizer에 때때로 수집되지 않으며 대신 기기 내부에서 얻을 수 있음.



### Memory Errors

다음 Exception Type은 메모리 관련 크래시 로그에서 전형적으로 보인다.

![스크린샷 2020-09-25 오후 6 14 29](https://user-images.githubusercontent.com/22453984/94249500-f4f41d80-ff5a-11ea-9963-2003a2cf232e.png)

BAD_ACCESS 에러는 두 가지로 나뉜다.

- Read-Only 메모리에 Write하려고 할 경우
- 유효하지 않은 메모리를 읽으려고 할 경우

크래시가 일어난 스레드를 살펴 보면 `objc_release` 메서드가 호출될 때 크래시가 일어난 것을 볼 수 있다. 해당 메서드는 참조 카운팅의 구현부에서 호출하는 것으로 이 크래시가 메모리와 관련 있다는 것을 암시적으로 알려준다.

![스크린샷 2020-09-25 오후 6 17 20](https://user-images.githubusercontent.com/22453984/94249800-5b793b80-ff5b-11ea-86eb-eb8b318bf1dd.png)

다음으로 어떤 객체를 release할 때 크래시가 일어났는 지 보려면 Stack Trace를 참고하자.

![스크린샷 2020-09-25 오후 6 20 12](https://user-images.githubusercontent.com/22453984/94250114-c1fe5980-ff5b-11ea-8dd4-82dc7527f8fc.png)

크래시를 일으킨 주소값을 보면 MALLOC 되는 영역 안에 들어가 있는 것을 볼 수가 있는데..

<img src="https://user-images.githubusercontent.com/22453984/94250601-65e80500-ff5c-11ea-864c-39a7a847648a.png" alt="스크린샷 2020-09-25 오후 6 24 48" style="zoom:80%;" />

보다 정확한 상황 파악을 위해서 `objc_release` 가 어떻게 동작하는 지 보자.

`objc_release` 메서드는 다음과 같이 동작한다.

1. release되려는 객체의 isa 포인터를 참조한다.
2. isa 포인터를 통해 Class 정보에 접근하여 dereference 한다. (참조 카운트를 감소시킨다.)

![스크린샷 2020-09-25 오후 6 24 03](https://user-images.githubusercontent.com/22453984/94250521-4cdf5400-ff5c-11ea-9c28-89c498e29cc7.png)

해당 과정은 일반적인 경우에 정상적으로 동작하지만 만약 이미 메모리에서 해제된 객체에 대해 동작할 경우 어떻게 될까?

`free` 메서드는 객체를 삭제(메모리에서 해제...?)하고 원래 isa 포인터가 있던 자리에 **free list ptr** 를 넣음으로써 freed 객체의 리스트에 넣는다. 

<img src="https://user-images.githubusercontent.com/22453984/94251303-4d2c1f00-ff5d-11ea-8dec-5e31b68b772d.png" alt="스크린샷 2020-09-25 오후 6 31 14" style="zoom: 67%;" />

이 free list 포인터는 리스트가 변경될 때 **rotated free list** 포인터로 새로 쓰여진다. 해당 리스트를 관리하는 이유는 해당 메모리 영역은 유효하지 않다는 정보를 유지하는 것이다.

<img src="https://user-images.githubusercontent.com/22453984/94251796-07bc2180-ff5e-11ea-8a69-d23c18e8ed76.png" alt="스크린샷 2020-09-25 오후 6 36 28" style="zoom:67%;" />

때문에 `objc_release` 함수가 해당 메모리에 접근할 경우 rotated free list ptr를 읽고 dereference 작업을 하는 것은 크래시를 일으킨다.

이제 어떤 객체를 release 할 때 크래시가 발생하는 지 좀 더 좁혀보자. 아래와 같이 `LoginViewController` 에서 `__ivar_destroyer` 메서드를 실행할 때 발생하는 것은 알 수 있지만 해당 메서드는 우리가 직접 호출하는 것이 아닌 컴파일러가 자동으로 생성한 것이다.

![스크린샷 2020-09-25 오후 6 41 34](https://user-images.githubusercontent.com/22453984/94252331-bd877000-ff5e-11ea-9bda-a0936acb3dee.png)

때문에 정확한 지점을 찾기 위해서는 `+42`라는 숫자에 주목해야 한다. 해당 숫자는 `__ivar_destroyer` 함수의 어셈블리 코드의 offset이기 때문에 정확한 지점을 찾는 단서가 된다. `__ivar_destroyer` 함수를 disassemble하여 해당 단서를 활용하자.

커맨드 콘솔을 통해 크래시 로그 파일을 부른 뒤에 disassemble 작업을 수행할 수 있다. 해당 크래시 로그를 로드해서 우리가 읽을 수 있는 것은 dSym 파일과 앱의 복사본이 모두 맥에 있기 때문이다. lldb에서는 Spotlight를 통해 불러오려는 크래시 로그에 맞는 앱 파일과 dSym 파일을 찾게 된다.

<img src="https://user-images.githubusercontent.com/22453984/94253198-e9efbc00-ff5f-11ea-8c52-306293d14caf.png" alt="스크린샷 2020-09-25 오후 6 49 56" style="zoom:67%;" />

아래는 `__ivar_destroyer` 메서드를 disassemble한 모습이다.

각각의 프로퍼티들을 release하는 것을 볼 수 있으며 +42에서는 views를 release하는 것을 볼 수 있다. 중요한 것은 스택 트레이스에서 보여주는 offset은 해당 호출의 반환 주소값이라는 것이다. 즉, views를 release하는 위치로 반환되었기 때문에 실제로 크래시가 일어난 offset은 +37이라는 것이다.

해당 offset에서는 `database` 프로퍼티를 release하고 있다.

<img src="https://user-images.githubusercontent.com/22453984/94253235-f5db7e00-ff5f-11ea-9f7c-fc2caf07145e.png" alt="스크린샷 2020-09-25 오후 6 50 17" style="zoom:67%;" />

 이제 남은 것은 이 `database` 를 사용하는 코드를 보며 원인을 찾는 것이다.

#### Crash Log Analysis Summary

지금까지의 과정을 요약하자면 다음과 같다.

- 크래시를 일으킨 원인 이해
- 크래시가 일어난 스레드의 스택 트레이스 분석
- Bad address와 disassebly를 통해 단서 찾기

#### Common Memory Error Symptoms

<img src="https://user-images.githubusercontent.com/22453984/94254603-fd038b80-ff61-11ea-9a13-e4e42bb4d81c.png" alt="스크린샷 2020-09-25 오후 7 04 48" style="zoom:67%;" />

