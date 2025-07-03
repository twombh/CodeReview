# Code Review
## al.base
### 역할
앱의 공통 동작을 캡슐화하고 일관된 구조룰 제공하기 위한 기반 클래스
### Code
#### base.ALBaseActivity
- 안드로이드 앱에서 사용하는 추상 클래스
-  앱의 주요 Activity들이 상속해서 공통 로직을 사용할 수 있게 만든 기반(Base) 액티비티
- 공통 UI 처리, 데이터 바인딩, ViewModel 상태 관찰, 화면 전환 로직을 일괄적으로 관리하는 앱의 기반 액티비티 클래스

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|binding|protected|액티비티와 XML 바인딩|lateinit|
|progress|private|진행상태 다이얼로그를 WeakReference로 메모리 누수 관리|-|

|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|onCreate|Bundle|override|데이터 바인딩 설정 및 뷰 애니메이션 시작|코루틴으로 루트 뷰에 자동 애니메이션 적용|
|startMain|-|protected|데이터를 초기화하고 MainActivity로 이동|-|
|finishReservationStack|-|protected|예약 내역 액티비티로 이동하면서 스택 정리|-|
|finishReservationStackAndGoCancelInfo|journeySequenceNo, journeySequenceCode|protected|예약 취소 정보 전달하며 예약 상세 액티비티 이동|-|
|ViewModel|-|abstract|하위 클래스에서 viewModel 제공|ALBaseViewModel형|
##### 역할
- 데이터 바인딩과 UI 초기화를 간소화
- ViewModel의 UI 상태를 옵저빙하여 다이얼로그, 토스트 등 공통 UI 처리
- 메모리 누수 방지 (WeakReference 사용)
- 재시작 시 초기화 처리
- 액티비티 전환 로직의 공통화


#### base.ALBaseBottomSheet
- 커스텀 하단 시트(BottomSheet) 기반 클래스
- 안드로이드의 BottomSheetDialogFragment를 확장하며 하위 BottomSheet 클래스들이 상속해서 사용할 수 있도록 공통 동작을 정의
- ALBaseBottomSheet는 중복 표시 방지, 테마 지정, 기기별 화면 높이 계산 기능을 공통 제공하는 커스텀 BottomSheet 기반 클래스
##### 변수 및 함수
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|onCreate|Bundle|override|바텀시트에 사용할 커스텀 테마 지정|-|
|show|FragmentManager|default|FragmentManager에 현재 바텀시트와 같은 타입의 Fragment가 이미 보이는 중이면 재표시하지 않고 아니면 super.show()를 통해 표시|this@...로 인스턴스 직접 명시|
|screenHeight|-|Protected|상태바와 네비바를 제외한 실제 컨텐츠 높이를 반환하여 특정 디바이스에서 상태바 높이가 잘못 포함되는 버그 우회|-|
##### 역할
- 공통 바텀시트의 기반 역할 제공
- 테마 지정, 중복 방지 표시, 기기 호환 화면 높이 계산
- 	다양한 디바이스에서 안정적인 바텀시트 동작 보장
#### base.ALBaseDialog
- 안드로이드의 DialogFragment를 상속한 클래스
- 앱에서 공통적으로 사용하는 다이얼로그(Dialog) UI의 기반 클래스
- Fragment already added 예외 방지, 투명 배경 설정, 중복 표시 방지 로직을 담당
##### 변수 및 함수
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|onViewCreated|View, Bundle|override|다이얼로그 뒷배경을 투명하게 설정|-|
|show|FragmentManager|default|중복으로 다이얼로그가 표시되지 않도록 확인|-|
|show|FragmentActivity|default|FragmentActivity만 받아서 중복으로 다이얼로그가 표시되지 않도록 확인|-|
|dismiss|-|override|다이얼로그가 닫히면 플래그를 false로 초기화하여 다음에 다시 표시 가능하도록 함|-|
##### 역할
- dialog 창을 커스텀 UI로 쓸 수 있게 함
- isShowLoading, isAdded, isShowing 조건으로 show 중복 방지
- Fragment already added 예외를 수동 플래그로 해결
- FragmentManager 또는 FragmentActivity에서 쉽게 호출 가능
- 닫을 때 플래그 초기화로 다음 표시 가능하게 함
#### base.ALBaseRepository
- 안드로이드 앱에서 레포지토리(Repository) 패턴의 기반이 되는 클래스
- RxJava(Single) + Retrofit2 + Gson을 사용한 네트워크 응답 처리 공통 로직 담당
- 서버 응답을 공통 포맷(ALBaseResponse)으로 받아 모델(ALBaseModel)로 매핑하는 역할
- ALBaseRepository는 서버 응답을 공통 모델로 변환하면서 실패 시에도 기본 에러 정보를 담아내는 공용 매핑(mapper) 함수를 제공하는 클래스
##### 변수 및 함수
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|mapper|ALBaseResponse<T>.() -> R?|protected|retrofit으로 받은 Single<Response<ALBaseResponse<T>>>를 ALBaseModel로 변환|응답이 성공이면 body()를 꺼내서 block() 함수로 변환 수행, 실패한 경우에도 모델 객체(R)를 생성한 후 errorBody()를 파싱해서 errorCode, errorMessage를 설정하고 406, 412이면 중복 로그인으로 간주|

##### 역할
- 서버 응답(ALBaseResponse)을 앱 모델(ALBaseModel)로 매핑
- 성공/실패 모두 일관된 방식으로 처리
- Gson으로 errorBody 파싱하여 모델에 에러코드, 메시지 주입

#### base.ALBaseViewModel
- ALBaseModel
    - ViewModel의 공통 동작을 추상화
    - API 호출과 UI 상태 이벤트를 통합 관리
- ALUiState
    - UI에 전달되는 일회성 상태 메시지들을 정의
    - 로딩, 에러, 알림 등 UI에 보여줄 상태를 표현하는 구조
##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|compositeDisposable|protected|ViewModel이 사라질 때 메모리 누수 방지|lazy|
|_uiState, uiState|private|ViewModel에서 UI로 단방향 상태 전달|Event래퍼로 LiveData가 UI에 한 번만 이벤트 전달할 수 있도록 함|

|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|callToRemote|다수|protected|서버 API 호출을 실행하고, 결과에 따라 onSuccess, onError, onFailed, ALUiState 등으로 UI에 상태를 전달하는 메서드|single<T>로 서버 api 호출|
|callToLocal|다수|protected|값이 없는 비동기 작업 성공만 콜백으로 전달|-|
|onCleared|-|override|등록된 모든 Rx 구독 해제하여 메모리 누수 방지|-|
|ALUiState|-|sealed|UI에 전달할 수 있는 다양한 상태들을 타입 안전하게 표현|sealed class를 써서 패턴 매칭에 유리|
|ALUiState.Alert|message, isFinish|-|문자열 메시지로 알림 다이얼로그 표시|data class|
|ALUiState.AlertWithResource|@StringRes message|-|문자열 리소스를 기반으로 알림|data class|
|ALUiState.Toast|@StringRes message|-|리소스 기반 토스트 메시지|data class|
|ALUiState.Loading|isShow|-|로딩 시작/종료|data class|
|ALUiState.DuplicateLogin|-|-|중복 로그인 발생 시|data class|

##### 역할
- 모든 ViewModel에서 서버 API 호출을 쉽게 하도록 도와주는 공통 비동기 처리 도우미 클래스
- UI에 로딩/에러/토스트 같은 상태 이벤트를 안전하게 전달하는 중간 관리자


## al.data

### 역할
각각 서버와 직접 통신하는 역할을 하는 클래스

### Code
#### data.datasource.ALBusInfoDataSource
##### 역할
- 기초 코드 정보 조회
- lstDate를 기준으로 서버에서 공통 코드 데이터를 가져옴

#### data.datasource.ALBookmarkDataSource
##### 역할
- 즐겨찾는 노선 조회, 저장, 삭제 요청을 각각 담당
- ViewModel/Repo는 요청 객체만 넘기면 됨

#### data.datasource.AlCommonDataSource
##### 역할
- 프로모션 배너, 팝업, 결제 수단, 약관, 카드 BIN 정보 등
- 앱 내에서 공통으로 재사용되는 정보들을 조회

#### data.datasource.ALReservationDataSource
##### 역할
- 예약 생성, 결제 요청, 환불 요청, 사유 기반 환불 등
- 티켓/여정 예약 전후 단계의 API

#### data.datasource.ALTicketDataSource
##### 역할
- 발권/예약 내역 조회
- 예약 리스트, 상세 정보, 동기화 정보, 사용 이력 등을 서버로부터 가져옴

#### data.datasource.ALTravelDataSource
##### 역할
- 여행 상품 관련 정보 조회
- 여행 리스트, 요금 규칙 (출발 전/후/할인) 서버에서 조회

#### data.entities.request.ALBaseCodeInfoRequest
##### 역할
- 기초 코드 정보 조회 API 호출 시 요청 파라미터 전달
- 마지막 업데이트 시간을 서버에 전달하여 변경된 코드 정보만 요청할 수 있도록 함

#### data.entities.request.ALBookmarkDeleteRequest
##### 역할
- 즐겨찾기 삭제 API 호출 시 필요한 출발지/도착지 공항 코드를 서버에 전달
- 삭제 대상이 되는 즐겨찾기 노선을 식별하기 위해 사용됨