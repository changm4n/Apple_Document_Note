# Background Execution

보통 iOS는 사용중이지 않은 앱을 suspended 하기 전, background 상태로 유지한다. background 상태에선 여러 작업들을 수행할 수 있다. 불필요한 목적의 background 처리는 배터리 소모, 메모리 낭비를 야기하므로 꼭 필요한 목적으로만 background execution을 활용하도록 한다.



## Executing Finite-Length Tasks

App이 background 실행이 필요하다면, `beginBackgroundTaskWithName:expirationHandler:`  또는 `beginBackgroundTaskWithExpirationHandler:`  함수를 활용하여 App이 suspend되지 않도록 방지하며, 작업을 수행할 수 있다. 작업이 종료되었다면, `endBackgroundTask:` 를 호출하여 suspend가능한 상태로 전환하도록 한다. 

`beginBackgroundTaskWithName:expirationHandler:` `beginBackgroundTaskWithExpirationHandler:`   두 함수는  Task를 식별하는 토큰을 발행하며, 

`endBackgroundTask:` 메소드에 토큰을 넘겨줌으로서 작업 종료를 알릴 수 있다.  `endBackgroundTask:` 함수 호출에 실패한다면, App은 종료될 수 있으므로, 작업 Expiration Handler를 설정하여, 정상적으로 작업을 중단, App이 종료될 수 있도록 설정할 수 있다.



위 두 함수는 App이 Foreground 상태일때 또한 호출이 가능하다. 이는 더욱 편리한 구조를 제시한다. 

다음 예제 코드로 background 작업의 실행, 종료를 살펴보자.

```swift
func applicationDidEnterBackground(_ application: UIApplication) {
        var task = application.beginBackgroundTask(withName: "taskName") 		 {
            // Clean up task unfinished
            // Stop task
            application.endBackgroundTask("taskName")
            task = UIBackgroundTaskInvalid

        }
        // Start Task
        DispatchQueue.global().async {
            //Do task
            task1()
            task2()
   
            application.endBackgroundTask("taskName")
            task = UIBackgroundTaskInvalid
        }
    }
```



Expiration Handler는 항상 제공하여야 한다. `backgroundTimeRemaining`  property를 활용하여 App의 남은 실행시간을 확인할 수 있다.  Expiration Handler이 호출된 시점은, 이미 App이 종료되는 시간에 가깝단 의미이므로, 긴 시간이 소요되는 작업은 포함하지 않도록 한다.





## Downloading Content in the Background

`NSURLSesstion` 을 활용하여 background에서의 Downloading을 구현할 수 있다. 그 방법으로는, `NSURLSessionConfiguration` object를 생성하여 별도의 설정 후, `NSURLSesstion`의 생성자에 전달한다. 

background에서의 download가 완료된다면, 종료된 App은 재실행되어, `application:handleEventsForBackgroundURLSession:completionHandler:`를 호출한다. 



## Implementing Long-Running Tasks

장시간 작업이 필요한 작업에 대해서는 특정 권한 요청이 필요하다. 이 설정을 사용할 수 있는 App의 종류로는 오디오, 녹음, 위치, Voice over, download, 외부 장치로부터의 응답을 구현한 App이 있다.

위 기능을 다루는 App은 그 기능을 구현한 Framework와 서비스들을 `Capabilities` 또는 `info.plist`에 명시하여야 한다. 



### Tracking the User’s Location

높은 정확도의 위치정보가 필요한 앱이 아니라면,  사용자의 위치 변화를 탐지하는 수준의 Location service가 권장된다.  위치정보가 update되는 시점에 App이 종료된 상태라면, System은 App을 재실행하여 background 상태에서 update된 정보를 다루게 한다. 

Foreground에서만 위치정보를 다루도록 구현한다면, App이 background 상태에서의 특별한 구현이 없을 땐 위치정보를 update하지 않는다. 

`Capabilities` 에서 background 상태에서의 위치 정보 활성화를 설정할 수 있다. 이 설정이 활성화 되어 있다면, App이 suspend되더라도 위치정보가 update될 시, App은 다시 실행되어 작업을 실행한다. 

위치정보를 상시로 다루는 작업은 많은 전력을 소비하므로, 이 기능은 최소화하여 구현하는게좋다.



### Playing and Recording Background Audio

`Capabilities` 의 설정으로 background 상태에서의 오디오, 녹음 기능을 활성화 할 수 있다. background 상태에선 음악, 녹음, AirPlay를 활용한 playback, VoIP apps와 같은 기능을 활용할 수 있다.  음악 재생, 녹음과 같은 상황에선 App이 종료되지 않지만, 재생이 종료되거나, 녹음이 종료된다면 App은 종료될 수 있다. 



background에서 지정된 작업은 실행되며 callback 함수 또한 정상적으로 호출되므로, content를 제공하기 위한 작업들이 이루어져야 한다. 예) streaming을 위한 download

다양한 App이 background 상태에서 실행되며 audio를 활용 할 수 있다. Foreground 상태의 App이 우선순위를 가지며, background 상태의 App 또한 Audio를 재생할 수 있다. 이러한 결정은 App의 Audio session object를 통하여 설정할 수 있다. 



### Fetching Small Amounts of Content Opportunistically

정기적인 작업을 위한 App실행은 `Capabilities`의 설정으로 App을 재호출하여 구현이 가능하다. 이 설정을 활성화 하더라도, System 자체적인 조율을 통하여 App을 실행하기 때문에, 그 과정을 항상 보장할 수 없다.  기회가 되어,  App이 호출된다면, `application:performFetchWithCompletionHandler:` 함수가 호출된다. 이 함수에서 작업의 진행에 따라 App의 상태를 설정할 수 있다. 



### Using Push Notifications to Initiate a Download

'새로운 content가 유효할 때, 사용자에게 알리며 즉시 download를 시작하는 기능' 의 목적은 사용자가 알람을 확인하고, 새로운 content에 접근하는 시점의 간격을 좁히는데 사용된다. 이러한 기능은 `Capabilities`에서 `Remote notifications` 옵션을 활성화 하여 구현이 가능하다. `content-available` key의 값을 1로 설정한 원격 알람을 통하여 download를 시작할 수 있다. 

`application:didReceiveRemoteNotification:fetchCompletionHandler:`가 호출되며 App이 활성화된다. 



### Communicating with an External Accessory

외부 기기의 호출을 통하여 App을 활성화시킬 수 있다. `Capabilities` 의 설정으로 이 기능을 사용할 수 있다.  

이 기능은 다음의 사항을 준수하여야 한다.

- content 전송의 시작, 종료를 사용자가 설정할 수 있어야 한다.
- 활성화된 App은 10초 정도의 시간을 작업에 소요할 수 있다. `beginBackgroundTaskWithExpirationHandler:` 함수를 활용하여 추가적인 시간을 요구할 순 있지만 권장되진 않는다.



### Communicating with a Bluetooth Accessory

BLE기기의 호출을 통하여 App을 활성화시킬 수 있다. `Capabilities`에서 설정이 가능하다. 이 기능이 활성화되어 있다면, `Core Bluetooth` framework는 peripheral과의 session을 유지한다. 기기의 data update, connection 시작, 종료를 신호로 App은 호출될 수 있다.  

이 기능은 다음의 사항을 준수하여야 한다.

- content 전송의 시작, 종료를 사용자가 설정할 수 있어야 한다.
- 활성화된 App은 10초 정도의 시간을 작업에 소요할 수 있다. `beginBackgroundTaskWithExpirationHandler:` 함수를 활용하여 추가적인 시간을 요구할 순 있지만 권장되진 않는다.



## Getting the User’s Attention While in the Background

Notification은 App이 실행 중이지 않거나, background 상태이거나, suspended 상태일 때 사용자의 주의를 끌 수 있는 방법이다. 

`UILocalNotification` class를 이용하여 Local Notification을 구현할 수 있다. Local Notification은 그 형태(소리, 경고, 배지)와 시각에 대한 정보를 담고있다.

Note : `UILocalNotification`은 iOS10 에서 deprecated되었다. UNNotificationRequest 를 사용한다.

## Understanding When Your App Gets Launched into the Background

App이 종료되었더라도, 다음과 같은 환경에서 System은 App을 다시 실행시킨다.

- 위치 정보를 사용 시, Location 정보가 update 되었다.
- 기기가 지정된 지역에 도달하였다.
- 음악 서비스 제공 시, 새로운 Data를 호출하여야 한다.
- BLE기기를 사용 시, 기기가 central 역할을 수행 중 일때, 연결된 peripheral로 부터 data update를 수신하였다.
- 기기가 peripheral 역할을 수행 중 일때, central로 부터 commands를 수신하였다.
- Background download를 진행 중일 때, `content-available : 1 ` 값을 가지는 알람을 수신하였다.
- 새로운 content를 download하기 위하여, System이 비정기적으로 App을 실행시킨다.
- Newsstand app 종료로 인해 download가 시작되었다.

만약 사용자가 강제로 App을 종료시켰다면, 위치 정보를 사용하는 App은 예외로 System 은 App을 재실행하지 않는다. 기기가 잠금상태일때 또한, System은 App을 재실행하지 않는다.



## Being a Responsible Background App

App이 background상태라면, 다음과 같은 guideline을 준수하여야 한다.

- OpenGL ES를 호출해선 안 된다.
- App이 종료 전 모든 **Bonjour-related services** 를 종료하여야 한다.
- Network작업의 오류처리를 구현하여야 한다.
- background 진입 전, App의 state를 저장하여야 한다.
  - 메모리 관리에 의해 불시에 해제될 수 있다.
- background 진입 전, 모든 강한 참조를 제거하여야 한다.
  - 특별히, image관련 참조를 해제하여야 한다.
- suspended 되기 전, 공유 자원 사용은 중단하여야 한다.
- window,view update 를 지양하여야 한다.
- 외부 기기로 부터의 연결,해제 handler를 구현하여야 한다.
- background 진입 전, 활성화된 alert 자원을 해제하여야 한다.
  - 활성화된 `UIActionsheet`나 `UIAlertView` 는 자동으로 해제되지 않는다.
- 민감한 정보들을 표현하는 views들을 제거하여야 한다.
  - Background진입 시, System은 현제 App의 Snapshot을 노출시킨다. `applicationDidEnterBackground:` 함수에서 처리해주어야 한다.
- 최소한의 작업만을 수행하여야 한다.





## Opting Out of Background Execution

만약 App의 Background 상태에서의 실행을 원하지 않는다면, `info.plist`의 `UIApplicationExitsOnSuspend`값을 설정함으로써 방지할 수 있다.

만약 이 설정이 활성화되어 있다면, App은 `not-running`,`inactive`,`active` 상태만을 갖는다. 

사용자가 홈버튼을 눌러 App을 종료하려 한다면,  `applicationWillTerminate:` 함수가 호출되어 약 5초 후 `not-running`상태로 진입한다.



이 설정은 권장되지 않는다. App을 단순화 하거나, 큰 규모의 작업을 다룬다면, 이 설정은 해답이 될 수 있다. 