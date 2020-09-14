# Handle the Limited Photos Library in your app

개인 정보를 보호하면서 앱에 필요한 사진과 동영상에 액세스 한다. 
새로운 Limited Photos Library 기능을 통해 사람들은 개인 콘텐츠를 보호하기 위해 앱이 액세스 할 수있는 사진과 비디오를 직접 제어 할 수 있습니다.

![image-20200913232710645](/Users/user/Library/Application Support/typora-user-images/image-20200913232710645.png)

- 기존 Photo Kit API는 사용자 권한을 허용 한다면, 모든 사진 라이브러리에 대해서 접근이 가능하다.

![image-20200913233226277](/Users/user/Library/Application Support/typora-user-images/image-20200913233226277.png)

- Limited Photo Library 는 사용자가 선택한 사진에만 접근이 가능하다.
- 새 API를 채택하지 않은 앱의 경우, 사용자에게 전체 액세스 권한을 부여하거나 제한된 수의 사진 만 선택하여 액세스 권한을 부여하라는 메시지가 표시됩니다.

- **Limited Photo Library** 에 대한 옵션 `select photos` 추가

### 앱에서 선택한 사진을 수정할 수 있는 방법

- 사용자는 iPhone에서 설정 앱을 방문하여 주어진 액세스 권한을 업데이트하여 사진 액세스를 업데이트 할 수 있습니다.

- 제한된 접근을 허용 한다면, 다음 앱 수명주기에서 다시 권한을 요청할 때까지 앱은 업데이트 된 인증권한 또는 추가 사진을 받을 수 없습니다.

### PHPicker

- `UIImagePickerController` 의 좋은 대체제이다.
- 검색기능, 다중 선택기능이 기본적으로 제공됨.
- 유저의 권한허용이 필요하지 않음.

### Querying for authorization status

- 새로운 상태 권한타입 추가
  -  `.limited`
- 앱에 제한된 액세스 권한이 있음을 나타 내기 위해 `PHAccessLevel` 추가
  - `.addOnly` 
  - `.readWrite`
- 권한 체크

![image-20200914002721992](/Users/user/Library/Application Support/typora-user-images/image-20200914002721992.png)

- 수동으로 액세스를 요청하면 현재 상태가 미확인 인 경우에만 사용자에게 메시지가 표시됩니다

![image-20200914002904781](/Users/user/Library/Application Support/typora-user-images/image-20200914002904781.png)

- 기존에 사용하던 버전 **Deprecated**

![image-20200914003049932](/Users/user/Library/Application Support/typora-user-images/image-20200914003049932.png)

> 기존 API는 새로운 타입 `.limited` 을 인식하지 못하며, `.authorizied`를 반환하게 됨



### Notable API differences with limited access

- PHAssets created with PHAssetCreationRequests accessible to your app
- Can't create or fetch users albums 
- No cloud shared assets or albums



### Control photo library management UI

- 기본적으로 선택 UI를 표시하는 방법을 다루고 두 번째로 시작 후 photokit apis 호출을 수행 할 때 앱에 대해 자동 시스템 프롬프트가 발생하지 않도록하는 방법에 대해 설명하겠습니다.

![image-20200914004539834](/Users/user/Library/Application Support/typora-user-images/image-20200914004539834.png)

-  `PHPhotoLibraryChangeObserver` 를 통해 접근 변경을 감지할 수 있음.



### Preventing the automatic prompt on first access

- Set `PHPhotoLibraryPreventAutomaticLimitedAccessAlert` to `YES` in app’s `info.plist`

### 요약

- 먼저 앱이 사진 라이브러리 액세스를 사용해야하는지 다시 고려하세요. 올해 새로운 PHpicker 개선 사항은 많은 기능을 구현할 수있는 좋은 방법입니다.
- 새로운 인증 API를 채택하여 앱이 제한된 라이브러리 모드에 있는지 알 수 있습니다.
- 앱 디자인을 업데이트하여 사용자가 제한된 라이브러리 선택을 앱 컨텍스트에서 의미있는 방식으로 수정할 수 있도록합니다
- 마지막으로 Xcode에서 info Plist 키를 설정하여 앱이 실행 된 후 PhotoKit API를 처음 호출 할 때 시스템 프롬프트가 표시되지 않도록합니다.