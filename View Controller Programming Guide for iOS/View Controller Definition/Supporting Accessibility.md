# Supporting Accessibility

iOS App은 장애나 신체적 결함이 있는 사용자를 포함하여 모두가 이용할 수 있어야 한다. 접근성을 향상시키기 위하여, VoiceOver와 같은 기능을 활용할 수 있다. 추가적으로 접근성을 확보하기 위하여, 다음 항목을 준수하자.

- 컨트롤, 레이블을 포함하여 모든 UI는 접근성을 보장하여야 한다.
- 접근성을 가진 요소들이 명확한 정보를 제공하여야 한다.

프로그래밍 방식으로 VoiceOver 포커스 링의 위치를 설정하거나, 특별한 VoiceOver 동작에 응답하거나, 접근성 알림을 관찰하여 App에서 VoiceOver 사용자의 경험을 향상시킬 수 있다. 



### Moving the VoiceOver Cursor to a Specific Element

새로운 View를 표현할 때, VoiceOver의 커서 위치를 고려하여야 한다. 레이아웃이 바뀌면, 좌상단에 배치된 요소를 우선으로 재배치된다. 적절한 위치로 설정시켜 효율을 개선할 수 있다.

VoiceOver의 커서 위치를 변경하기 위해서, `UIAccessibilityScreenChangedNotification` 알람을 `UIAccessibilityPostNotification` 메소드를 활용하여 등록한다. 알람을 등록할때, 커서의 위치를 지정한다.



### Responding to Special VoiceOver Gestures

VoiceOver는 특정 액션을 정의할 수 있다. 이는 Responder chain을 적용하여 전달된다. 이는 별도의 설정이 가능하지만, 직관적이고 기존의 가이드라인을 준수하여야 한다.

- **Escape.**  - Modal dialog나 뒤로 이동을 지시하는  two-finger Z-shaped gesture
- **Magic Tap.** - two-finger double-tap 주요 액션을 실행한다.
- **Three-Finger Scroll.** - A three-finger swipe 수평, 수직 방향의 스크롤 이동
- **Increment.** -  A one-finger swipe up 값의 증가
- **Decrement.** - A one-finger swipe down 값의 감소



### Escape 

`accessibilityPerformEscape` 함수로 대응할 수 있다. View위에 표시되는 내용들을 사라지게 한다. 컴퓨터의 Esc키와 대응된다. UINavigationController의 Back 이동 기능 또한 적용되어야 한다.



### Magic Tap

`accessibilityPerformMagicTap`함수로 대응할 수 있다. 주요 기능을 실행하는 의미를 전달한다. 예로, 전화 앱에선 전화를 받거나 끊는 용도로, 시계 앱에선 스탑워치의 시작, 종료로 활용할 수 있다. 



### Three-Finger Scroll

 `accessibilityScroll:` 함수로 대응할 수 있다.  페이지 관련 컨텐츠의 페이지를 이동하거나 스크롤 제스쳐를 담당한다.



### Increment and Decrement

 `accessibilityIncrement` 또는  `accessibilityDecrement` 로 대응한다. 값의 증가, 또는 감소를 담당한다.



### Observing Accessibility Notifications

예를들어, Books App의 경우, VoiceOver를 활용하여 페이지 내 모든 내용을 음성으로 출력하였을 때, 페이지를 전환할 것에 대한 `UIAccessibilityAnnouncementDidFinishNotification`알람을 전송한다. 이러한 알람은 사용자에게 끊임없는 감상의 경험을 제공하기 위하여 구현된다.

