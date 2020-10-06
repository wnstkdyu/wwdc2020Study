# Support Local Network Privacy in Your App

iOS 기기를 통해 일반적인 인터넷 네트워크 대신 주변 기기과 소통할 수 있는데 이 때 사용되는 것이 '로컬 네트워크' 기능이다. iOS 기기에서 로컬 네트워크 기능을 사용할 때는 **Bonjour** 기능을 사용하면 된다.

> [봉쥬르(Bonjour)가 뭔가요?](https://m.blog.naver.com/PostView.nhn?blogId=ksw2015&logNo=220409484135&proxyReferer=https:%2F%2Fwww.google.com%2F)

Bonjour 서비스를 통해 로컬 네트워크에 접속하여 다음과 같은 작업이 가능하다.

- Media streaming
- Interactive games (Peer-to-peer games)
- Home and peripheral devices

또 보다 낮은 단의 네트워크 제어에도 사용된다.

- Network management
- Custom multicast protocols
  - Legacy 하드웨어와 통신 시에 사용된다고 한다.

## Privacy 문제

Local Network를 사용함으로써 앱의 강력한 기능을 구현할 수 있지만 큰 책임도 따르게 된다. 바로 Privacy 이슈이다.

<img src="https://user-images.githubusercontent.com/22453984/95210820-d62d3b00-0826-11eb-826f-fc6d364d57c6.png" alt="스크린샷 2020-10-06 오후 10 53 36" style="zoom: 33%;" />

로컬 네트워크를 사용하여 주변 기기를 탐색하는 것은 사용자의 위치에 대한 힌트를 제공할 수 있다. 왜냐하면 각 로컬 네트워크에 속한 기기들과 서비스들은 유니크하기 때문이다. 이러한 정보를 통해 사용자의 위치를 유추할 수 있다.

<img src="https://user-images.githubusercontent.com/22453984/95212064-47212280-0828-11eb-91b1-9fbf7b896fe9.png" alt="스크린샷 2020-10-06 오후 11 04 23" style="zoom:33%;" />

또한 로컬 네트워크를 통해 기기와 연결되어 필요한 작업을 하는 앱이 있을 때 다른 앱들도 해당 기기에 접근하여 기기와 소통할 수 있다. 이 경우 후자에 해당하는 앱은 기기와 연결될 필요가 없음에도 불구하고 기기에 접근이 가능하다는 문제점이 있다.

따라서 iOS 14부터는 사용자가 앱이 로컬 네트워크의 사용이 가능한 지에 대한 제어가 가능해진다. 이를 통해 사용자에게 로컬 네트워크를 사용하는 목적을 보다 명확하고 투명하게 전달할 수 있고 개발자에게는 정말 로컬 네트워크 기능이 필요한 지 검토해 볼 기회가 될 수 있다.

<img src="https://user-images.githubusercontent.com/22453984/95214897-535aaf00-082b-11eb-8414-c34f877dc6d1.png" alt="스크린샷 2020-10-06 오후 11 26 11" style="zoom: 50%;" />

위의 그림과 같이 일반적인 인터넷이나 시스템이 제공하는 로컬 네트워크 기능을 사용한다면 전혀 제한이 없다. 하지만 로컬 네트워크를 통해 TCP와 UDP 연결, Bonjour 서비스를 이용하는 등의 작업은 권한 획득을 먼저 해야 가능하다.

### Local network access prompt

<img src="https://user-images.githubusercontent.com/22453984/95215641-25c23580-082c-11eb-873b-3f2c48fcfdf4.png" alt="스크린샷 2020-10-06 오후 11 32 03" style="zoom:33%;" />

사용자가 로컬 네트워크 기능을 처음 사용하려고 할 때 시스템은 이러한 Prompt를 띄우게 된다. Prompt의 문구는 다른 권한 요청과 같이 Info.plist에서 사용 목적을 적어 구성하면된다. 만약 사용자가 권한을 허용하지 않는다면 이후에 로컬 네트워크를 통한 작업은 모두 막히게 된다.

또한, 외부 라이브러리에서 로컬 네트워크를 이용하는 경우에도 해당되기 때문에 유의해야 하며 해당 앱에서 허용할 경우에는 외부 라이브러리의 기능 또한 문제없이 동작할 것이다.

> iOS 14 용으로 업데이트되지 않은 앱은 기본 사유가 적혀 Prompt가 뜨게 된다.

<img src="https://user-images.githubusercontent.com/22453984/95217000-adf50a80-082d-11eb-9246-938a8f6fbd83.jpeg" alt="IMG_CC7AF6E62237-1" style="zoom:20%;" />

설정의 '개인 정보 보호' 탭에서 로컬 네트워크를 사용하는 앱들을 볼 수 있으며 단일 앱의 설정에서도 사용 여부의 확인 및 제어가 가능하다.

### App Requirements

#### Info.plist keys

- `NSLocalNetworkUsageDescription`: 로컬 네트워크를 사용하는 이유를 명시
- `NSBounjourService`: Bonjour 서비스 타입을 명시

<img src="https://user-images.githubusercontent.com/22453984/95217594-51deb600-082e-11eb-87ac-14b9f03da807.png" alt="스크린샷 2020-10-06 오후 11 47 35" style="zoom: 67%;" />



#### Avoid unexpected prompts

늘 그렇듯이 꼭 필요할 때에만 권한을 요청 및 기능을 사용하라고 권장하고 있다.

- 로컬 네트워크 사용을 미루자.
  - 앱 런치 시에 브라우징은 피하자.
  - 유저 인터랙션을 기다리자.
- 분명한 사용 이유를 밝히자.



