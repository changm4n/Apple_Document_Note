# Strategies for Implementing Specific App Features

각 feature들을 앱에 적용하는 방법에 대하여 알아본다.



## Privacy Strategies

개인 정보를 보호하는 것은 아주 중요하다.  개인 정보란 유저 데이터, 사용자 식별 정보 등을 포함한다. 프레임워크들이 제공되어 이러한 정보를 보호할 수 있다.

### Protecting Data Using On-Disk Encryption

데이터의 암호화, 복호화를 통한 정보 보호는 내장된 하드웨어를 활용하여 이루어진다. 잠금 상태에선 App이 직접 만든 파일일지라도, 접근이 불가능하다. 반드시 기기를 unlock하여야 해당 파일에 접근이 가능하다.

기기는 다음의 사항을 준수하여야 정보 보호가 이루어진다.

- 사용자 기기의 파일시스템은 정보보안을 지원하여야 한다.
- 사용자 기기는 암호 잠금이 설정되어 있어야 한다.



`NSData`,  `NSFileManager` 클래스를 활용하여 다음에 나열된 수준의 정보 보안 수준을 적용할 수 있다.

- No protection - 파일은 암호회 되었지만, 기기 잠금 시 비밀번호로 보호되지 않는다. `NSDataWritingFileProtectionNone`,  `NSFileProtectionNone` 옵션으로 적용한다.
- Complete - 파일은 암호화 되어있으며, 기기가 잠겨있을 시 접근이 제한된다.  `NSDataWritingFileProtectionComplete` ,`NSFileProtectionComplete`옵션으로 적용한다.
- Complete unless already open - 파일은 암호화 되었으며, 닫힌 파일은 기기 잠금시 접근할 수 없다. 잠금 해제 시 접근 가능하며, App이 최초 파일 접근 이후, 기기가 잠겨있더라도 접근이 가능하다. `NSDataWritingFileProtectionCompleteUnlessOpen`,`NSFileProtectionCompleteUnlessOpen`옵션으로 적용한다.
- Complete until first login - 파일은 암호화 되었으며, 기기 잠금시 접근할 수 없다. 최초로 기기가 켜지고, 잠금이 해재된 이후 접근이 가능하다. `NSDataWritingFileProtectionCompleteUntilFirstUserAuthentication` , `NSFileProtectionCompleteUntilFirstUserAuthentication` 옵션으로 젹용한다.

파일을 암호화 한다면, 파일에 접근이 제한될 수 있음을 대비하여야 한다. 다음의 방법으로 파일의 암호화 수준의 변화를 관찰할 수 있다.

- `AppDelegate`에서 `applicationProtectedDataWillBecomeUnavailable:` , `applicationProtectedDataDidBecomeAvailable:`함수를 구현한다.
- 객체에서 `UIApplicationProtectedDataWillBecomeUnavailable` ,`UIApplicationProtectedDataDidBecomeAvailable` 알람을 등록한다.
- shared `UIApplication`의 `protectedDataAvailable`값을 참고한다.

새로운 파일 생성 시, 데이터를 쓰기 전 파일 암호화를 적용할 것을 권장한다. `writeToFile:options:error:`  를 사용시 자동으로 적용된다.



### Identifying Unique Users of Your App

특정 사용자를 구분하여 적절한 서비스를 제공할 수 있도록 iOS는 해당 기술을 지원한다. 

**NOTE : 사용자를 식별할 때, 그 의도를 명확히 밝혀야 하며, 비밀리에 사용자 정보를 추적하는 것은 허용되지 않는다.**

각 상황별로 유저 정보에 접근할 수 있는 방법들이 존재한다.

- 사용자를 서버의 계정과 연결한다. - 로그인 화면을 제공하여 사용자가 정보들을 입력한다. 수집된 정보는 암호화하여 처리한다.
- 다른 기기에서 동작하는 앱과 구분한다. - `UIDevice`의  `identifierForVendor`  를 이용한다. 이 정보는 특정 사용자를 구분할 순 없다. 이 값은 App Store에서 부여하며, App Store에서 다운로드 된 App이 아니라면, bundle ID를 활용하여 생성된다. 사용자가 기기의 모든 App을 삭제하고 하나의 앱을 재설치 시 값은 변경된다.  Xcode의 test build를 설치하거나 , ad-hoc 배포를 이용시 변경될 수 있다.
- 광고를 목적으로 사용자 구분 -  광고를 위한 상황 시, `advertisingIdentifier` 활용이 권장된다.

한 사용자는 다양한 기기를 사용할 수 있기 때문에, Apple은 다양한 기기에서 같은 사용자를 구분하는 정보를 제공하지 않는다. 사용자 구분을 원한다면, 직접 구현한 기능을 활용하여야 한다.



## Respecting Restrictions

사용자는 App에서 재생되는 미디어의 등급을 제한할 수 있다. 미디어를 사용하는 App이라면, 제한사항을 반영하고, 변화에 대응하여야 한다. 현재 설정된 값들을 가져오기 위해선,  `standardUserDefaults`를 이용한다. 이 객체로부터 다음의 값들을 불러와 조회할 수 있다.

| Key                                      | Value                                    |
| ---------------------------------------- | ---------------------------------------- |
| com.apple.content-rating.ExplicitBooksAllowed | Books 들이 제한된다.                           |
| com.apple.content-rating.ExplicitMusicPodcastsAllowed | 음악, 영상, 팟케스트가 제한된다.                      |
| com.apple.content-rating.AppRating       | NSNumber 값이며, 0~1000의 범위를 갖는다. 해당 값보다 큰 값을 가진 App은 제한된다. |
| com.apple.content-rating.MovieRating     | NSNumber 값이며, 0~1000의 범위를 갖는다. 해당 값보다 큰 값을 가진 영상은 제한된다. |
| com.apple.content-rating.TVShowRating    | NSNumber 값이며, 0~1000의 범위를 갖는다. 해당 값보다 큰 값을 가진 TV Show는 제한된다. |

만약 값이 nil로 반환된다면, 제한사항이 정의되지 않은 것 이다. 

`NSUserDefaultsDidChangeNotification` 의 알람을 통해서 변화를 탐지할 수 있다. 

| Rating name | value |
| ----------- | ----- |
| 4+          | 100   |
| 9+          | 200   |
| 12+         | 300   |
| 17+         | 600   |

영화와 TV 의 값은 국가별로 차이가 있을 수 있다. 미디어를 포함하지 않는 앱일지라도, 제한사항을 적용할 수 있다.



## Supporting Multiple Versions of iOS

여러 버전의 iOS를 지원하는 App은, 이전 버전의 iOS에서 최신버전의 API를 사용하지 않도록 런타임 체크를 해야한다. 그 방법은 2가지가 있다.

- 해당 클래스가 존재하는지 확인한다.  해당 타입의 class 객체가 nil을 가진다면, 사용할 수 없다.
- 해당 매소드 지원여부를 확인하기 위해`instancesRespondToSelector:` ,`respondsToSelector:`를 사용할 수 있다.
- C-base 함수의 사용가능 여부를 확인하기 위해선, 함수 이름과 NULL 값을 비교한다. 



## Preserving Your App’s Visual Appearance Across Launches

Background상태의 App은 system의 메모리 관리 정책에 따라 불시에 종료될 수 있다. 사용자는 App이 일시정지 되었다고 생각하기 때문에, 이전의 상태를 복구하는 것이 좋다. 

UIKit이 이러한 복구 system을 지원한다. App의 콘텐츠를 이해한다면, 복원에 필요한 코드만 작성할 수 있다. UI를 업데이트 한다면, 기존의 콘텐츠를 새로운 콘텐츠로 연결하는 방법을 알아야 한다.



상태 복원 시 3가지를 고려해야 한다.

- `AppDelegate` - App의 최상위 상태를 관리한다.
- Viewcontroller Object - 전반적인 UI의 상태를 관리한다.
- Custom view - 보존되어야 할 데이터를 가지고 있다.

### Enabling State Preservation and Restoration in Your App

상태 저장과 복구는 자동으로 이루어지지 않는다. `AppDelegate`에서 다음의 함수를 구현하여 적용할 수 있다. `application:shouldSaveApplicationState:`,`application:shouldRestoreApplicationState:` 

위 함수의 리턴 값을 설정하여 저장,복원을 설정할 수 있다.

### The Preservation and Restoration Process

App이 복원되어야 할 객체를 지정한다면, UIKit은 적당한 시점에 객체를 저장, 복원한다. 

UIKit은 적절한 시점에(foreground,background진입) 상태를 저장한다. UIKit이 새로운 상태에 대한 업데이트가 필요하다면, 저장되어야 할 view,viewcontroller를 살핀다. 저장이 필요한 데이터는 디스크의 파일에 쓰고, 다음번 App이 실행될 때, UIKit이 복원한다. 암호화된 파일이므로, 기기가 잠금해제 되었을 때 발생한다.

복원의 과정에서, UIKit은 저장된 복원 데이터를 활용하지만, 실제 객체 생성은 코드로 이루어진다. 어떤 객체가 생성되어야 하고, 이미 존재하는지는 코드만이 알 수 있다. 

App은 이 과정들에서 다음의 의무를 가진다.

**Preservation**

- UIKit에게 상태 저장을 지원함을 알려야한다.


- UIKit에게 저장되어야 할 view, viewcontrollerf을 알린다.
- 저장될 객체의 데이터를 인코딩한다.

**Restoration**

- UIKit에게 상태 복원을 지원함을 알려야한다.
- UIKit에게 요청받은 객체를 생성한다.
- 복원을 위해 저장된 데이터를 디코딩한다.

UIKit에게 저장,복원이 되어야 할 객체를 알려주는 것이 가장 중요하다. 어떤 viewcontroller들은 story board에 의해 자동으로 생성되지만, 다른 탭에서 생성되거나 push되는 객체들은 별도의 처리 없인 생성되지 않는다.  

*restoration identifi*er를 지정한 객체는 UIKit이 저장한다. 저장의 과정에서, UIKit은 전체 계층 구조를 탐색하며 *restoration identifi*가 저장된 객체를 저장한다. 상위 객체가 *restoration identifi*를 갖지 않는다면, 하위객체들은 저장되지 않는다. 다음 번 App이 실행될 때, `application:willFinishLaunchingWithOptions:`를 호출하여 이전 상태를 복원한다. 



#### Flow of the Preservation Process

데이터 저장이 이루어지기 전에, UIKit은 `AppDelegate`에게 `application:shouldSaveApplicationState:`를 통하여 저장이 이루어져야 하는지 묻는다. YES를 반환한다면, 데이터를 수집하여 저장한다.

//사진



App이 다시 실행된다면, 저장된 파일들을 찾아 표시한다. 이때의 데이터들은 실행 시 상태복원에만 필요하므로, 완료 시 제거된다. 이 데이터는 복원,저장 과정에서 오류 발생시에도 제거된다.

#### Flow of the Restoration Process

기본적인 초기화와 UI로드가 완료된다면, UIKit은 `AppDeleaget`에게 복원이 일어나야 하는지  `application:shouldRestoreApplicationState:`를 통해 묻는다. 이 함수에서 복원이 가능한지 점검하며, 복원될 viewcontroller의 참조를 가진다. 

//사진



UIKit은 자동으로 복원을 진행하지만, 객체 사이의 관계에 대해서는 복원하지 않는다. 대신, 각 viewcontroller들은 이전의 상태로 돌아가기 위한 정보를 저장하여야 한다. 예로 navigation controller는 stack의 view controller들을 저장한다. 



이러한 상태 저장,복원시 과정이 진행되는 동안의 UI처리도 필요하다. 



### What Happens When You Exclude Groups of View Controllers?

restoration identifier가 지정되지 않는 Viewcontroller와 그 하위 객체는 저장되지 않는다. 이 viewcontroller를 저장하지 않는다고 하더라도, 이 계층구조에서 사라지게 해선 안 된다. 

view controller가 저장되지 않았더라도, 객체에 대한 참조를 저장할 수 있다. 상위 뷰가 restoration identifier를 갖지 않더라도, 그 참조를 저장한다면, 저장된다. 



### Checklist for Implementing State Preservation and Restoration

상태 저장, 복원을 지원하기 위해선 각 객체를 인코딩,디코딩 할 수 있도록 해야한다. 아래의 항목을 확인하자.

- (필수) `application:shouldSaveApplicationState:`, `application:shouldRestoreApplicationState:`를 구현한다.
- (필수) 각 객체의 restorationIdentifier를 지정한다.
- (필수) `application:willFinishLaunchingWithOptions`에서 window를 표시한다. 복원할 스크롤 위치, 관련 정보를 복원한다. 
- 적절한 view controller에게 restoration classes를 지정한다. 
- (권장)  `encodeRestorableStateWithCoder:`,`decodeRestorableStateWithCoder:` 를 이용하여 객채를 인코딩, 디코딩 한다.
- `application:willEncodeRestorableStateWithCoder:` ,`application:didDecodeRestorableStateWithCoder:` 를 이용하여 상태나 정보들을 저장한다.
- collectionview,tableview의 datasource가 되는 객체는  `UIDataSourceModelAssociation`프로토콜을 상속한다.

### Enabling State Preservation and Restoration in Your App

앱 상태 저장,복원은 자동으로 이루어지지 않는다. `application:shouldSaveApplicationState:` `application:shouldRestoreApplicationState:` 두 함수를 구현해야 한다.

두 함수의 return값을 YES로 함으로써, 저장, 복원을 지정할 수 있지만, 조건적인 구현을 위해선 NO를 반환하여 처리하도록 한다. 

#### Marking Your View Controllers for Preservation

객체를 저장하기 위해, 적절한 `restorationIdentifier` 를 선택하는게 중요하다. 모두 다른 타입의 객체라면, 타입 명으로 지정하여도 된다. 

restoration path 는  restorationID의 연속으로 이루어진다. 이 값은 유일해야한다. 

#### Restoring Your View Controllers at Launch Time

UIKit은 복원 과정에서, 저장된 UI를 활용하여 객체를 생성한다. 그 과정에서 다음의 절차를 따른다.

- view controller가 restoration class를 가지고 있다면, UIKit은 그 class 에게 view controller를 제공할지 묻는다.  `viewControllerWithRestorationIdentifierPath:coder:`를 호출하여 view controller를 가져혼다. nil을 반환한다면, 그 과정을 멈춘다.
- view controller가 restoration class를 가지고 있지 않다면, `AppDelegate`에게 view controller를 제공하지 묻는다.  `application:viewControllerWithRestorationIdentifierPath:coder:`를 호출하여 위와 같은 과정을 진행한다.
- 올바른 restoration path가 존재한다면, UIKit은 그것을 이용한다. 
- View controller가 story board를 활용하여 생성되었다면, UIKit은 저장된 storyboard를 사용하여 배치, 생성한다.

restoration class를 활용한다면, `viewControllerWithRestorationIdentifierPath:coder:`함수의 선언을 통하여 새로운 객체를 생성하고, 초기화, 결과 객체를 반환하여야 한다. 



새로운 view controller를 생성할 때 새로운 restoration identifier를 지정하는 것은, 좋은 습관이다. restoration identifier를 저장하는 가장 간단한 방법은, identifierComponents 의 마지막 원소를 지정하는 것이다.



#### Encoding and Decoding Your View Controller’s State

저장될 객체에 대하여 UIKit은 각 객체의 `encodeRestorableStateWithCoder : `를 호출하여 저장의 기회를 제공한다. 디코딩 과정에선, `decodeRestorableStateWithCoder : `함수가 호출되어 객체에 적용된다. 이 함수의 구현은 권장되며 다음의 정보를 저장할 수 있다.

- 표시될 정보에 대한 참조값
- 부모 객체에서의 자식 객체에 대한 참조
- 현재 선택사항에 관련된 정보
- 사용자 구성이 가능한 view에서의 구성 정보

NSCoding 프로토콜을 상속하는 모든 객체를 인코딩, 디코딩 할 수 있다. view나 view controller 객체에선 coder가 restoration id를 저장하고, 그 객체를 저장가능한 상태로 지정한다. 

 `encodeRestorableStateWithCoder:` , `decodeRestorableStateWithCoder:` 함수 호출시 반드시 super객체의 함수를 호출하여야 한다.



coder객체는 인코딩, 디코딩 과정에서 공유되지 않는다. 저장 가능한 상태의 객체들은 고유의 coder를 받으며 이를 데이터 저장,읽기에 활용한다. unique한 coder의 사용은 namespace 충돌의 걱정을 없앤다. 하지만, 특별한 이름의 key names 사용은 피해야 한다.

### Preserving the State of Your Views

view가 저장해야할 정보를 가진다면, view controller와 함께 그 상태를 저장할 수 있다. 보통 view들은 그들을 포함하는 view controller에 의해 구성되므로, 대부분은 상태정보를 저장할 필요가 없다. view의 상태를 저장해야하는 순간은, 그것이 가진 데이터나 view controller에  독립적인 환경에서 사용자에 의해 변화되는 순간이다. 예로 scroll view 의 position이 있다.

view의 상태를 저장하기 위해서는 다음을 준수하여야 한다.

- view의  `restorationIdentifier`를 지정한다.
- `restorationIdentifier`가 지정된 view controller의 view를 사용한다.
- table view, collection view의 datasource에는  `UIDataSourceModelAssociation` protocol을 적용한다.

#### UIKit Views with Preservable State

대표적으로 다음의 UIKit view들은 상태를 저장할 수 있다.

- UICollectionView
- UIImageView
- UIScrollView
- UITableView
- UITextField
- UITextView
- UIWebView

#### Preserving the State of a Custom View

상태를 저장해야할 custom view가 있다면, `encodeRestorableStateWithCoder:`, `decodeRestorableStateWithCoder:` 함수를 활용하여 인코딩,디코딩할 수 있다. 이 함수를 이용하여 다른 목적으로 변환될 수 없는 데이터를 저장한다. view에 표시되는 데이터나, view controller로 쉽게 설정될 수 있는 데이터는 표시하지 않는다.

#### Implementing Preservation-Friendly Data Sources

collection View 나 table View의 datasource가 `UIDataSourceModelAssociation` 프로토콜을 상속하다면, 선택이나 표시되는 cell에 관한 정보를 저장할 수 있다. 

`UIDataSourceModelAssociation` 를 적용하기 위해선, 후에 App이 시작되었을 때의 상태와 구별이 가능해야 한다. 데이터에 지정하는 id값은 불변해야함을 의미한다. 

CoreData를 사용하는 App은 이 프로토콜을 효과적으로 사용할 수 있다. CoreData에 저장된 각 객체들은 unique한 id를 가지고있으므로, 이를 활용하여 시간이 지난 후에 다시 데이터를 불러와 배치할 수 있다.

### Preserving Your App’s High-Level State

UIKit은 view나 view controller에 저장되는 데이터 왜에도 App의 다양한 데이터의 저장을 지원한다.  `UIApplicationDelegate`의 `application:willEncodeRestorableStateWithCoder:` `application:didDecodeRestorableStateWithCoder:`두 함수로 구현 가능하다.



### Tips for Saving and Restoring State Information

- App의 버전과 상태를 함께 인코딩한다.
- 상태 정보에 데이터 모델 객체를 포함하지 않는다.
- 상태 저장은 설계된 방식 대로 view controller를 사용하길 기대한다.
- 모든 view controller를 저장하는 것은 원하는 바가 아닐 수 있다.
- 복원 시점에서 view controller 객체를 교체하면 안 된다.
- 사용자가 종료를 강제한다면, 저장된 정보는 삭제된다.

## Tips for Developing a VoIP App

iOS8이상에서, APNs와 PushKit Framework를 활용해 VoIP App을 개발할 수 있다. 푸시 알람을 활용하면, VoIP서비스를 위해 지속적인 네트워크 연결,소캣을 구성할 필요가 없다. 

다른 음악 App처럼, audio session또한 적절히 설정되어야 한다. playback과 recording은 항상 이루어지지 않기 때문에, 필요한 시점에서만 활성화 해야한다. VoIP App이 확인해야할 항목들은 다음과 같다.

- VoIP Background 모드를 활성화한다.
- PuchKit API를 활용하여 VoIP알람 수신을 처리한다.
- Audio session의 사용과 종료를 처리한다.
- 더 나은 UX를 위해, Core Telephony framework를 활용하라.
- 개선된 VoIP App을 제공하기 위해, System Configuration framework를 활용하여 네트워크 변경을 감지하고, 가능한 많이 sleep 상태를 유지하라.
- 알람 서버에 VoIP를 연결할 수 있도록 VoIP Services certificate를 요청하라.



### Using the Reachability Interfaces to Improve the User Experience

VoIP서비스는 네트워크 의존도가 높기 때문에 `System Configuration framework`의 `reachability interface`를 사용하여야 한다. 변경사항을 감지하여 사용자에게 VoIP 연결 상태에 대하여 알려주어야 한다.

- 원격 호스트를 위해 `SCNetworkReachabilityRef` 를 생성하라.
- `SCNetworkReachabilityRef` 의 콜백 함수를 정의하여 타겟의 접근 상태 변화를 처리하라.
- `SCNetworkReachabilityScheduleWithRunLoop` 함수를 활용하여 타겟을 run loop에 할당하라.

네트워크 상태를 감지해 App을 조정한다면, 배터리 사용을 개선할 수 있으며, Sleep 상태를 오래 유지할 수 있다.