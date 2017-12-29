# Expected App behaviors

IOS 앱은 단순히 기기,시뮬레이터에서 올바르게 동작한다는 것 만으로 App Store에서 판매될 순 없다. 
유저에게 다양한 경험을 제공할 수 있도록 정보를 효과적으로 표현하고, 다루어야 한다.
이 챕터에선 모든 앱들이 다루어야 하고, 설계 과정에서 고려해야하는 부분을 설명한다.



### 필수 정보

앱이 iOS 기기에서 동작하려면 다음과 같은 필수 정보들이 필요하다.

- ##### Info.plist

  시스템이 앱과 소통하기 위한 여러  metadata들을 담고있다. 

- ##### App's Capabilities

  앱이 사용하는 하드웨어 사양, 특성을 명시하여야 한다. 

- ##### App icons

  다양한 환경에서 앱의 아이콘이 표시된다. 하나 이상의 icon은 필수적이다.

- ##### Launch images

  Launch image는 사용자에게 앱이 곧 실행될 것 임을 나타낸다. 하나 이상의 launch image는 필수이다.

  ​

#### The App Bundle

Bundle은 App의 다양한 정보를 한 곳으로 모아 저장한 directory이다. 아래 표에 일반적인 bundle file들이 정리되어 있다.

| File                                   | Description                              |
| :------------------------------------- | ---------------------------------------- |
| App executable                         | App의 컴파일된 코드이다. 필수적이다.                   |
| info.plist                             | 설정 데이터들이 저장되어 있다.  필수적이다.                |
| app icons                              | App의 Icon 이미지 파일이다. 파일 이름에 @2x를 붙여 레티나 디스플레이에 대응한다. 필수적이다. |
| launch images                          | 앱의 launch image이다. 앱이 실행되는 시점에 일시적으로 표시되며, 곧 앱이 실행됨을 유저에게 알려준다. 필수적이다. |
| Storyboard files                       | App의 Screen에 표시되는 view 들을 담고있다. info.plist의 설정을 바꿈으로서 MainStoryboard의 이름을 변경할 수 있다. 필수는 아니지만 권장된다. |
| Ad hoc distribution icon               | ad hoc 배포가 목적이라면, icon 이미지를 포함하여야 한다. App Store엔 배보되지 않으므로, 대체할 이미지를 의미한다. Ad hoc 배포라면 필수적이며, 그렇지 않을 경우 선택적이다. |
| Settings bundle                        | Settings 앱을 통한 별도의 App 설정을 원한다면 필요하다. 선택적이다. |
| Nonlocalized resource files            | 범용적인 resources들을 의미한다. app bundle의 최상위에 위치하여야 한다. |
| Subdirectories for localized resources | 현지화 자료들은 언어별로 정해진 폴더에 위치하여야 한다. 현지화에 필요한 다양한 자료들은 폴더에 정리되어 있어야 한다. |

##### Note : iOS App bundle은 Resources라는 이름을 폴더명으로 지정할 수 없다.

------



### The Information Property List File

> Xcode는 다양한 목적으로 compile 시점에 info.plist를 참조한다. 중요한 정보가 담겨있으므로, 필수적으로 포함하여야 한다.  Default로 설정된 값들도 있지만, 개발자가 필요에 따라 설정하여 사용할 수 있다.
>
> #### 유용하게 사용할 수 있는 설정들을 알아보자.

- ##### capabilities 

  App의 하드웨어 사양 정보를 의미한다. App Store는 이 정보를 App 구동 시 요구사항 확인을 위해 사용한다.


- ##### UIRequiresPersistentWiFi 

  서버와 통신하는 App의 경우 비활성화 상태에서도 Wi-Fi 커넥션이 유지되어야 한다. 이 값을 YES로 설정 함으로써, Wi-Fi 연결 해제를 방지한다.


- ##### UINewsstandApp

  Newsstand App의 콘텐츠를 표시한다면, 설정하여야 한다.


- ##### Define custom document types

  App이 특수한 type의 파일을 다룬다면 명시하여야 한다.


- ##### URL schmes

  다른 App과 통신하기 위한 특별한 URL schemes을 다룬다면 명시하여야 한다. 


- ##### Accessing user data & app features

  사용자 정보 접근과 같은 privacy와 관련된 문제가 발생한다면, iOS는 즉시 사용자에게 알리고, 권한을 요청한다. App이 사용자의 정보를 사용한다면, 그 목적을 info.plist 에 명시하여야 한다.



---



### Supporting User Privacy

> 많은 iOS 기기는 유저정보를 담고있기 때문에, 그걸 고려한 App 설계는 필수적이다. 유저 정보에 대한 접근은 보호되어야 하며, 정보의 사용은 투명해야한다.

- 정보가 필요하다면, 사용 목적을 담은 String을 명시해야한다. iOS에 의해 보호되는 개인정보로는 위치, 연락처, 일정, 사진, 미디어 등등이다. 

- 정보 사용은 투명하게 진행되어야 한다. 그에 대한 정책이나 서명을 iTunes Connect 의 metadata로 제공할 수 있다.

- 유저가 개인 정보 사용에 대한 설정을 할 수 있게 기능을 제공하여야 한다.

- 최소한의 정보를 사용하여야 한다.

- 정당한 과정을 통하여 정보를 보호하여아 한다. local에 저장시엔 iOS data protection feature를 참고하며, network를 활용시엔 ATS를 활용하여야 한다.

- ASIdentifierManager class를 사용한다면, advertisingTrackingEnabled 변수를 활용하여야 한다. 사용자에 의해 NO로 설정된다면, ASIdentifierManager를 활용하여 제한된 광고를 제공하여야 한다.

- uniqueIdentifier 프로퍼티로 제공되는 UDID사용을 자제하라. iOS 5.0에서 deprecated 되었으며, 이를 사용하는 App은 허용되지 않는다. UIDevice class의 identifierForVendor나,  ASIdentifierManager class의 advertisingIdentifier 를 활용하여야 한다.

- Audio input을 제공한다면, recording이 시작되는 시점에 audio session을 활성화 하여야 한다.

  ​

#### Internationalizing Your App

> iOS App은 다양한 국가에 배포되기 때문에, 현지화는 다양한 사용자의 접근을 가능하게 한다. 현지화는 사용하는 resource에 대한 현지화와 제공되는 언어의 현지화를 의미한다. 
>
> 완벽한 현지화를 위하여 다음과 같은 file들의 현지화가 필요하다.

- **Storyboard** - Storyboard는 현지화가 필요한 다양한 label과 content가 포함되어 있다. text길이 변화에 따른 대응 또한 필요하다.

- **Strings files** - 특정 문구에 대한 현지화 데이터를 포함한다.
- **Image files** - 특정 문화에 대한 현지화의 목적이 아니라면, image현지화는 권장되지 않는다. 이미지에 text를 직접적으로 포함하지 않음으로써, 효율적인 현지화가 가능하다.
- **Video and audio files** - 특정 문화에 대한 현지화의 목적이 아니라면, 권장되지 않는다. 