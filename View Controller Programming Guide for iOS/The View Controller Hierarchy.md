# The View Controller Hierarchy

각 View controller 의 관계는 서로의 용도를 정의한다. 적절한 관계를 유지한다면, 필요한 순간에 예정된 동작들이 알맞은 View controller 에게 전달되도록 한다. 이 관계에 문제가 생기면, 올바르게 작동하지 않는다.



### The Root View Controller

Root View controller 는 모든 계층 관계에서의 기준점이 된다. 모든 윈도우는 하나의 Root View controller 를 가지고 있다. Root View controller 는 사용자에게 처음 표시될 내용을 정의한다. Window 자체는 컨텐츠를 표시할 수 없으므로, View controller 의 view가 컨텐츠를 표시한다.

Root View controller 는 Window 객체의  `rootViewController` 프로퍼티로 접근할 수 있다. StoryBorad를 사용중이라면, UIKit 이 스스로 설정한다.  코드로 작성하였다면, 별도로 지정해주어야 한다.

### Container View Controllers

Container View controller 를 사용하면, 보다 편하게 재사용 가능한 뷰들을 관리할 수 있다. Container View controller는 여러 자식 뷰들을 합쳐 새로운 인터페이스를 구성한다. 그 예로, `UINavigationController`는 자식 뷰로부터 컨텐트를 불러와 네비게이션 바, 툴바와 함께 표시한다. 이러한 종류로는 `UINavigationController`,`UISplitViewController`,`UIPageViewController`가 있다.

Container View controller는 주어진 공간을 체우는 역할을 한다. 보통 Window로 부터 Root View controller 의 역할을 하며, Modal한 형태로 표현될 수 있다. Container는 자신이 포함하는 자식들의 포지셔닝을 담당해야 한다. 내부에 배치되는 자식 뷰들은, Container View controller 에 속해있더라도, Container View controller 와 자기 형제 뷰에대한 최소한의 정보만 가질 수 있다.



### Presented View Controllers

새로운 View controller를 표시함으로써, 기존의 View controller의 내용을 숨긴다. 이러한 방식은 새로운 View controller 를 표현하는 일반적인 방법이다. View controller 를 표시하면, UIKit은 기존의 View controller 와 새로이 표시된 View controller 사이의 관계를 설정한다. 이러한 관계는 View controller  간의 계층관계를 형성하며, 새로운 View controller 를 runtime에 추가하는 방법이 된다.



//사진



Container View controller 가 포함된다면, UlKit은 표현 chain을 수정하여 코드를 단순화시킨다. View controller 를 표현하는 방법에 따라 서로 다른 규칙을 가진다. 예로, Full-screen 표현은 화면 전체를 덮는다. View controller 를 표시할 때, UIKit은 적절한 context를 제공한다. 대부분의 경우엔, 제일 가까운 Container View controller 를 선택하지만, Window의 Root View controller 를 선택할 수 있다. 가끔의 경우엔 직접 표현 context를 지정하여 UIKit에게 알려줄 수 있다.



//사진



전체 화면 표시를 실행하면, 표현되는 View controller 는 화면 전체를 덮어야 한다. 자식 뷰에게 Container 의 bounds를 알도록 하기 보단, container가 그 표현을 담당할지 결정한다.  다음 예에선, NavigationController가 전체 화면을 덮음으로, Presenting View controller 의 역할을 하며, 표현을 담당한다.









