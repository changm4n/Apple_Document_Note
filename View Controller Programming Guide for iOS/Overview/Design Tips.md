# Design Tips

View controller는 앱 구성에서 필수적이다. 다음의 항목을 준수하며 View controller를 활용하자.



### Use System-Supplied View Controllers Whenever Possible

많은 iOS Framework들이 View controller를 활용할 수 있도록 제공한다. 

이러한 형태로 제공되는 View controller 들은 각자의 역할을 다하도록 디자인되어있다. Custom View controller 를 디자인하기 전에, 기존의 frameworks를 참고하여 활용하자.

- 경고, 사진 또는 동영상 촬영, iCloud 파일 관리를 지원하는 View controller 를 제공한다. 
- GameKit 은 사용자 매칭, 리더보드 관리, 성과등 여러 게임 내 기능들을 제공한다.
- 연락처 UI framework는 사용자의 연락처를 선택하여 다룰 수 있도록 제공한다.
- MediaPlayer framework는 영상 재생, 관리 등 사용자의 영상을 관리할 수 있도록 제공한다.
- EventKit은 달력 정보를 활용하도록 제공한다.
- GLKit은 OpenGL rednering을 제공한다.
- Multipeer Connectivity 는 다른 사용자를 탐지하고, 연결로 초대하는 기능을 제공한다.
- Message UI는 이메일과 sms전송 기능을 제공한다.
- PassKit은 Passbook관리와 Pass 추가 기능을 제공한다.
- Social framework는 Twitter, Facebook, 등 여러  social media 기능 사용을 제공한다.



중요 : 시스템이 제공하는 View controller 의 구조를 수정하면 안 된다. 각각 고유의 구졸르 가지고 있으며, 구조의 무결성을 유지한다. 구조를 변화시킴으로서 버그를 야기할 수 있고, 올바른 동작을 막을 수 있다. 



### Make Each View Controller an Island

View controller 는 항상 독립적이어야 한다. 다른 View controller 에 대한 내부 작동이나 구조를 알고 있어서는 안 된다. 두 View controller 가 통신이 필요한 상황이라면, 서로 독립된 인터페이스를 활용하여야 한다.

delegation 디자인 패턴이 흔이 사용된다. delegation을 이용하여, 한 객체가 통신을 위한 protocol 을 선언하고, 다른 객체는 protocol을 준수함을 선언한다. delegate 오브젝트의 타입은 무의미하다. protocol의 메소드를 구현하는 것이 중요하다.



### Use the Root View Only as a Container for Other Views

내부 컨텐츠의 container로써 Root View를 사용하라. 이렇게 구현함으로써, 모든 View 들에게 동일한 부모 view를 지정할 수 있다. Auto Layout 구현 시 공통의 부모 View가 필요하다.



### Know Where Your Data Lives

MVC 디자인 패턴에서,  View controller의 역할은 모델과 view사이에 데이터 흐름을 관리하는 것 이다. View controller 는 데이터를 임시 변수에 저장하고, 인증 과정을 실행할 것 이다. 하지만 이것의 주된 목적은 View들이 알맞은 정보를 가지고 있는지 확인하는 것 이다. Data객체의 역할은 그 정보가 올바른 정보인지 무결성을 보장하는 것 이다.

이러한 구조는 `UIDocument`와  `Viewcontroller `의 관계로 살펴볼 수 있다. 기본적으로 존재하는 관계는 없지만, `UIDocument`는 새로운 데이터를 호출하고 저장하며, `Viewcontroller`는 View들의 표현을 관리한다. 두 사이에 관계를 형성한다면, View controller 는 오직 성능향상을 목적으로 캐싱 데이터만 소유할 수 있다. 



### Adapt to Changes

앱은 다양한 기기에서 실행될 수 있다. View controller 는 다양한 크기에 대응할 수 있도록 디자인되었다. 여러 View controller 를 활용하여 대응하기보단, 내장된 기능을 활용하여 크기 변화에 대응하는 것이 좋다. UIKit이 크기 변화에 따라 Notification을 전송한다. 이를 활용하여 대응할 수 있다.