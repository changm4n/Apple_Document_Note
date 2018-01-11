# Performance Tips

App을 설계할 때, 전력 소비와 메모리 사용은 중요한 요소이다.  



## Reduce Your App’s Power Consumption

하드웨어 기능을 종료함으로서,  전력을 관리할 수 있다. 다음의 항목을 최적화 하여 배터리 수명을 연장할 수 있다.

- CPU
- Wi-Fi, Bluetooth, baseband radios
- Core Location framework
- Accelerometers
- Disk

다양한 작업을 효율적으로 수행하여 최적화를 구현한다. 이엔 Instrument를 활용할 수 있지만, 여전히 전력 소모엔 부정적 영향을 주는 요소가 있을 수 있다. 다음 항목을 고려해보자.

- CPU가 sleep  상태로 전환되는걸 막는 Polling 작업을 지양하라. `NSRunLoop`, `NSTimer`를 활용하라.
- `UIApplication`객체의 `idleTimerDisabled`값을 NO로 유지하라. 이는 자동으로 스크린이 꺼지는 작업을 수행한다. 
- 작업들을 합쳐 작업 수행에 소모되는 전체 시간을 줄여야 한다. 
- 잦은 디스크 접근을 지양하라. 
- 필요 이상의 screen 업데이트는 전력 낭비의 원인이 된다. 
- `UIAccelerometer`를 사용한다면, 필요하지 않은 순간엔 disable하라. 
- 필요한 순간, 외부 네트워크에 접속하며 외부 서버를 poll하지 않는다.
- compact data format을 사용하여, 네트워크 통신에 사용되는 데이터를 최소화 하라. 
- 데이터는 한꺼번에 전송하라. system은 App이 활동이 줄어들 시 Wi-Fi나 radio를 중지한다. `NSURLSession`을 활용하여 일련의 작업들을 queueing하여 작업하라.
- cellular 데이터 보단  Wi-Fi를 사용하라.
- Core Location을 사용 중 이라면, 위치 업데이트를 최소화 하며, 거리 filter와 정확도를 적절히 설정하라. 



## Use Memory Efficiently

System이 가용할 수 있는 메모리를 최대화 하기 위하여, App이 사용하는 메모리를 최소화 하여야 한다.



### Observe Low-Memory Warnings

system으로 부터 low-memory 경고를 받는다면, 즉시 대응하여야 한다. 이는 원치 않은 reference 해제를 야기할 수 있다. low-memory 경고는 `AppDelegate`, `UIViewcontroller` 등을 통하여 감지할 수 있다. 

이러한 경고를 수신할 시, 불필요한 메모리를 해제하고 존재하는 cache, images 들을 제거하여야 한다. 큰 규모의 Data Structure를 할당 중 이라면, release해주어야 한다.



### Reduce Your App’s Memory Footprint

| Tip                                      | Actions                                  |
| ---------------------------------------- | ---------------------------------------- |
| Eliminate memory leaks                   | 메모리 누수를 방지하여야 한다. Instruments를 활용하여 누수를 탐지할 수 있다. |
| Make resource files as small as possible | 사용되는 image파일들을 압축하여야 한다. plist 파일을 binary format으로 작성하여 그 크기를 줄일 수 있다. |
| Use Core Data or SQLite for large data sets | 큰 규모의 data를 저장 시, Core Data나 SQLite를 사용하라. |
| Load resources lazily                    | 리소스가 꼭 필요한 시점 이전에 로드해선 안 된다. 미리 로드하는 작업은 시간을 절약할 수 있지만, 전체적인 성능을 감소시킨다. |



### Allocate Memory Wisely

큰 해상도의 이미지 대신, iOS에 최적화된 크기의 이미지를 사용하는게 좋다. 큰 용량의 리소스를 사용한다면, 그 파일을 부분적으로 로드하여 사용하는 방법을 찾아야한다. 

작업에 요구되는 자원을 한정하라. 그 작업이 가용할 수 있는 메모리보다 더 큰 메모리를 요구한다면, 그 작업을 처리될 수 없다.



## Tune Your Networking Code

iOS에선, 네트워크 통신을 위한 여러 인터페이스를 제공한다. CFNetwork와 NSStream을 이용할 수 있다.



### Tips for Efficient Networking

네트워크를 통하여 데이터를 송,수신하는 작업은 많은 자원을 소모하는 작업 중 하나이다. 이러한 작업의 시간을 줄이므로서 배터리 소모를 최적화 할 수 있다. 

- compact한 data format을 정의하여 사용하라.
- 불필요한 형태의 프로토콜은 지양하라.
- 데이터 전송을 짧은 시간, 최대한 많이 이루어져야 한다.

Wi-Fi와 cellular는 미사용시 중지되므로, 데이터 교환은 최대한 짧은 시간 이루어져야 한다.  

네트워크 통신에서, 교환되는 패킷은 손실될 수 있으므로, 최대한 견고한 처리를 구현하여야 하며 네트워크 상태 변화도 감지하여 대응하는게 좋다.



### Using Wi-Fi

 `Info.plist`에 `UIRequiresPersistentWiFi`를 추가하여 Wi-Fi 를 활용하여 통신함을 알려야 한다. 이를 설정하면, system은 주변 Wi-Fi나 Hotspot을 탐지 시 dialog를 표시한다. 또한 App을 사용 중, Wi-Fi 가 자동으로 정지되지 않도록 한다.

30분 이상 Wi-Fi를 사용하지 않으면, System이 Wi-Fi기능을 종료하지만, `UIRequiresPersistentWiFi`키가 추가된 App은,  자동으로 종료시키지 않는다. 

**NOTE: `UIRequiresPersistentWiFi`키가 YES로 설정되어도, 기기가 잠금 상태일 땐, App이 Inactive 상태이므로 적용되지 않는다.****



### The Airplane Mode Alert

다음 조건을 만족하면,  비행기모드에서 App이 실행될 시, 사용자에게 알려준다.

- `Info.plist`의 `UIRequiresPersistentWiFi`키의 값이  YES다.
- 비행기모드에서 App이 실행된다.
- 비행기모드 설정 후, Wi-Fi 기능이 활성화되지 않았다.



## Improve Your File Management

디스크에 쓰는 작업은 최소화 하여야 한다. 다음 항목을 살펴보자.

- 수정 작업은 병합하여, 작업된 부분만 Write한다. 
- 저장된 데이터가 무작위로 접근된다면, Core Data나 SQLite를 이용하라.



## Make App Backups More Efficient

무선 통신으로 데이터가 iCloud나 iTunes로 백업된다. app sandbox에 저장된 데이터는 백업여부가 결정된다. 큰 규모의 데이터를 자주 저장한다면, 전체적인 백업 과정을 느리게 만들 것 이다.



