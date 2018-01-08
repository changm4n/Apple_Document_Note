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

