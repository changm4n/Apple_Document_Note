# Inter-App Communication

App들은 서로 간접적으로 통신한다. AirDrop, URL scheme,Document Picker을 이용하여 구현 가능하다.



## Supporting AirDrop

AirDrop을 이용하여 사진, 문서, URL, 기타 여러 데이터를 근거리 통신으로 전송할 수 있다. 



### Sending Files and Data to Another App

`UIActivityViewController` 을 이용하여 activity sheet를 표시하여, AirDrop을 이용할 수 있다. 전송 가능한 데이터에는 사진, 문서, URL등이 있으며, 특별한 데이터는  `UIActivityItemSource`프로토콜을 상속한 object형태로 전송 가능하다.



Activity view를 표시하면, 자동적으로 전송하려는 데이터에 알맞은 형태를 보여준다. 

```swift
let activity = UIActivityViewController(activityItems: [object], applicationActivities: nil)
present(activity, animated: true, completion: nil)
```



### Receiving Files and Data Sent to Your App

AirDrop으로 데이터를 수신하기 위해서는, 다음의 항목을 만족하여야 한다.

- Xcode에서, 허용하는 파일의 형태를 선언한다.
- AppDelegate에서,  `application:openURL:sourceApplication:annotation:` 함수를 구현한다. 

데이터가 수신되면, App이 실행되며 위 함수가 호출된다. App이 foreground라면 위 함수를 이용하여 파일을 열고 표시하여야 한다. background라면, 추후에 파일을 열 수 있다. AirDrop은 파일을 암호화하여 전송하기 때문에 기기가 잠겨있다면 열 수 없다.

App은 파일을 열고 지울 권한을 가진다. 쓰기 권한은 갖지 않는다. 파일을 수정하길 원한다면, 기존의 위치로부터 파일을 옮긴 후 수정하여야 한다. 수정 후엔, 원본 파일을 삭제하는 것을 권장한다.



## Using URL Schemes to Communicate with Apps

URL scheme을 활용하면, 개발자가 정의한 protocol 방식으로 데이터 교환이 가능하다. 올바른 형태의 URL을 system에게 요청하여 실행할 수 있으며, custom scheme을 사용 시 그 내용을 명시하여야 한다.

Apple은 http, mailto, tel, sms에 대한 기본적인 URL scheme은 정의되어 있다. 이는 수정할 수 없으며 custom scheme과 중복 시 Apple에서 제공되는 App이 우선 실행된다. 



### Sending a URL to Another App

URL Scheme을 통하여 데이터를 전송하기 위해선, 올바른 URL을 작성하여 `openURL:`을 호출한다.  호출 시 control 은 새 App으로 넘어가며 App이 실행된다.



### Implementing Custom URL Schemes

App이 custom URL을 수신할 수 있으므로, 해당 URL을 등록해놓아야 한다. 



#### Registering Custom URL Schemes

`Info.plist`의 `CFBundleURLTypes`key의 값으로 URL을 등록한다. 이 값이 다른 App과 중복된다면, 선택할 수 있는 과정은 존재하지 않는다.



#### Handling URL Requests

Custom URL scheme을 가진 App은 해당 URL요청 시 그 요청을 처리하여야 한다. 요청은 `AppDelegate`에서 수신되며,  `application:willFinishLaunchingWithOptions:` , `application:didFinishLaunchingWithOptions:` 함수를 활용하여 처리할 수 있으며 return 값을 YES로 하여 App을 실행할 수 있다.  실행 후에, `application:openURL:sourceApplication:annotation:` 함수를 활용하여 파일을 열 수 있다.

//사진 6-1



App이 실행 중 이지만, Background 상태라면, Foreground로 변경된다. 변경 후에, `application:openURL:sourceApplication:annotation:` 를 호출하여 URL을 확인, 실행한다. 

//사진 6-2

**NOTE: URL scheme을 통한 실행 시, 다른 launch image를 표시할 수 있다.** 

전송되는 url은 `NSURL` 객체로 수신되므로, 여러 URL 형식에 맞는 옵션을 사용할 수 있다.



### Displaying a Custom Launch Image When a URL is Opened

Lauch Image의 파일명을 `*<basename>-<url_scheme><other_modifiers>.png`로 지정하여 URL Scheme을 통한 App실행 시 표시할 수 있다.

`











