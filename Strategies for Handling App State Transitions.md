//보충가능 체크

# Strategies for Handling App State Transitions

iOS App은 여러 상태를 가지며, 상태별로 특성을 갖는다. 다른 상태로 전환 시 system은 `app object`로 상태 전환을 알려, 개발자는  `UIApplicationDelegate`의 methods를 통해서 처리할 수 있다. 



## What to Do at Launch Time

App이 시작되는 시점에선  `application:willFinishLaunchingWithOptions:` 와 `application:didFinishLaunchingWithOptions:` 함수가 호출된다. 이 두 함수의 역할은 다음과 같다.

- App의 시작 옵션들이 담긴 Dictionary를 통하여 어떻게 App이 실행되었는지를 확인, 대응한다.
- App의 자료구조를 초기화한다.
- App이 나타낼 Window와 View를 준비한다. //보충가능

App이 시작되는 시점에서 system은 Main storyboard를 호출하고, 시작 Viewconteroller를 호출한다. 상태복구이 필요한 App의 경우, 위의 두 함수가 호출되는 사이에 복구가 이루어져야 한다. `application:willFinishLaunchingWithOptions:`함수로 복원할 window를 설정하며,

`application:didFinishLaunchingWithOptions:` 함수로 최종 UI보정을 실행한다.



### The Launch Cycle

App이 실행되면, App은 정지 상태에서 활성화 또는 백그라운드 상태로 진입한다. 

실행 과정이 foreground 상태로 이어진다면, 이러한 시작과정에서 App은 process를 생성하며 main function을 호출하기 위한 main thread를 생성한다. 



//사진첨부



App이 실행되어 백그라운드 상태로 진입한다면, 이전에 소개한 과정과는 다르게, event를 처리하며 window엔 표시되지 않지만, App의 UI File들을 로드한다. 



이 두 상태는 `application:willFinishLaunchingWithOptions:` 와  `application:didFinishLaunchingWithOptions:` 두 함수의 `UIApplication` 객체의 `applicationState` 프로퍼티로 확인할 수 있다.

시작시점에서 URL Request가 호출된다면, 위의 두 설명과는 다른 과정을 진행한다.



### Launching in Landscape Mode

일반적으로 App은 portrait 모드로 실행되며, 기기의 회전에 맞게 UI를 변화시킨다. App이 오직 Landscape 모드만을 사용한다면, system 에게 별도로 안내하여야 한다. 

- info.plist에 `UIInterfaceOrientation`키를 추가하여  `UIInterfaceOrientationLandscapeLeft` 나`UIInterfaceOrientationLandscapeRight`값으로 설정한다.
- view 구성을 Landscape로 진행한다.
- `shouldAutorotateToInterfaceOrientation:`  함수의 값을 설정하여 가로모드가 좌,우로 회전하도록 설정한다.



