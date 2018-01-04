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




### Installing App-Specific Data Files at First Launch

App 동작에 필요한 데이터, 설정파일들은 첫 실행 시점을 활용하여 설정이 가능하다.

앱의 고유한 데이터들은 `Library/Application Support/*<bundleID>/*`  경로에 위치해야 하며, iCloud 폴더, `Documents`폴더를 이용하는 방법도 존재한다. 만약 수정을 원하는 파일이 bundle에 존재한다면,  `Application Support`폴더와 같은 sandbox환경으로 옮긴 후 수정하여야 code sign 문제를 피할 수 있다.



## What to Do When Your App Is Interrupted Temporarily

인터럽트가 발생하면, 일시적으로 App에 대한 제어를 잃게된다. App이 foreground에 존재하더라도, touch event를 받을 수 없다. 이러한 상황을 위하여, `applicationWillResignActive:`와 같은 함수를 이용하여 다음의 항목을 실행하여야 한다.

- data, state를 저장한다.
- Timer와 같은 주기성 작업을 종료한다.
- Metadata 쿼리들을 종료한다.
- 새로운 작업을 실행하지 않는다.
- 영상 재생을 중지한다.
- 게임 App이라면, 일시정지 상태로 진입한다.
- OpenGL ES Frame rate를 조절한다.
- dispatch queue, operation queue들을 중지한다.



앱이 다시 active 상태로 돌아온다면, `applicationDidBecomeActive:`  함수를 통하여 이전의 상태로 돌아갈 수 있다. `NSFileProtectionComplete`로 보호되는 파일을 사용중인 App이라면, 화면이 잠겨있는 동안  파일에 대한 참조를 해제하여야 한다.  암호가 설정된 기기라면, 화면을 잠구는 동작은 system에게 존재하는 decryption key를 해제하도록 하여야 한다.

### Responding to Temporary Interruptions

전화와 같은 인터럽트가 발생한다면, App은  Inactive 상태로 전환되며, 사용자의 선택을 기다린다. 그 선택에 따라 App은 foreground, background로 이동할 수 있다. 

배너를 표시하는 인터럽트는 App을 비활성화 시키지 않는다. 하지만 사용자가 배너를 당겨 알람 센터를 연다면, App은 Inactive 상태가 되며, 배너를 제거할 때까지 대기한다. 

잠금버튼을 누르는 경우, App은 background로 진입하며, event또한 받을 수 없다. 



## What to Do When Your App Enters the Foreground

App이 foreground로 돌아간다면, 이전에 background에 진입 시 중지했던 작업들을 재개할 수 있다. `applicationWillEnterForeground:`  함수를 통해 이전에 중지했던 작업들을 복구할 수 있으며, 

 `applicationDidBecomeActive:` 함수를 이용하여 이전에 실행 환경에서의 작업들을 진행한다.



### Be Prepared to Process Queued Notifications

정지된 App은 다시 foreground나 background로 진입 시 그 동안 queue에 저장된 알람들을 처리할 준비가 되어있어야 한다. 정지된 App은 어떠한 코드로 실행할 수 없으므로, 다양한 변화들을 반영할 수 없다. 이러한 변화들이 손실되지 않도록 system은 모두 저장해둔 뒤, App이 복귀되는 시점에 불러온다. 

App이 전달하는 알람에는 아래 항목이 있다.

- 외부기기 등록/해제
- 기기 방향 전환
- 장기간의 시간 변화
- 배터리 상태 변화
- 근접 상태 변화
- 보호된 파일의 상태 변화
- 화면 출력 변화
- 설정App에서의 설정값 변화
- 언어, 지역 변경
- iCloud 계정 변경

이러한 형태의 저장된 알람들은 touch event나 다른 user input 전에 App의 main loop로 들어온다. 저장된 알람은 인지 가능한 정도의 지연 없이 처리되어야 한다. background 상태로 복귀 시 App이 느려짐을 확인한다면, `Instruments` 를 통하여 지연을 발생시키는 원인을 찾을 수 있다.



### Handle iCloud Changes

iCloud와 관련된 변화가 발생하여 알람이 수신된다면, system은 `NSUbiquityIdentityDidChangeNotification` 알람을 전송한다. iCloud에 로그인/아웃을 하거나,  동기화 설정을 수정한다면 알람이 발생한다. 이러한 알람을 통하여 관련된 UI나 cache를 업데이트 하여야 한다. iCloud사용 여부를 사전에 확인하였다면, iCloud 상태가 변경되어도 반복되는 확인 요청을 하지 않도록 한다. 관련 설정은 설정App을 통하여 할 수 있도록 한다.



### Handle Locale Changes

App이 종료된 동안 지역이 변경된다면, `NSCurrentLocaleDidChangeNotification` 알람을 받을 수 있다. 이 알람을 활용하여 지역과 관련된 업데이트를 진행하여야 한다.



### Handle Changes to Your App’s Settings

설정 App에서 변경 된 값이 존재한다면,  `NSUserDefaultsDidChangeNotification`  알람을 수신할 수 있다. 이 알람을 활용하여 설정 값과 관련된 업데이트를 진행하여야 한다. 이 알람을 활용함으로써  보안과 관련된 문제를 예방할 수 있다.



## What to Do When Your App Enters the Background

App이 background 상태로 진입 시,  `applicationDidEnterBackground:` 함수를 활용하여 다음의 작업을 수행할 수 있다. 

- App의 스크린 샷 준비
- 상태 정보 저장
- Free memory 

App delegate의 `applicationDidEnterBackground:`함수는 작업 종료까지 약 5초의 시간을 제공한다. 제한 시간이 지난 후에도 함수가 return 되지 않는다면, App은 종료되며 memory는 해제된다. 추가적인 시간이 필요하다면, `beginBackgroundTaskWithExpirationHandler` 함수를 통하여 긴 작업을 secondary thread에서 진행할 수 있다. 



system은 `UIApplicationDidEnterBackgroundNotification`알람을 전송하며 `applicationDidEnterBackground:`  함수를 호출한다. 위 알람을 활용하여 여러 작업 정리를 실행할 수 있다.



### The Background Transition Cycle

사용자가 홈버튼, 잠금버튼을 누르거나 다른 App이 실행된다면, 기존의 App은  Inactive상태를 거쳐 background 상태로 진입한다. 이 과정에선 `applicationWillResignActive:` 와 `applicationDidEnterBackground:`  두 함수가 호출된다. `applicationDidEnterBackground:`함수 호출 후, 보통 App들은 잠시나마 정지 상태가 된다. background작업을 실행하거나, 다른 실행 시간을 필요로 하는 App의 경우, 더 오랜시간 유지된다.



### Prepare for the App Snapshot

`applicationDidEnterBackground:`함수가 반환된 후 system은 App의 스크린샷을 저장한다. App이 background 작업을 위하여 활성화 된다면, system 은 스크린샷을 업데이트 한다. 

background 진입 시 UI변경을 적용하고 싶다면,  `snapshotViewAfterScreenUpdates:` 함수를 호출하여 적용한 변화의 rendering을 강제할 수 있다. view에서 `setNeedsDisplay` 함수를 호출하는 것은 다음 drawing주기 전에 스크린샷을 저장하여 변화가 적용되지 않으므로 비효율적이다. 

### Reduce Your Memory Footprint

모든 App은 background진입 시 memory들을 해제하여야 한다. background상에서 많은 memory를 사용하는 App은 memory확보를 위한 system의 App종료 대상의 1순위이다.

App 내에서의 강한 참조들은 더이상 필요하지 않을 시 해제되어야 한다. 이는 memory가 효율적으로 사용되는 것에 도움을 준다. 만약 성능의 향상을 위해 cache를 원한다면, 참조를 제거하기 전에 App이 background로 전환 될 때까지 기다릴 수 있다. 

다음의 항목에선 강한 참조를 제거해야 한다.

- 이미지 객체
- 큰 미디어, 데이터 파일
- 재생성이 쉬운 파일

App은 메모리 확보를 위해 자체적으로 자원을 수집한다.

- CA Layer 들을 최적화 한다. layer들을 memory에서 제거하지는 않는다. screen에 반영되는걸 막으며 이는 background에선 진행되어서는 안 되는 작업들이다.
- cache된 이미지에 대한 system참조를 제거한다.
- System-managed data cache들의 강한 참조를 제거한다.