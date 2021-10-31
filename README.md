# 프로젝트 관리 앱
#### 프로젝트 기간 21.10.25 ~ 21.11.19

## 앱 소개

### 적용 기술
#### UI - SwiftUI
- 이번 프로젝트는 오로지 SwiftUI로만 UI를 구성한다.

#### 아키텍쳐 - MVVM 디자인 패턴
- MVC 패턴을 사용하며 뷰컨트롤러가 거대해지는 단점을 경험한 적이 있어 MVVM을 선택했다.
- SwiftUI 사용시 MVVM의 구조적인 이점을 취할 수 있고 둘의 조합이 좋아 선택했다.

#### 데이터 저장소 - 로컬은 SQLite, 리모트는 Firebase의 Cloud FireStore
- SQLite
  - Firebase의 오프라인 기능을 사용할지 고민이 되었으나 현재 프로젝트는 오프라인 데이터베이스가 더 중심이 되는 데이터베이스라 판단했다.
    - Firebase는 인터넷 연결이 끊겼을 때 로컬의 캐시를 사용하고, 다시 연결이 되면 이를 서버의 데이터베이스와 연동한다. 하지만 데이터가 많아지거나 수정사항이 많을 때에도 캐시가 데이터베이스를 충분히 대체할 수 있는지 의문이 들어 별도의 로컬 데이터베이스를 두기로 선택했다.
  - 현재 프로젝트는 데이터 구조가 복잡하지 않고 규모도 크지 않아 SQLite를 적용하기 적절하다고 판단했다.
    - SQLite는 널리 사용되는 데이터베이스고 SQL 쿼리문을 사용하기에 익혀두면 이후 프로젝트나 협업에 도움이 될 것이라 판단해 선택했다.- 
  - SQLite는 편리성을 위한 [SQLite.swift](https://github.com/stephencelis/SQLite.swift)보다는 Swift 내부 SQLite3 모듈을 import해서 사용했다.
  - 예전부터 많이 사용되어왔던 로컬 데이터베이스니 안정적이고 지속가능하다고 판단했다. 그리고 Firt-Party인 Core Data가 SQLite 저장 타입을 지원하니 저 지속가능성이 있을 것이라 추측했다.
- Firebase
  - Firebase는 리모트 저장소로 많이 쓰이는 기능이며 소셜 로그인 인증 기능이 필요해 선택했다.
  - Firebase의 데이터베이스 중 Cloud Firestore를 선택한 이유는 정렬과 필터링 등의 한번에 하는 복잡한 쿼리가 가능하다는 점이 크게 작용했다. 
    - 부수적으로 Realtime Database보다 더 구조적이라는 것과 더 최신 기술이라는 이유 등이 있다.
  - 의존성 도구는 CocoaPods을 사용했다. 
    - First-Party인 Swift Package Manager를 써보고 싶었으나 이에 필요한 Xcode 최소 버전인 12.5를 충족하지 못했다. Swift Package Manager를 사용하려면 Xcode를 업그레이드하거나 Firebase를 7.11로 버전을 낮추어 사용해야해 CocoaPods을 선택했다. 
    - Firebase 문서에서 [CocoaPods](https://firebase.google.com/docs/database/ios/start#:~:text=we%20recommend%20using%20cocoapods%20to%20install%20the%20firebase%20libraries)을 권장한다고 되어 있었다.
  - 많이 사용되는 데이터베이스며 구글에서 지원하니 안정적이고 지속가능성이 있다고 판단했다. 다만 Freemium 서비스라 유료화가 될 수 있는 리스크가 있다.

## 앱 설계/구조

### Database 동기화
<img src="https://user-images.githubusercontent.com/52592748/139563269-8716e483-04b8-49d8-b602-874ceec79b48.png" width="600"/>

#### 개인 프로젝트 관리 기능
- 현재 앱에서 지원하는 것은 개인 프로젝트만 관리하는 기능들이다.
- 따라서 수정사항을 로컬 데이터베이스에 먼저 반영하고 인터넷에 연결되어있다면 리모트 데이터베이스에도 반영한다.
- 만약 인터넷에 연결되어있지 않다면 로컬데이터베이스만 반영한다. 그리고 인터넷에 연결되는 순간 로컬의 내용을 모두 리모트에 업데이트해준다.

#### 공유 프로젝트 관리 기능
- 다른 사용자들과 공유하는 프로젝트도 기능은 아직 지원하지 않는다.
- 지원하는 경우 공유 프로젝트는 인터넷이 연결되어있을 때에만 CRUD가 가능하도록 제한해 DB간 충동을 방지한다.
- 인터넷에 연결되어 있다면 수정사항을 리모트 데이터베이스에 업데이트하고 그것을 바로 로컬에 반영한다.
- 인터넷에 연결되어 있지 않다면 수정이 불가하다고 사용자에게 안내한다.

