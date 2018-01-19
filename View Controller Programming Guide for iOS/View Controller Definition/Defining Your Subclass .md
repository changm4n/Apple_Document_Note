# View Controller Definition



### Defining Your Subclass

개발자는 `UIViewController`클래스를 상속하여 자신만의 subclass를 구현한다. 이는 보통 Content View controller 의 형태이다. 즉, 내부 View들과 데이터에 대한 책임을 가지고있다. Container View controller 는 이와 반대되는 특성을 가진다. 이와같은 Custom View controller 를 구현하기 앞서, 다음 항목을 살펴보자.

- 만약 Main View가 Table 의 형태라면, `UITabieViewController`를 사용하라.
- 만약 Main View가 Collection 의 형태라면, `UICollectionViewController`를 사용하라.
- 다른 View controller 에 대해선, UIViewController를 사용하라.

Container View Controller의 경우, 상위 클래스는 기존 컨테이너 클래스를 수정하는지 고유 한 컨테이너 클래스를 작성하는지에 따라 달라진다. 이미 존재하는 컨테이너의 경우, 수정을 원하는 Class를 선택할 수 있다. 새로 생성되는 Container View controller 에 대해서는, `UIViewController`를 상속하여 구현한다.



### Defining Your UI

View controller 에 대한 UI구현은 스토리보드를 활용한다. 코드를 활용하여 작성할 수 있지만, 스토리보드는 View controller 의 내용들을 시각화 해주는 훌륭한 기능이며 View들의 계층 구조또한 관리할 수 있다. 스토리보드에선, 뷰의 이동과 연결 관계를 Segue를 이용한다. 또한, 스토리보드에선 다음의 기능을 활용하여 쉽게 UI를 구성할 수 있다.

- View들을 정렬하고, 배치한다.
- Outlet과 Action을 활용한다.
- View controller 사이에 Segue를 활용하여 서로를 연결한다.
- 여러 Size 클래스에 대응한다.
- View의 interaction을 다루기위해 gesture recognizer를 생성한다.



### Handling User Interactions

앱의 Responder 객체는 이벤트를 수신하고, 적절한 액션을 취한다. View controller 또한 Responder 객체이지만, 이 자체가 직접적으로 이벤트를 처리하는 경우는 드물다. 일반적으로 다음과 같은 과정으로 이벤트를 처리한다.

- View controller 는 높은 수준의 이벤트를 처리하기 위해 액션을 구현한다. 이 액션 메소드는 다음을 처리한다 
  - Special Action - 컨트롤이나 다른 뷰들은 특정 interation을 처리하기 위해 액션을 호출한다.
  - Gesture Recognizer - Gesture Recognizer는 현재 Gesture의 상태를 전달하기 위해 액션을 호출한다.
- 시스템이나 다른 View로 부터의 알람을 탐지한다. 
- View controller 는 다른 Object를 위한 Data Source, Delegate로 이용될 수 있다. - TableView, CollectionView의 Datasource로 활용되기도 한다. `CLLocationManager`의 Delegate로 활용할 수 있다. 

이벤트에 대응하는 것은, view의 내용을 업데이트 시키는 과정을 포함한다. 이러한 과정에선 View에대한 참조가 필요한데, Outlet을 통하여 해결할 수 있다. 



### Displaying Your Views at Runtime

스토리보드는 View controller 상의 View들을 표시하는 과정을 단순화 시킨다. UIKit은 View들을 필요한 시점에  스토리보드에서 호출한다. 그 과정에서, 다음의 절차를 실행한다.

- 스토리보드 파일을 활용하여 View를 초기화 한다.
- Outlet과 Action을 연결한다.
- View controller 의 View들에게 Root View를 지정한다.
- View controller 의 `awakeFromNib()`을 호출한다. - 이 함수가 호출될 때, 특성 컬렉션은 비어있고, View들은 자신의 위치에 배치되어있지 않을 수 있다.
- View controller 의 `viewDidLoad()`를 호출한다. - View들을 추가, 제거하거나 레이아웃을 수정, 데이터를 불러오는 작업을 진행한다.

View들을 화면에 표시하기 전에, UIKit은 그들을 수정할 기회를 제공한다. 

- `viewWillAppear:` View가 곧 표시되는 순간 호출된다.
- layout을 수정한다.
- View들을 화면에 표시한다.
- 표시 후 `viewDidAppear:`가 호출된다.

View의 위치, 크기를 추가, 제거, 수정할 시 View 에 적용된 Constraint를 추가,제거 해주어야 한다. 레이아웃 관련 작업은 UIKit의 레이아웃 관련설정을 지저분하게 할 수 있다. 

View Controller Life Cycle - https://www.codementor.io/hemantkumar434/view-controller-lifecycle-ios-applications-7oyju9lp6

### Managing View Layout

View의 크기나 위치가 변경되면, UIKit은 계층 구조 상에서의 레이아웃을 변화시킨다. Auto layout이 적용된 View 의 경우, 현재 Constraint에 따라 위치, 크기를 변경히킨다.  레이아웃 설정 과정에서, UIKit은 개발자에게 몇가지 포인트를 알려주며, 이를 활용하여 레이아웃 관련 작업을 진행할 수 있다. 

- View Controller 의 특성 컬랙션을 수정하라. 
- View Controller 의 `villWillLayoutSubViews`함수를 호출한다.
- 현재 UIPresentationController 객체의 `containerViewWillLaytoutSubviews`함수를 호출한다.
- `layoutSubviews`함수를 호출한다. 기본적인 구현으로 유효한 constraints의 레이아웃 정보를 계산한다. 그 후에 하위 View 들의 `layoutSubviews` 를 순차적으로 호출한다.
- View에게 계산된 레이아웃 정보를 적용한다.
- View controller 의 `viewDidLayoutSubview`를 호출한다.
- 현재 UIPresentationController의 `containerViewDidLayoutSubviews`를 호출한다.

View Controller 는 `viewWillLayoutSubviews`와 `viewDidLayoutSubview`를 활용하여 추가적인 레이아웃 업데이트를 실행할 수 있다. 레이아웃 설정 이전에, View들을 추가하고 제거하고, 사이즈, 위치를 설정하며, constraints를 업데이트 하거나 다른 속성을 수정한다. layout 후에, 데이터를 리로드하고, 다른 View들의 내용을 수정하거나 최종적인 업데이트를 진행한다.

다음으로는 Layout을 효율적으로 관리하는 Tip이다.

- Auto Layout을 활용하자. 

- top, bottom layout guide의 이점을 활용하자. - 이러한 guide를 활용함으로써, 항상 View들이 보여지도록 설정할 수 있다.  top guide는 status bar, navigation bar를 고려한다. 비슷하게, bottom guide는 tab bar나 toolbar를 고려한다.

- View를 제거, 추가할 때 constraints를 수정하라. 

- View에 animation을 적용할 때, constraints를 제거하라. - UIKit의 CoreAnimation을 활용하여 animation을 적용할 때, constraints를 제거하며 종료 후 추가하여야 한다. 애니메이션이 종료된 후, 알맞은 값을 재설정 해주어야 한다.

  - ```swift
    self.topDistanceConstraint.constant = 200 
    UIView.animate(withDuration: 0.5, animations: {
     self.view.layoutIfNeeded()
     self.topView.alpha = 0.25
    })
    ```




### Managing Memory Efficiently

메모리 할당 부분은 개발자가 결정할 수 있지만, 다음 표는 이러한 작업들을 처리할 수 있는 함수들을 나열한다. 대부분의 할당 해제는 강한 참조를 제거하는 것을 의미한다. 강한 참조를 제거하기 위해선, 프로퍼티나 값들에 `nil`을 할당한다.

| Task                                     | Methods                   | Discussion                               |
| ---------------------------------------- | ------------------------- | ---------------------------------------- |
| Allocate critical data structures required by your view controller. | Initialization methods    | 커스텀한 생성자는 View controller 를 알맞게 배치하여야 한다. 이 함수는 적절한 구동을 위한 자료구조의 할당으로 활용될 수 있다. |
| Allocate or load data to be displayed in your view. | `viewDidLoad`             | 표시하기 위한 데이터를 할당하기 위하여 이 함수를 사용한다. 이 함수가 호출되는 순간. View 는 알맞은 위치에 할당되어 있어야 한다. |
| Respond to low-memory notifications.     | `didReceiveMemoryWarning` | 중요하지 않은 객체들을 해제하는 용도로 사용하여야 한다.          |
| Release critical data structures required by your view controller. | `dealloc`                 | 이 함수는 별도의 최적화를 구현할 시에만 활용한다. 시스템은 자동으로 객체를 해제한다. |























