# Code Review (AL)
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
- ViewModel의 UI 상태를 옵저빙하여 다이얼로그, 토스트와 같은 공통 UI 처리
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
    - 로딩, 에러, 알림과 같은 UI에 보여줄 상태를 표현하는 구조
##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|compositeDisposable|protected|ViewModel이 사라질 때 메모리 누수 방지|lazy|
|_uiState, uiState|private|ViewModel에서 UI로 단방향 상태 전달|Event래퍼로 LiveData가 UI에 한 번만 이벤트 전달할 수 있도록 함|

|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|callToRemote|다수|protected|서버 API 호출을 실행하고, 결과에 따라 onSuccess, onError, onFailed, ALUiState로 UI에 상태를 전달하는 메서드|single<T>로 서버 api 호출|
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


## al.data.datasource

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
- 프로모션 배너, 팝업, 결제 수단, 약관, 카드 BIN 정보
- 앱 내에서 공통으로 재사용되는 정보들을 조회

#### data.datasource.ALReservationDataSource
##### 역할
- 예약 생성, 결제 요청, 환불 요청, 사유 기반 환불
- 티켓/여정 예약 전후 단계의 API

#### data.datasource.ALTicketDataSource
##### 역할
- 발권/예약 내역 조회
- 예약 리스트, 상세 정보, 동기화 정보, 사용 이력을 서버로부터 가져옴

#### data.datasource.ALTravelDataSource
##### 역할
- 여행 상품 관련 정보 조회
- 여행 리스트, 요금 규칙 (출발 전/후/할인) 서버에서 조회

## al.data.entities

#### data.entities.request.ALBaseCodeInfoRequest
##### 역할
- 기초 코드 정보 조회 API 호출 시 요청 파라미터 전달
- 마지막 업데이트 시간을 서버에 전달하여 변경된 코드 정보만 요청할 수 있도록 함

#### data.entities.request.ALBookmarkDeleteRequest
##### 역할
- 즐겨찾기 삭제 API 호출 시 필요한 출발지/도착지 공항 코드를 서버에 전달
- 삭제 대상이 되는 즐겨찾기 노선을 식별하기 위해 사용

#### data.entities.request.ALBookmarkSaveRequest
##### 역할
- 즐겨찾기 노선 저장 API 호출 시 필요한 요청 데이터 전달
- 출발지/도착지 공항 코드, 고정 핀 여부, 별명 정보를 서버에 전달하여 즐겨찾기 노선을 등록

#### data.entities.reuqest.ALCardInfoRequest
##### 역할
- 카드 BIN 정보를 기반으로 카드사 정보를 조회할 때 사용되는 요청 모델
- 사용자가 입력한 카드 번호 앞 6자리를 cardBin으로 서버에 전달함

#### data.entities.request.ALFareDiscountRuleRequest
##### 역할
- 항공 운임에 적용 가능한 신분 할인 규칙 조회를 위한 요청 모델
- 여정별 운임 정보(IdFareInfo) 리스트를 서버에 전달하여 각 여정에 대한 할인 가능 여부와 금액을 계산
- 편도는 1개, 왕복/다구간은 2개 이상의 IdFareInfo 포함

##### 내부 필드 역할
|필드명|설명|
|:-----:|:---:|
|carrierCode|운항 항공사 코드|
|deptAirport/arrAirport|출발과 도착 공항 코드|
|departureDate/arrivalDate|출발과 도착 일시|
|flightNumber|편명|
|cabinClass|좌석 클래스|
|bookingClass|운임 클래스|
|airFare|일반 항공 운임|
|aireFareChild|아동 항공 운임|
|airTax|성인 공항세|
|airTaxChild|아동 공항세|
|airline|예약 항공사 코드|
|journeykey|여정 키 값|
|fakrekey|운임 키 값|

#### data.entities.request.ALPaymentMethodRequest
##### 역할
- 결제 수단 목록 조회 API 호출 시 사용하는 요청
- utlzSvcDvsCd 필드로 서비스 구분 코드를 서비스를 전달하며 해당 서비스에 맞는 결재 수단을 필터링 함

#### data.entities.request.ALPaymentRequest
##### 역할
- 항공권 결제 요청 시 필요한 여정, 탑승객, 카드, 운임 정보을 서버에 전달하기 위한 요청
- 성인/아동 운임, 프로모션 할인 금액, 카드정보, 결제 수단, 이지페이 정보와 같은 결제에 필요한 모든 데이터를 포함

|필드명|설명|
|:-----:|:---:|
|orderKey|주문번호|
|journeys(CardInfo)|각 여정별 결제 수단|
|pnrInfos|여정 예약 정보|
|passengerInfos|탑승객 정보|
|totalAirFare, totalFare|운임 총액 (세금 포함 여부에 따라 구분)|
|totalPromotionDiscountAmount 외|다양한 할인 금액 합계|
|CardInfo|카드 결제 수단 및 관련 정보, 결제 금액|
|FreeBaggage|여정별 무료 수하물 서비스 정보|
|EasyPayInfo|이지페이 결제에 필요한 인증값 (대한항공에서 사용됨)|

#### data.entities.request.ALPostFareRuleRequest
##### 역할
- 결제 이후 적용되는 운임 규칙을 조회할 떄 사용
- orderKey와 detailOrderId를 서버에 전달하여 해당 예약 건에 대한 후속 운임 규칙을 확인

#### data.enitities.request.ALPreFareRuleRequest
##### 역할
- 항공권 예약 이전 단계에서 운임 규칙을 조회할 때 사용하는 요청 모델
- 여정 키, 운임 키, PAIR 키, 항공사 코드, 출발/도착 공항, 출발일자와 같은 여정의 식별 정보를 기반으로 서버에 운임 규칙을 요청함
- 선택적으로 프로모션 ID도 함께 전달

#### data.entities.request.ALPromotionBannerRequest
##### 역할
- 프로모션 배너 조회 API 호출 시 사용되는 요청 모델
- 배너의 서비스 구분 코드(advrSvcDvsCd)와 노출 위치 코드(advrLocDvsCd)를 서버에 전달하여
- 특정 위치에 맞는 배너 리스트를 조회

#### data.entities.request.ALPromotionPopupRequest
##### 역할
- 프로모션 팝업 조회 API 호출 시 사용되는 요청
- 팝업의 서비스 구분 코드(advrSvcDvsCd)와 노출 위치 코드(advrLocDvsCd)를 서버에 전달하여
특정 화면에 띄울 팝업 광고 정보를 조회

#### data.entities.request.ALReservationCancelByReasonRequest
##### 역할
- 예약 취소 사유를 포함한 환불 요청을 서버에 전달할 때 사용하는 요청
- 예약 코드(orderKey)와 함께 취소 사유 코드(qqCode)와 여정 및 탑승객 정보(cancelInfos)를 포함하여
부분 취소 또는 특정 사유 기반 환불 요청을 처리

#### data.entities.request.ALReservationCancelRequest
##### 역할
- 항공 예약 건에 대해 전체 또는 부분 취소 요청을 서버에 전달할 때 사용
- 예약 코드와 여정 정보 및 탑승객별 운임/개인 정보를 포함한 상세 취소 요청 데이터를 서버에 전달

#### data.entities.request.ALReservationCompleteDetailRequest
##### 역할
- 예약 완료 후 특정 여정에 대한 상세 정보를 조회할 때 사용
- 여정 식별을 위한 일련번호(jrnySno)를 서버에 전달

#### data.entities.request.ALReservationRequest
##### 역할
- 항공 예약 요청 시 서버에 전달되는 전체 예약 정보 모델
- 여정(Journey), 예약자(Contact), 탑승객(PassengerInfo) 정보를 포함하여 예약 API 호출 시 사용

#### data.entities.request.ALRollingBannerRequest
##### 역할
- 슬라이드 배너 정보를 조회할 때 사용
- 서비스 구분 코드(advrSvcDvsCd)와 노출 위치 코드(advrLocDvsCd)를 서버에 전달하여 홈 화면과 같은 특정 위치에 띄울 롤링 배너 정보를 서버에서 가져올 때 사용

#### data.entities.request.ALSelectTicketHisListRequest
##### 역할
- 탑승 완료된 티켓 내역을 조회할 때 서버에 전달
- 사용자가 자신의 탑승 이력을 페이징 처리하여 조회할 수 있도록 pageNo 값을 전달하여 요청

#### data.entities.request.ALTermsListRequest
##### 역할
<<<<<<< HEAD
- 약관 목록을 조회할 때 서버에 전달
- 연동 서비스 코드(stplGrpCd)와 약관코드(stplDvsCd)를 서버에 전달하여 사용자가 특정 연동 서비스에서 사용할 약관 리스트를 요청

#### data.entities.request.ALTravelListRequest
##### 역할
- 항공편 검색 조건을 서버에 전달
- 사용자가 출발지/도착지/날짜/좌석등급/구분에 맞는 탑승객 수를 선택한 정보를 기반으로 서버에 운항 스케줄 리스트를 조회할 때 사용


#### data.entities.response.ALBaseCodeInfoResponse
##### 역할
- 공통 코드 및 공항 정보 조회 API
- 앱 실행 시 또는 특정 시점에 서버로부터 기준 코드와 공항 데이터를 받아 저장하거나 갱신할 때 사용

#### data.entities.response.ALBaseResponse
##### 역할
- 모든 API 응답의 공통 베이스 클래스
- 실제 데이터와 에러 코드, 메시지를 포함

#### data.entities.response.ALBookmarkDeleteResponse
##### 역할
- 빈 클래스

#### data.entities.response.ALBookmarkListResponse
##### 역할
- 즐겨찾기 목록 조회 API
- 서버로부터 받은 즐겨찾기 경로 리스트(jrnyList)를 담고 있음

#### data.entities.response.ALBookmarkSaveResponse
##### 역할
- 즐겨찾기 저장 API
- 즐겨찾기 경로가 성공적으로 저장된 등록 일시(rgtDtm)를 서버로부터 받아오기 위한 데이터 클래스

#### data.entities.response.ALCardInfoResponse
##### 역할
- 카드 정보 조회 API
- 입력된 카드 정보를 기준으로 카드에 대한 이름, 승인 기관 코드 명, 체크/법인 여부, 발급사를 서버로부터 받아오는 데 사용

#### data.entities.response.ALFareDiscountRuleResponse
##### 역할
- 신분 할인 운임 정보 조회
-  가는편(goIdFareRules)과 오는편(backIdFareRules) 각각에 대해 성인(adt)/소아(chd)/유아(inf)(paxType) 별 할인 정보(FareDiscountRule)를 포함함
- 할인 명칭, 신분 할인 코드, 운임/세금 및 할인율과 같은 정보를 포함
- 특정 여정에 대해 탑승객 신분에 따라 적용 가능한 운임 할인 규칙들을 서버에서 받아오는 데 사용

#### data.entities.response.ALFareRuleResponse
##### 역할
- 운임 규정 정보 응답
- 항공권 예매 시 적용되는 다양한 운임 규정 정보를 담는 모델
- 규정명, 설명, 정렬 순서, 유형 정보를 포함

#### data.entities.response.ALPaymentMethodResponse
##### 역할
- 결제수단 정보 조회
- 앱에서 결제수단 화면에 출력할 문구, 노출 여부를 서버에서 받아오는 역할
결제 화면에 보여줄 문구 및 표시 여부를 서버에서 받아옴

#### data.entities.response.ALPaymentRespone
##### 역할
- 결제 요청 후 서버로부터 받은 응답
- 결제가 완료된 여정 정보, 예약 키, 상세 예약 키 목록을 포함

#### data.entities.response.ALPromotionBannerResponse
##### 역할
- 배너 광고 정보 조회
- 서버에서 내려준 광고 목록 데이터를 담고 있음
- advrList: 광고 배너 데이터 리스트 (각 항목은 ResponseInqrAdvrPupStupInf.Data.AdvrList 타입, 별도 모듈 tbike에서 정의된 광고 정보 모델을 재사용)

#### data.entities.response.ALPromotionPopupResponse
##### 역할
- 팝업 형태의 광고 또는 공지사항 응답 모델
- 서버에서 내려준 광고 리스트(advrList) 또는 공지사항 리스트(ntcMttrList) 중 하나를 담고 있음
- actScsTyp 값에 따라 어떤 종류의 팝업을 보여줄지를 판단하고
해당 목록을 UI에 바인딩하는 데 사용

#### data.entities.response.ALReservationCancelByReasonResponse
##### 역할
- 사유 기반 예약 취소 요청에 대한 응답
- 취소된 예약 정보 리스트(cancelResult)를 통해 취소된 예약 정보 및 탑승객 목록을 반환함
- 각각의 항목(CancelInfo)은 개별 예약(orderKey, orderId)과
해당 예약에 포함된 취소된 탑승객(CanceledPassengerInfo) 목록을 포함
- 사용자가 특정 사유를 선택하여 예약을 취소했을 때 서버로부터 수신되는 취소 성공 결과를 담음

#### data.entities.response.ALReservationCancelResponse
##### 역할
- 일반 예약 취소 요청의 결과를 나타냄
- 취소 요청 결과와 여정 상태를 클라이언트에 전달
- 예약 취소가 성공했는지, 추가 취소 접수가 필요한지, 여정의 상태가 어떤지를 확인
##### 여정 상태 코드
|코드|설명|
|:-----:|:---:|
|OK|예약완료|
|XX|취소완료|
|QQ|취소요청|
|HL|발권대기|

#### data.entities.response.ALReservationCompleteDetailResponse
##### 역할
- 예약이 완료된 후 해당 예약 여정에 대한 상세 정보를 조회하기 위한 데이터 모델
- 예약/주문 변호, 예약자/탑승객, 항공사 코드/명, 결제 정보, 항공편 정보와 같은 모든 예약 결과의 종합 정보 제공

##### 데이터 설명
|항목|설명|
|:-----:|:---:|
|jrnySno|여정 일련번호|
|jrnyStatu|여정 상태 (OK, XX, HL, QQ)|
|pnrNumber|예약번호(PNR)|
|orderKey|주문 번호|
|passengerInfos|탑승객 리스트 및 각자의 요금/환불 정보|
|airline, airlineName|항공사 코드 및 이름|
|deptAirport, arrAirport|출발/도착 공항 코드|
|departureDate, arrivalDate|출발/도착 일시|
|flightType, flightNumber|여정 타입 및 항공편명|
|pairKey|여정 조합 키 (왕복과 같은 구분용)|
|cabinClass, totalFlightTime|좌석 등급 및 총 비행시간|
|freessrName, freessrValue|무료 수화물 정보|
|aprvDtm, aprvInfo|결제 일시 및 정보|
|cancDtm, cancFee, cancTotalAmt|취소 일시, 수수료, 총 환불 금액|
|shareAirlineName|공동 운항 항공사 명|
|PassengerInfo|탑승객 인덱스, 타입, 성별, 생년월일, 요금 정보, 환불 정보 포함|
|FareInfo|해당 여정에 대한 운임, 세금, 유류세, 수수료와 같은 상세 정보|
|Refund|환불 금액 정보|
|Name|탑승객 이름 구조(first, middle, last)|

#### data.entities.response.ALReservationCompleteListResponse
##### 역할
- 항공 예약 시스템에서 사용자의 예약 완료 목록 조회 시 사용되는 응답
- 예약이 완료된 항공권들의 여정 목록 정보를 조회

#### data.entities.response.ALReservationResponse
##### 역할
- 항공 예약 완료 후, 주문번호 및 여정, 탑승객, 결제금액, 할인 정보를 포함한 전체 예약 정보를 서버로부터 응답 받음
- 사용자에게 예매 완료를 표시하기 위한 핵심 정보들이 포함
##### 주요 구성 정리
1. 주문/결제 정보
2. journeys(여정 정보 리스트)
3. passengerInfos(탑승객 정보)
4. FareInfo & TotalFareInfo

#### data.entities.response.ALRollingBannerResponse
##### 역할
- 앱 홈 화면 또는 주요 화면 상단에 노출되는 롤링 배너 정보를 응답 받음
- 앱 홈 또는 특정 화면에 공지/광고 배너를 띄워야 할지 판단하고 필요한 정보를 표시하기 위해 사용
##### 실행 성공 유형
|유형|설명|
|:-----:|:---:|
|T|공지|
|A|광고|
|N|응답 없음|

#### data.entities.response.ALTermsListResponse
##### 역할
- 약관 목록을 받아오는 응답

#### data.entities.response.ALTravelListResponse
##### 역할
- 항공편 조회 결과를 담음
- 출발지, 도착지, 날짜를 기반으로 가는편/오는편 항공편 리스트 조회

#### data.entities.response.ALUsingListRespoonse
##### 역할
- 사용자 탑승 이력 또는 사용 중인 여정 목록을 조회하는 데 사용
- 사용자가 예약/탑승한 항공편의 이용 내역을 보여주는 응답

## al.data.enums

### 역할
앱에서 사용되는 상수값들을 타입 안전하게 관리하기 위한 enum 클래스들

### Code

#### data.enums.ALAirportSelectType
- 공항 선택 단계를 구분하는 enum 클래스
- 편도/왕복/다구간 여행에서 출발지/도착지 선택 시 현재 선택 단계를 식별하기 위해 사용

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|code|val|정수형 식별 코드|각 선택 타입별 고유 번호|

|enum 값|code|설명|
|:-----:|:---:|:---:|
|FIRST_DEPARTURE|1|첫 번째 여정 출발지|
|FIRST_ARRIVAL|2|첫 번째 여정 도착지|
|SECOND_DEPARTURE|3|두 번째 여정 출발지|
|SECOND_ARRIVAL|4|두 번째 여정 도착지|

##### 역할
- 다구간/왕복 여행에서 공항 선택 순서 관리
- UI에서 현재 선택해야 할 공항이 출발지인지 도착지인지 구분
- 첫 번째/두 번째 여정 구분을 통한 복수 여정 처리

#### data.enums.ALDeepLinkType
- 앱 딥링크 진입점을 구분하는 enum 클래스
- 외부에서 앱으로 진입할 때 어느 화면으로 이동할지 결정하기 위해 사용

##### 변수 및 함수
|enum 값|설명|
|:-----:|:---:|
|MAIN|메인 화면으로 이동|
|RESERVATION_DETAIL|예약 상세 화면으로 이동|

##### 역할
- 딥링크를 통한 특정 화면 직접 접근 관리
- 외부 링크나 푸시 알림을 통한 앱 진입점 제어
- 사용자 경험 향상을 위한 직접 화면 이동

#### data.enums.ALJourneyType & ALMultiWayType
- 항공 여행의 여정 타입을 구분하는 enum 클래스들
- 편도, 왕복, 다구간 여행을 식별하고 다구간에서는 순서를 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|code|val|정수형 식별 코드|각 여정 타입별 고유 번호|

|enum 값 (ALJourneyType)|code|설명|
|:-----:|:---:|:---:|
|ROUND_TRIP|1|왕복 여행|
|ONE_WAY|2|편도 여행|
|MULTI_WAY|3|다구간 여행|

|enum 값 (ALMultiWayType)|code|설명|
|:-----:|:---:|:---:|
|FIRST|1|첫 번째 구간|
|SECOND|2|두 번째 구간|
|NONE|2|해당 없음|

##### 역할
- 항공권 검색 시 여정 타입 구분
- 다구간 여행에서 각 구간별 순서 관리
- 요금 계산 및 검색 조건 설정에 활용

#### data.enums.ALPaymentMethod
- 결제 수단을 구분하는 enum 클래스
- 항공권 결제 시 사용할 결제 방식을 식별하기 위해 사용

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|code|val|정수형 식별 코드|각 결제 방식별 고유 번호|

|enum 값|code|설명|
|:-----:|:---:|:---:|
|EASY_PAY|1|간편결제|
|GENERAL_PAY|2|일반결제|
|NONE|3|결제 방식 미선택|

##### 역할
- 결제 플로우에서 결제 방식 구분
- UI에서 결제 옵션 표시 및 선택 상태 관리
- 결제 API 호출 시 결제 타입 전달

#### data.enums.ALPassengerType
- 탑승객 타입을 구분하는 enum 클래스
- 성인, 아동, 유아를 식별하여 요금 계산 및 좌석 배정에 활용

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|code|val|정수형 식별 코드|각 탑승객 타입별 고유 번호|

|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|getPaxType|-|public|IATA 표준 탑승객 코드 반환|ADT/CHD/INF 문자열|
|getName|-|public|탑승객 타입 한글명 반환|ALGlobalData에서 조회|

|enum 값|code|getPaxType()|설명|
|:-----:|:---:|:---:|:---:|
|ADULT|1|ADT|성인 탑승객|
|CHILD|2|CHD|아동 탑승객|
|INFANT|3|INF|유아 탑승객|

##### 역할
- 항공 요금 계산 시 탑승객별 차등 적용
- IATA 표준 코드 변환을 통한 항공사 API 연동
- 탑승객 정보 관리 및 좌석 할당 규칙 적용

#### data.enums.ALSeatType
- 항공기 좌석 등급을 구분하는 enum 클래스
- 이코노미/비즈니스 클래스 구분과 화면 표시명, 항공사 코드를 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|code|val|정수형 식별 코드|각 좌석 등급별 고유 번호|
|displayName|val|화면 표시용 좌석명|다국어 리소스에서 가져온 문자열|
|seatName|val|항공사 표준 좌석 코드|Y(이코노미), C(비즈니스)|

|enum 값|code|displayName|seatName|설명|
|:-----:|:---:|:---:|:---:|:---:|
|ECONOMY|1|이코노미석|Y|일반석|
|BUSINESS|2|비즈니스석|C|비즈니스석|

##### 역할
- 항공권 검색 시 좌석 등급 필터링
- 요금 조회 및 예약 시 좌석 클래스 구분
- UI에서 좌석 등급 표시 및 선택 옵션 제공

#### data.enums.ALTextStyle
- 텍스트 스타일을 구분하는 enum 클래스
- 앱 내 텍스트 표시 시 일관된 폰트 스타일 적용을 위해 사용

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|style|val|안드로이드 스타일 리소스 ID|R.style 참조|

|enum 값|style|설명|
|:-----:|:---:|:---:|
|REGULAR|R.style.ALRegularText|일반 폰트 굵기|
|MEDIUM|R.style.ALMediumText|중간 폰트 굵기|
|BOLD|R.style.ALBoldText|굵은 폰트|

##### 역할
- UI 텍스트의 일관된 스타일 관리
- 디자인 시스템 기반 텍스트 스타일 적용
- 커스텀 뷰나 컴포넌트에서 동적 스타일 설정

## al.data.model.appmodel

### 역할
앱 내부에서 사용되는 핵심 데이터 모델들을 정의하는 클래스들

### Code

#### data.model.appmodel.ALAppAirportInfo
- 공항 정보를 담는 데이터 클래스
- 항공권 검색 및 예약 시 출발지/도착지 공항 정보를 관리하기 위해 사용

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|airport|val|공항 코드|nullable String|
|airportArea|val|공항 지역명|nullable String|

##### 역할
- 공항 선택 화면에서 공항 정보 표시
- 여정 정보에서 출발지/도착지 공항 데이터 저장
- 검색 조건 및 예약 정보에 공항 코드 제공

#### data.model.appmodel.ALAppEasyPayRequestInfo
- 간편결제 요청 시 필요한 정보를 담는 데이터 클래스
- 대한항공과 같은 특정 항공사의 간편결제 연동 시 사용되는 여정별 결제 정보 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|isFirstJourneyKoreanAir|val|첫 번째 여정 대한항공 여부|nullable Boolean|
|firstOrderId|val|첫 번째 여정 주문 ID|nullable String|
|firstArrivalAirportCode|val|첫 번째 여정 도착 공항 코드|nullable String|
|firstCardAmount|val|첫 번째 여정 결제 금액|발권 대행료 제외, nullable String|
|isSecondJourneyKoreanAir|val|두 번째 여정 대한항공 여부|nullable Boolean|
|secondOrderId|val|두 번째 여정 주문 ID|nullable String|
|secondArrivalAirportCode|val|두 번째 여정 도착 공항 코드|nullable String|
|secondCardAmount|val|두 번째 여정 결제 금액|발권 대행료 제외, nullable String|

##### 역할
- 다구간/왕복 여행에서 여정별 간편결제 정보 관리
- 대한항공 전용 간편결제 시스템 연동
- 여정별 결제 금액 및 주문 정보 분리 처리

#### data.model.appmodel.ALAppFareInfo
- 항공 운임 정보를 담는 데이터 클래스
- 탑승객별 요금 세부 내역을 관리하며 결제 화면에서 요금 표시에 사용

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|airFare|var|항공 운임료|Int|
|airTax|var|항공세|Int|
|fuelCharge|var|유류세|Int|
|tkFee|var|수수료|Int|
|totalFare|var|합계 금액|Int|
|isFirstIndex|var|첫 번째 인덱스 여부|Boolean|
|isFirstJourney|var|첫 번째 여정 여부|Boolean|
|name|var|탑승객 이름|String|
|passengerType|var|탑승객 구분|ALPassengerType enum|

##### 역할
- 탑승객별 요금 세부 내역 계산 및 표시
- 여정별 요금 정보 구분 관리
- 결제 화면에서 요금 구성 요소별 금액 표시

#### data.model.appmodel.ALAppJourneyInfo
- 여정 정보를 담는 핵심 데이터 클래스
- 항공권 검색부터 예약까지 전체 여정 정보를 관리하며 유효성 검증 로직 포함

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|journeyType|var|여정 타입|ALJourneyType enum, 기본값 왕복|
|departureDate|var|출발일|Calendar?, nullable|
|arrivalDate|var|도착일/복귀일|Calendar?, nullable|
|firstDepartureAirport|var|첫 번째 출발지 공항|ALAppAirportInfo?, nullable|
|firstArrivalAirport|var|첫 번째 도착지 공항|ALAppAirportInfo?, nullable|
|secondDepartureAirport|var|두 번째 출발지 공항|ALAppAirportInfo?, 다구간용|
|secondArrivalAirport|var|두 번째 도착지 공항|ALAppAirportInfo?, 다구간용|
|passengerInfo|var|승객 정보|ALAppPassengerInfo?, nullable|

|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|isEnable|-|public|여정 정보 유효성 검증|여정 타입별 필수 조건 확인|

##### 역할
- 전체 여정 정보의 중앙 집중 관리
- 편도/왕복/다구간 여행 타입별 유효성 검증
- 항공권 검색 API 호출 시 필요한 모든 조건 제공
- 사용자 입력 완성도 체크 및 다음 단계 진행 가능 여부 판단

#### data.model.appmodel.ALAppPassengerInfo
- 탑승객 정보를 담는 데이터 클래스
- 성인/아동/유아 탑승객 수와 좌석 등급을 관리하며 탑승객 수 조절 기능 제공

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|adultCount|var|성인 탑승객 수|Int, 기본값 1|
|childCount|var|아동 탑승객 수|Int, 기본값 0|
|infantCount|var|유아 탑승객 수|Int, 기본값 0|
|seatType|var|좌석 등급|ALSeatType enum, 기본값 이코노미|

|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|setNumberMinus|ALPassengerType|public|특정 타입 탑승객 수 감소|체이닝 가능|
|setNumberPlus|ALPassengerType|public|특정 타입 탑승객 수 증가|체이닝 가능|
|totalNumber|-|public|전체 탑승객 수 반환|Int 반환|

##### 역할
- 항공권 검색 시 탑승객 조건 설정
- 탑승객 선택 UI에서 인원 수 증감 처리
- 요금 계산 시 탑승객 타입별 인원 수 제공
- 좌석 등급 선택 및 관리

#### data.model.appmodel.ALAppTravelFilter
- 항공편 검색 결과 필터링을 위한 조건들을 담는 데이터 클래스
- 사용자가 설정한 출발시간, 운임가격, 항공사 필터 조건을 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|departureDateRange|var|출발시간 범위|Pair<String, String>?, HHMM 형식|
|freightPrice|var|운임가격 범위|Pair<Int, Int>?, 최소-최대 금액|
|airlineList|val|선택된 항공사 목록|ArrayList<ALBaseCodeInfoModel.Code>|

##### 역할
- 항공편 검색 결과 화면에서 필터 조건 적용
- 사용자 맞춤 항공편 검색 결과 제공
- 출발시간대, 가격대, 선호 항공사별 결과 필터링
- 검색 결과 최적화를 통한 사용자 경험 향상



## al.data.model

### 역할
서버 응답 데이터를 앱 내부에서 사용하는 모델로 변환하고 관리하는 클래스들

### Code

#### data.model.ALBaseModel
- 모든 응답 모델의 기반이 되는 기본 클래스
- 서버 응답의 공통 에러 처리 및 성공/실패 판단 로직을 제공

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|errorCode|var|에러 코드|"0000" 정상, 이외 에러|
|errorMessage|var|에러 메시지|에러 발생 시 사용|
|duplicateLogin|var|중복 로그인 여부|Boolean, 기본값 false|

|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|isSuccess|-|public|정상 응답 여부 확인|errorCode가 "0000"인지 검사|
|isDuplicateLogin|-|public|중복 로그인 여부 확인|duplicateLogin 값 반환|

##### 역할
- 모든 API 응답 모델의 공통 기반 제공
- 에러 코드 기반 성공/실패 판단 표준화
- 중복 로그인 감지 및 처리
- 일관된 에러 처리 패턴 제공

#### data.model.ALBaseCodeInfoModel
- 서버에서 제공하는 기초 코드 정보와 공항 정보를 관리하는 모델
- 앱 전체에서 사용되는 공통 코드와 공항 데이터를 구조화하여 제공

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|codeList|val|공통 코드 목록|List<Code>, 기본값 빈 리스트|
|airportList|val|공항 정보 목록|List<Airport>, 기본값 빈 리스트|

|내부 클래스|설명|
|:-----:|:---:|
|Code|공통 코드 정보 (코드그룹, 코드, 코드명, 정렬순서, 사용여부, 체크여부, 부가속성1~9)|
|Airport|공항 정보 (공항명, 공항코드, 공항지역)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALBaseCodeInfoModel로 변환|정렬 및 매핑 처리|

##### 역할
- 서버 기초 코드 데이터의 구조화된 관리
- 공항 정보 및 공통 코드의 앱 내 표준 형식 제공
- 서버 응답의 snake_case를 camelCase로 변환
- 코드 사용 여부 및 정렬 순서 관리

#### data.model.ALBookmarkDeleteModel
- 즐겨찾기 삭제 API 응답을 처리하는 모델
- 삭제 요청의 성공/실패 여부만 확인하는 단순 응답 모델

##### 변수 및 함수
|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALBookmarkDeleteModel로 변환|에러 정보만 설정|

##### 역할
- 즐겨찾기 삭제 요청의 결과 처리
- 기본 에러 처리 로직만 제공
- 삭제 성공/실패 여부 확인

#### data.model.ALBookmarkListModel
- 즐겨찾기 목록 조회 API 응답을 처리하는 모델
- 사용자가 저장한 즐겨찾기 노선 목록을 ALBookmarkEntity로 변환하여 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|bookmarkList|val|즐겨찾기 목록|List<ALBookmarkEntity>, 기본값 빈 리스트|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALBookmarkListModel로 변환|서버 응답을 ALBookmarkEntity로 매핑|

##### 역할
- 사용자 즐겨찾기 노선 목록 관리
- 서버 응답 데이터를 Room Entity 형태로 변환
- 고정 핀 여부, 별명, 생성시간과 같은 메타 정보 포함
- 로컬 DB 저장을 위한 데이터 구조 제공

#### data.model.ALBookmarkSaveModel
- 즐겨찾기 저장 API 응답을 처리하는 모델
- 즐겨찾기 저장 시 서버에서 반환되는 생성시간 정보를 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|createTime|val|생성시간|Long, 기본값 0L|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALBookmarkSaveModel로 변환|생성시간 추출 및 에러 정보 설정|

##### 역할
- 즐겨찾기 저장 요청의 결과 처리
- 서버에서 부여한 생성시간 정보 관리
- 로컬 DB 동기화를 위한 타임스탬프 제공

#### data.model.ALCardInfoModel
- 카드 BIN 정보 조회 API 응답을 처리하는 모델
- 사용자가 입력한 카드 번호 앞 6자리로 카드사 정보를 조회한 결과를 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|cardName|val|카드명|nullable String|
|crpcNm|val|승인기관코드명|nullable String|
|checkCardYn|val|체크카드 여부|nullable String|
|cprtCardYn|val|법인카드 여부|nullable String|
|issurCd|val|발급사 코드|nullable String|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALCardInfoModel로 변환|rstCode 200을 성공으로 변환|

##### 역할
- 결제 시 카드 정보 유효성 검증
- 카드사별 결제 옵션 제공
- 체크카드/법인카드 구분을 통한 결제 방식 결정
- 카드 BIN 기반 자동 카드사 인식

#### data.model.ALFareDiscountRuleModel
- 신분 할인 규칙 조회 API 응답을 처리하는 모델
- 여정별, 탑승객 타입별 신분 할인 정보를 구조화하여 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|firstFareDiscountRuleList|val|첫 번째 여정 할인 규칙|FirstFareRules?, nullable|
|secondFareDiscountRuleList|val|두 번째 여정 할인 규칙|FirstFareRules?, nullable|

|내부 클래스|설명|
|:-----:|:---:|
|FirstFareRules|성인/아동/유아별 할인 규칙 그룹|
|FareDiscountRule|개별 할인 규칙 (승객구분, 할인명, 할인코드, 운임/세금 할인률)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALFareDiscountRuleModel로 변환|가는편/오는편 할인 규칙 분리 매핑|

##### 역할
- 탑승객 타입별 신분 할인 옵션 제공
- 여정별 할인 규칙 분리 관리
- 할인률 기반 요금 계산 지원
- 예약 시 적용 가능한 할인 혜택 표시

#### data.model.ALFareRuleModel
- 운임 규칙 조회 API 응답을 처리하는 모델
- 항공권 예약/취소 시 적용되는 운임 규정을 구조화하여 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|fareRuleList|val|운임 규칙 목록|List<FareRule>, 기본값 빈 리스트|

|내부 클래스|설명|
|:-----:|:---:|
|FareRule|개별 운임 규칙 (규정명, 규정설명, 규정유형)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALFareRuleModel로 변환|정렬 순서 적용 후 매핑|

##### 역할
- 항공권 구매 전 운임 규정 안내
- 취소/변경 규정 및 수수료 정보 제공
- 규정 유형별 분류 (일반/취소/기타)
- 사용자 동의 및 확인을 위한 규정 텍스트 관리


#### data.model.ALPaymentCardModel
- 결제 카드 정보를 관리하는 모델
- 기존 카드 정보(CardInfo)를 항공 결제용 카드 모델로 변환하여 보안 처리된 카드 데이터 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|cardList|val|결제 카드 목록|MutableList<PaymentCard>, 기본값 빈 리스트|

|내부 클래스|설명|
|:-----:|:---:|
|PaymentCard|개별 카드 정보 (카드번호, 보안데이터, 유효기간, 카드명, 카드사)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|List<CardInfo>를 ALPaymentCardModel로 변환|plainString으로 카드번호 복호화|

##### 역할
- 기존 등록 카드를 항공 결제용으로 변환
- 카드번호 보안 처리 및 분할 관리
- 메인 카드 설정 및 카드 종류 구분
- 결제 시 카드 선택 옵션 제공

#### data.model.ALPaymentMethodModel
- 결제 수단 정보를 관리하는 모델
- 특정 서비스에서 사용 가능한 결제 방식과 프로모션 정보를 제공

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|promotionList|val|프로모션 목록|List<String>, 기본값 빈 리스트|
|isShowEasyPay|val|간편결제 표시 여부|Boolean, 기본값 false|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALPaymentMethodModel로 변환|rstCode 200을 성공으로 처리|

##### 역할
- 서비스별 사용 가능한 결제 수단 필터링
- 간편결제 노출 여부 제어
- 결제 관련 프로모션 정보 제공
- 결제 화면 UI 구성을 위한 옵션 관리

#### data.model.ALPaymentModel
- 결제 완료 후 응답 정보를 관리하는 모델
- 여정별 결제 결과와 상세 주문 ID를 구조화하여 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|journeys|val|여정별 결제 정보|List<Journey>, 기본값 빈 리스트|

|내부 클래스|설명|
|:-----:|:---:|
|Journey|개별 여정 결제 정보 (예약키, 상세주문ID 목록)|
|DetailOrderId|상세 주문 정보 (주문ID)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALPaymentModel로 변환|여정별 결제 정보 매핑|

##### 역할
- 결제 완료 후 예약 번호 및 주문 정보 관리
- 여정별 결제 결과 분리 저장
- 예약 조회 및 관리를 위한 키 정보 제공
- 후속 처리(발권, 취소)를 위한 참조 데이터

#### data.model.ALPromotionBannerModel
- 프로모션 배너 정보를 관리하는 모델
- 특정 위치에 표시할 배너 이미지와 링크 정보를 제공

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|firstBanner|val|첫 번째 배너|ResponseInqrAdvrPupStupInf.Data.AdvrList?, nullable|
|secondBanner|val|두 번째 배너|ResponseInqrAdvrPupStupInf.Data.AdvrList?, nullable|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALPromotionBannerModel로 변환|첫 번째/두 번째 배너 분리|

##### 역할
- 메인 화면 프로모션 배너 관리
- 최대 2개 배너 동시 표시 지원
- 배너 클릭 시 이동할 링크 정보 제공
- 광고 노출 위치별 배너 컨텐츠 관리

#### data.model.ALPromotionPopupModel
- 프로모션 팝업 정보를 관리하는 모델
- 앱 진입 시 표시할 팝업 광고나 공지사항을 구분하여 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|activeType|val|실행 성공 유형|String?, T(공지)/A(광고)/N(없음)|
|promotionList|val|광고 목록|List<ResponseInqrAdvrPupStupInf.Data.AdvrList>, 기본값 빈 리스트|
|notice|val|공지사항 정보|Notice?, nullable|

|내부 클래스|설명|
|:-----:|:---:|
|Notice|공지사항 정보 (공지유형, 공지내용)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALPromotionPopupModel로 변환|타입별 컨텐츠 분리 처리|

##### 역할
- 앱 실행 시 팝업 표시 여부 결정
- 광고와 공지사항 구분 관리
- 우선순위 기반 컨텐츠 표시 (공지 > 광고)
- 사용자 경험을 고려한 팝업 노출 제어

#### data.model.ALReservationCancelByReasonModel
- 사유 기반 예약 취소 요청 결과를 관리하는 모델
- 특정 취소 사유를 포함한 환불 요청의 처리 결과를 구조화

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|cancelResult|val|취소 결과 목록|List<CancelInfo>, 기본값 빈 리스트|

|내부 클래스|설명|
|:-----:|:---:|
|CancelInfo|취소 정보 (예약코드, 상세예약코드, 취소된 탑승객 정보)|
|CanceledPassengerInfo|취소된 탑승객 정보 (탑승객ID, 탑승객구분)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALReservationCancelByReasonModel로 변환|취소 결과 상세 정보 매핑|

##### 역할
- 부분 취소 결과 관리
- 탑승객별 취소 처리 상태 추적
- 취소 사유 기반 환불 처리 결과 제공
- 예약 관리 화면에서 취소 상태 표시

#### data.model.ALReservationCancelModel
- 일반 예약 취소 요청 결과를 관리하는 모델
- 예약 전체 또는 부분 취소 후 처리 상태와 후속 액션 정보를 제공

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|orderKey|val|상위 예약 코드|nullable String|
|detailOrderIdList|val|상세 예약 ID 목록|List<DetailOrderId>, 기본값 빈 리스트|
|actionCode|val|액션 코드|빈값(일반취소), "QQ"(취소접수요청)|
|journeySequenceNo|val|여정 일련번호|nullable String|
|journeyStatusCode|val|여정 상태 코드|HL/OK/XX/QQ|

|내부 클래스|설명|
|:-----:|:---:|
|DetailOrderId|상세 예약 ID 정보|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALReservationCancelModel로 변환|취소 처리 상태 및 액션 코드 매핑|

##### 역할
- 예약 취소 처리 결과 및 상태 관리
- 취소 완료 vs 취소 요청 구분
- 후속 화면 이동 로직 결정 (actionCode 기반)
- 여정 상태별 사용자 안내 메시지 제공

#### data.model.ALReservationCompleteDetailModel
- 예약 완료 상세 정보를 관리하는 종합 모델
- 탑승객, 운임, 일정, 결제와 같은 예약과 관련된 모든 세부 정보를 구조화

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|journeySequenceNo|val|여정 일련번호|nullable String|
|journeyStatusCode|val|여정 상태 코드|HL/OK/XX/QQ|
|pnrNumber|val|예약 번호|nullable String|
|orderKey|val|주문 번호|nullable String|
|passengerInfos|val|탑승객 정보 목록|List<PassengerInfo>, 기본값 빈 리스트|
|airline ~ shareAirlineName|val|항공편 정보|항공사, 공항, 일시, 좌석|

|내부 클래스|설명|
|:-----:|:---:|
|PassengerInfo|탑승객 상세 정보 (개인정보, 운임정보, 환불정보)|
|FareInfo|운임 상세 정보 (운임료, 세금, 수수료, 상태)|
|Refund|환불 정보|
|Name|이름 정보 (성, 이름, 중간이름)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALReservationCompleteDetailModel로 변환|날짜 포맷 변환 및 복합 데이터 매핑|

##### 역할
- 예약 완료 후 상세 정보 통합 관리
- 탑승객별 개별 정보 및 운임 내역 제공
- 예약 상태별 화면 표시 제어
- 취소/환불 시 참조 데이터 제공
- 날짜/시간 데이터의 사용자 친화적 변환
- Parcelable 지원으로 화면 간 데이터 전달 최적화


#### data.model.ALReservationCompleteListModel
- 예약 완료 목록 조회 API 응답을 처리하는 모델
- 사용자의 예약 내역을 목록 형태로 간략하게 표시하기 위한 요약 정보 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|journeyList|val|여정 목록|List<Journey>, 기본값 빈 리스트|

|내부 클래스|설명|
|:-----:|:---:|
|Journey|개별 여정 요약 정보 (일련번호, 상태, 항공사, 공항, 일시, 탑승객 수, 좌석등급)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALReservationCompleteListModel로 변환|여정별 요약 정보 매핑|

##### 역할
- 예약 내역 목록 화면에서 사용할 요약 정보 제공
- 여정 상태별 목록 필터링 및 표시
- 예약 상세 화면 진입을 위한 기본 정보 관리
- 탑승객 수 및 좌석 등급과 같은 핵심 정보 요약 표시

#### data.model.ALReservationModel
- 예약 생성 API 응답을 처리하는 종합 모델
- 예약 전체 정보를 상세하게 관리하며 여정, 탑승객, 운임 정보를 구조화

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|orderKey|val|주문번호|nullable String|
|journeys|val|여정 정보 목록|List<Journey>, 기본값 빈 리스트|
|passengerInfos|val|탑승객 정보 목록|List<PassengerInfo>, 기본값 빈 리스트|
|totalAirFare ~ promotionYn|val|운임 관련 필드들|각종 할인 금액 및 프로모션 정보|
|totalFareInfos|val|결제용 운임 정보|List<TotalFareInfo>?, nullable|

|내부 클래스|설명|
|:-----:|:---:|
|Journey|여정 상세 정보 (항공사, 공항, 일시, 세그먼트, PNR)|
|Segment|항공편 구간 정보 (예약등급, 좌석등급, 운행사, 비행시간)|
|Leg|개별 비행편 정보 (운행사, 공항, 일시, 대기시간)|
|PassengerInfo|탑승객 상세 정보 (개인정보, 운임정보, 환불정보)|
|FareInfo|운임 상세 정보 (각종 요금, 할인, 프로모션)|
|Refund|환불 정보|
|Name|이름 정보 (성, 이름, 중간이름)|
|TotalFareInfo|결제용 총 운임 정보|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALReservationModel로 변환|복합 구조의 상세 매핑|

##### 역할
- 예약 생성 후 전체 예약 정보의 통합 관리
- 다구간/왕복 여행의 복잡한 세그먼트 구조 처리
- 탑승객별 개별 운임 및 할인 정보 관리
- 각종 프로모션 및 할인 금액의 세부 계산
- 결제 시 필요한 운임 정보 제공
- PNR, 여정 키와 같은 항공사 연동을 위한 참조 데이터 관리

#### data.model.ALRollingBannerModel
- 롤링 배너 조회 API 응답을 처리하는 모델
- 메인 화면 상단에 표시할 슬라이드 배너 정보를 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|rollingBanners|val|롤링 배너 목록|List<ResponseInqrAdvrPupStupInf.Data.AdvrList>, 기본값 빈 리스트|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALRollingBannerModel로 변환|rstCode 200을 성공으로 처리|

##### 역할
- 메인 화면 롤링 배너 컨텐츠 관리
- 여러 배너의 슬라이드 표시 지원
- 배너 클릭 시 이동할 링크 정보 제공
- 광고 캠페인 및 프로모션 배너 노출

#### data.model.ALTermsListModel
- 약관 목록 조회 API 응답을 처리하는 모델
- 서비스 이용약관 및 개인정보처리방침과 같은 약관 정보를 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|termsInfoList|val|약관 정보 목록|List<TermsInfo>, 기본값 빈 리스트|

|내부 클래스|설명|
|:-----:|:---:|
|TermsInfo|개별 약관 정보 (외부URL, 일련번호, 제목)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALTermsListModel로 변환|첫 번째 약관 그룹의 상세 목록 추출|

##### 역할
- 앱 내 약관 및 정책 문서 목록 관리
- 약관 제목 및 외부 링크 정보 제공
- 약관 동의 화면에서 목록 표시
- 웹뷰를 통한 약관 내용 연결

#### data.model.ALTravelListModel
- 항공편 검색 결과 API 응답을 처리하는 종합 모델
- 가는편/오는편 항공편 목록과 상세 운임 정보를 구조화하여 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|firstTravelList|val|가는편 항공편 목록|List<Travel>, 기본값 빈 리스트|
|secondTravelList|val|오는편 항공편 목록|List<Travel>, 기본값 빈 리스트|

|내부 클래스|설명|
|:-----:|:---:|
|Travel|개별 항공편 정보 (항공사, 공항, 일시, 운임, 좌석, 프로모션)|
|FreeBaggage|무료 수하물 서비스 정보 (서비스명, 코드, 값, 대상, 단위)|
|PaxTypeFares|탑승객 타입별 운임 정보 (기본요금, 세금, 할인)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALTravelListModel로 변환|날짜 파싱 및 가는편/오는편 분리 매핑|

##### 역할
- 항공편 검색 결과의 통합 관리
- 가는편/오는편 구분 및 조합 선택 지원
- 탑승객 타입별 운임 정보 세부 제공
- 무료 수하물 정보 및 부가 서비스 안내
- 프로모션 및 할인 정보 적용
- 항공편 선택을 위한 비교 정보 제공
- 예약 진행을 위한 여정 키/운임 키 관리

#### data.model.ALUsingListModel
- 이용 내역 조회 API 응답을 처리하는 모델
- 과거 탑승 완료된 항공편의 이용 기록을 관리

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|usingList|val|이용 내역 목록|List<UsingList>, 기본값 빈 리스트|

|내부 클래스|설명|
|:-----:|:---:|
|UsingList|개별 이용 내역 (여정번호, 상태, 공항, 일시, 탑승객 수, 결제/취소일)|

|확장함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|transform|-|global|ALBaseResponse를 ALUsingListModel로 변환|rstCode 200을 성공으로 처리|

##### 역할
- 사용자의 항공편 이용 이력 관리
- 탑승 완료된 여정의 기록 보관
- 이용 통계 및 마일리지 적립 참고 자료
- 재예약 시 이전 이용 정보 참조
- 고객 서비스 및 문의 시 이용 증빙 자료

## al.data.repository

### 역할
데이터 소스와 로컬 DB를 연결하여 데이터 접근을 추상화하고 비즈니스 로직을 담당하는 Repository 패턴 구현

### Code

#### data.repository.baseinfo.ALBaseInfoRepository & ALBaseInfoRepositoryImpl
- 앱에서 사용하는 기초 코드 정보와 공항 정보를 관리하는 Repository
- 서버에서 기초 데이터를 조회하고 로컬 DB에 캐싱하여 오프라인 지원 및 성능 최적화

##### 변수 및 함수 (Interface)
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|getBaseCodeInfo|-|public|기초 코드 정보 조회|Single<ALBaseCodeInfoModel> 반환|
|deleteAll|-|public|모든 로컬 데이터 삭제|캐시 초기화|

##### 변수 및 함수 (Implementation)
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|dataSource|private|기초 정보 데이터 소스|ALBaseInfoDataSource|
|baseCodeDao|private|기초 코드 DAO|ALBaseCodeDao|
|airportDao|private|공항 정보 DAO|ALAirportDao|

|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|getBaseCodeInfo|-|override|기초 코드 정보 조회 및 캐싱|서버 응답 상태에 따라 캐싱/로컬 데이터 반환|
|deleteAll|-|override|로컬 DB 전체 삭제|baseCodeDao, airportDao 모두 삭제|

##### 역할
- 서버 기초 데이터 동기화 및 로컬 캐싱 관리
- lastDate 기반 증분 업데이트로 효율적 데이터 관리
- 네트워크 오류 시 로컬 캐시 데이터 제공
- 공통 코드 및 공항 정보의 오프라인 지원

#### data.repository.bookmark.ALBookmarkRepository & ALBookmarkRepositoryImpl
- 사용자의 즐겨찾는 노선 정보를 관리하는 Repository
- 로컬 DB와 서버 동기화를 통해 즐겨찾기 데이터를 일관성 있게 관리

##### 변수 및 함수 (Interface)
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|getBookmarkList|-|public|즐겨찾기 목록 조회|Single<List<ALBookmarkEntity>> 반환|
|insertOrUpdate|ALBookmarkEntity|public|즐겨찾기 추가/수정|서버 저장 후 로컬 업데이트|
|delete|ALBookmarkEntity|public|즐겨찾기 삭제|서버 삭제 후 로컬 제거|
|requestBookmarkSync|-|public|서버와 동기화|서버 데이터로 로컬 갱신|
|deleteAll|-|public|모든 즐겨찾기 삭제|로컬 데이터 초기화|

##### 변수 및 함수 (Implementation)
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|dataSource|private|즐겨찾기 데이터 소스|ALBookmarkDataSource|
|dao|private|즐겨찾기 DAO|ALBookmarkDao|

##### 역할
- 사용자 즐겨찾기 노선의 서버-로컬 동기화
- 즐겨찾기 추가/삭제 시 서버 API 호출 후 로컬 반영
- 네트워크 상태와 관계없이 로컬 즐겨찾기 조회 지원
- 서버 동기화를 통한 멀티 디바이스 즐겨찾기 공유

#### data.repository.common.ALCommonRepository & ALCommonRepositoryImpl
- 앱에서 공통으로 사용되는 정보들을 관리하는 Repository
- 프로모션, 배너, 카드, 약관과 같은 다양한 공통 데이터를 통합 관리

##### 변수 및 함수 (Interface)
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|getPromotionBanner|String|public|프로모션 배너 조회|위치 코드별 배너|
|getPromotionPopup|-|public|프로모션 팝업 조회|Single<ALPromotionPopupModel>|
|getRollingBanner|-|public|롤링 배너 조회|Single<ALRollingBannerModel>|
|getPaymentMethod|-|public|결제 수단 조회|Single<ALPaymentMethodModel>|
|getTermsList|-|public|약관 목록 조회|Single<ALTermsListModel>|
|getCardList|-|public|등록 카드 목록 조회|메인 카드 우선 정렬|
|getCardInfo|String|public|카드 BIN 정보 조회|카드번호 앞 6자리로 조회|

##### 변수 및 함수 (Implementation)
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|dataSource|private|공통 정보 데이터 소스|ALCommonDataSource|

##### 역할
- 다양한 공통 정보의 통합 관리
- 프로모션 및 광고 컨텐츠 제공
- 사용자 등록 카드 정보 관리 및 메인 카드 우선 처리
- 카드 BIN 기반 카드사 정보 조회
- 약관 및 정책 문서 정보 제공

#### data.repository.passengerinfo.ALPassengerInfoRepository & ALPassengerInfoRepositoryImpl
- 사용자가 자주 사용하는 탑승객 정보를 관리하는 Repository
- 예약 시 편의성을 위해 탑승객 정보를 로컬에 저장하고 관리

##### 변수 및 함수 (Interface)
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|getList|-|public|탑승객 목록 조회|Single<List<ALPassengerEntity>>|
|insert|ALPassengerEntity|public|탑승객 정보 추가|Completable 반환|
|deleteAll|-|public|모든 탑승객 정보 삭제|로컬 데이터 초기화|
|delete|ALPassengerEntity|public|특정 탑승객 정보 삭제|Completable 반환|

##### 변수 및 함수 (Implementation)
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|dao|private|탑승객 정보 DAO|ALPassengerDao|

##### 역할
- 자주 사용하는 탑승객 정보의 로컬 저장 및 관리
- 예약 시 탑승객 정보 자동 완성 기능 지원
- 개인정보 보호를 위한 로컬 전용 저장
- 탑승객 정보 추가/삭제를 통한 예약 편의성 향상


#### data.repository.recentroute.ALRecentRouteRepository & ALRecentRouteRepositoryImpl
- 사용자의 최근 검색한 노선 정보를 관리하는 Repository
- 편도/다구간 검색 기록을 로컬 DB에 저장하여 재검색 편의성 제공

##### 변수 및 함수 (Interface)
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|getList|-|public|최근 검색 목록 조회|Single<List<ALRecentRouteEntity>>|
|insert|출발지, 도착지 코드|public|편도 검색 기록 저장|Completable 반환|
|insert|첫번째 출발지/도착지, 두번째 출발지/도착지|public|다구간 검색 기록 저장|시간차를 두어 순서 보장|
|deleteAllAsync|-|public|비동기 전체 삭제|Completable 반환|
|deleteAll|-|public|동기 전체 삭제|즉시 실행|
|delete|ALRecentRouteEntity|public|특정 기록 삭제|Completable 반환|

##### 변수 및 함수 (Implementation)
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|dao|private|최근 검색 DAO|ALRecentRouteDao|

##### 역할
- 사용자 검색 편의성을 위한 최근 검색 기록 관리
- 편도/다구간 검색에 따른 차별화된 저장 로직
- 시간 기반 검색 기록 정렬 및 관리
- 개인정보 보호를 위한 로컬 전용 저장

#### data.repository.reservation.ALReservationRepository & ALReservationRepositoryImpl
- 항공권 예약 및 결제, 취소 요청을 처리하는 Repository
- 예약 생성부터 취소까지 전체 예약 생명주기를 관리

##### 변수 및 함수 (Interface)
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|requestReservation|ALReservationRequest|public|예약 요청 처리|Single<ALReservationModel>|
|requestPayment|ALPaymentRequest|public|결제 요청 처리|Single<ALPaymentModel>|
|requestReservationCancel|ALReservationCancelRequest|public|일반 예약 취소|Single<ALReservationCancelModel>|
|requestReservationCancelByReason|ALReservationCancelByReasonRequest|public|사유 기반 예약 취소|Single<ALReservationCancelByReasonModel>|

##### 변수 및 함수 (Implementation)
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|dataSource|private|예약 데이터 소스|ALReservationDataSource|

##### 역할
- 항공권 예약 프로세스의 핵심 비즈니스 로직 처리
- 예약 생성, 결제, 취소의 통합 관리
- 일반 취소와 사유 기반 취소의 구분 처리
- 서버 API 호출 및 응답 데이터 변환

#### data.repository.ticket.ALTicketRepository & ALTicketRepositoryImpl
- 사용자의 예약 내역 및 이용 기록을 조회하는 Repository
- 예약 목록, 상세 정보, 동기화, 이용 내역를 통합 관리

##### 변수 및 함수 (Interface)
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|getReservationList|-|public|예약 목록 조회|Single<ALReservationCompleteListModel>|
|getReservationDetailList|String|public|예약 상세 조회|여정일련번호로 조회|
|getReservationSyncList|-|public|예약 동기화 목록|서버에서 최신 상태 조회|
|getUsingList|ALSelectTicketHisListRequest|public|이용 내역 조회|페이징 처리 지원|

##### 변수 및 함수 (Implementation)
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|dataSource|private|티켓 데이터 소스|ALTicketDataSource|

##### 역할
- 사용자 예약 내역의 종합 관리
- 예약 상태별 목록 조회 및 상세 정보 제공
- 서버와의 예약 정보 동기화
- 과거 이용 내역 조회를 통한 고객 서비스 지원

#### data.repository.travel.ALTravelRepository & ALTravelRepositoryImpl
- 항공편 검색 및 운임 규칙 조회를 담당하는 Repository
- 여행 상품 검색부터 운임 규정 확인까지 예약 전 단계 관리

##### 변수 및 함수 (Interface)
|함수|매개변수|범위/종속|내용|특징|
|:-----:|:---:|:---:|:---:|:---:|
|getTravelList|ALTravelListRequest|public|항공편 검색|Single<ALTravelListModel>|
|getPreFareRule|ALPreFareRuleRequest|public|예약 전 운임 규칙 조회|Single<ALFareRuleModel>|
|getPostFareRule|ALPostFareRuleRequest|public|예약 후 운임 규칙 조회|Single<ALFareRuleModel>|
|getFareDiscountRule|ALFareDiscountRuleRequest|public|신분 할인 규칙 조회|Single<ALFareDiscountRuleModel>|

##### 변수 및 함수 (Implementation)
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|dataSource|private|여행 데이터 소스|ALTravelDataSource|

##### 역할
- 항공편 검색 조건에 따른 결과 제공
- 예약 전후 운임 규칙 및 약관 정보 조회
- 신분 할인 대상자를 위한 할인 규칙 확인
- 항공편 선택 및 예약 진행을 위한 핵심 정보 제공
- 운임 계산 및 할인 적용을 위한 규칙 데이터 관리

## al.ext
### 역할
- 각종 확장 함수 모음


### Code
#### ext.CalendarExt
##### 변수 및 함수
|함수|포맷|설명|
|:-----:|:---:|:---:|
|displayDataPattern()|yyyy.MM.dd(E)|날짜 + 요일|
|displayDataPattern2()|MM.dd(E)|월/일 + 요일|
|yyyyMMddPattern()|yyyyMMdd|-|
|yyyyMMddPattern2()|yyyy-MM-dd|-|
|yyyyMMddhhmmssPattern()|yyyyMMddHHmmss|-|
yyyyMMddPattern3()|yyyy.MM.dd(EE)|긴 요일 형태|

##### 역할
- Calendar 객체를 다양한 포맷의 날짜 문자열로 변환하는 확장 함수 모음
- 날짜 포맷은 Locale.KOREA로 구성



#### ext.ContextExt
##### 변수 및 함수

| 함수명 | 매개변수 | 대상 | 설명 |
|:--------:|:------:|:------:|:------:|
| Context.startBrowser | url | Context | URL을 기본 브라우저로 실행 |
| Context.clipboard | label, text | Context | 텍스트 클립보드 복사 후 Toast 노출 (OS 13 이하 - 13부터는 자동) |
| AppCompatActivity.hideKeyboard() | AppCompatActivity | 현재 포커스된 뷰의 키보드 숨김 |
| Context.hasApp | intent | Context | 인텐트 기반 앱 설치 여부 확인 |
| Context.startMarket | intent | Context | 앱 마켓으로 이동 (market://, https://) |
| Activity.startBannerLink | banner, loginCallBack? | Activity | 배너 이동 링크 처리 (로그인 여부/딥링크/WebView 분기) |

##### 상세 로직 설명
-  startBrowser(url: String?)
    - Intent.ACTION_VIEW를 사용하여 외부 브라우저 실행
    - 예외 발생 시 ConsoleLog로 로그 출력
- clipboard(label: String, text: String)
    - 클립보드에 텍스트 복사
    - Android 13 미만은 직접 Toast 노출 (TIAToastBuilder 사용)
    - Android 13 이상은 시스템 자동 노출
- hideKeyboard
    - 현재 포커스된 뷰의 WindowToken 기준으로 키보드 숨김
- hasApp(intent: Intent): Boolean
    - PackageManager를 사용하여 패키지 존재 여부 확인
    - Android 13 대응을 위한 조건 포함
- startMarket(intent: Intent)
    - PlayStore 앱이 없을 경우 웹 링크로 fallback 처리
- startBannerLink(banner: ResponseInqrAdvrPupStupInf.Data.AdvrList, loginCallBack: (() -> Unit)? = null)
    - authTknNedYn == "Y"일 경우 로그인 상태 확인 후 토큰 삽입
    - 이동 경로 타입 코드(mvmnPathTypCd)에 따라 분기
        - "I": 딥링크 (DeepLinkManager)
        - "W": 타이틀 있는 WebView (TxbusAdvWebViewActivity.startWithTitle)
        - "O": 외부 링크
        - "V": 전체 화면 WebView (startWithNoTitle)

##### 역할
- AppCompatActivity의 공통 동작을 확장 함수로 제공  
- 브라우저 열기, 클립보드 복사, 키보드 숨기기, 앱 설치 확인 및 마켓 이동, 배너 클릭 시 동작 처리 구현

#### ext.DateExt
##### 변수 및 함수
|함수|포맷|설명|
|:-----:|:---:|:---:|
|hhmmPattern()|HH:mm|-|
|yyyyMMddPattern()|yyyyMMdd|-|
|yyyyMMddhhmmssPattern()|yyyyMMdd|-|
|displayDatePattern()|yyyy.MM.dd(E)|-|
|displayDatePattern2()|yyyy-MM-dd|-|
|displayDatePattern3()|yyyy.MM.dd(E)|-|
|displayDatePattern4()|yyyy년 M월 d일|-|
|displayDatePattern5()|yyyy.MM.dd(E) HH:mm|-|
|hhmmPattern2()|HHmm|-|
|yyyyMMddhhmmssPattern2()|yyyy-MM-dd'T'HH:mm:ss|-|
|MMddDayPattern()|MM.dd(E)|-|
|MMddDayPattern2()|MM월 dd일 E요일|-|
|MMddDayPattern3()|MM월 dd일 E요일, a hh:mm~|-|
##### 역할
- Date 객체를 다양한 포맷의 날짜 및 시간 문자열로 변환하는 확장 함수 모음
- 날짜 포맷은 Locale.KOREA로 구성


#### ext.AirportExt
##### 역할
- List<Airport> 객체에 대한 공항 코드 기반 검색 및 이름 반환 확장 함수 제공
- 공항 코드로부터 공항명 또는 전체 객체를 빠르게 조회할 수 있도록 함

##### 변수 및 함수
| 함수명 | 제공 타입 | 반환 타입 | 설명 |
|--------|-------------|-----------|------|
| getName(airportCode) | List<ALBaseCodeInfoModel.Airport> | String | 주어진 공항 코드에 대응하는 airportArea(공항명) 반환 |
| getAirport(airportCode) | List<ALBaseCodeInfoModel.Airport> | Airport? | 주어진 공항 코드에 해당하는 Airport 객체 반환 |

##### 상세 로직 설명
- getName(airportCode: String): String
    - airportCode와 일치하는 항목을 찾아 해당 airportArea(공항명)를 반환
    - 찾지 못한 경우 빈 문자열 반환
    - UI 노출 시 fallback 처리 용이
- getAirport(airportCode: String): ALBaseCodeInfoModel.Airport?
    - airportCode와 일치하는 항목을 찾아 Airport 객체 자체를 반환
    - 객체 내 상세 속성(airportArea, airportCode)을 활용


#### ext.StringExt
##### 역할
- String 및 Int 타입에 유용한 확장 함수들을 정의
- 항공사, 공항, 국가와 같은 코드를 이름으로 매핑
- 이메일, 이름, 생년월일 유효성 검사
- 금액 형식화
- DecimalFormat을 사용한 원화 쉼표 처리 함수 포함

##### 변수 및 함수

| 함수 | 제공 타입 | 반환 타입 | 설명 |
|------|-------------|-----------|------|
| airlineLogo() | String | String | 항공사 코드로 로고 URL 생성 |
| airlineColor() | String | String | 항공사 코드 로 색상 코드 반환(attr6) |
| airportArea() | String | String | 공항코드로 공항 지역명 반환 |
| airportName() | String | String | 공항코드로 공항명 반환 |
| airport() | String | Airport? | 공항코드로 Airport 객체 반환 |
| airportWithName() | String | Airport? | 공항명으로 Airport 객체 반환 |
| nationalityName() | String | String | 국가코드로 국가명 반환 |
| seatName() | String | String | 좌석코드 → 좌석명 반환 |
| emailValidation() | String | Boolean | 이메일 형식 유효성 검사 |
| nameValidation() | String | Boolean | 한글 or 영문만 허용하는 이름 유효성 검사 |
| isOnlyUpper() | String | Boolean | 영대문자만 포함하는지 검사 |
| isOnlyKorean() | String | Boolean | 한글만 포함하는지 검사 |
| isContainLower() | String | Boolean | 소문자가 하나라도 포함되었는지 검사 |
| dateOfBirthValidation() | String | Boolean | yyyyMMdd 포맷 생년월일 유효성 검사 (실제 날짜 비교 포함) |
| dateOfBirthValidation2() | String | Boolean | yyyy-MM-dd 포맷 생년월일 유효성 검사 (정규식 기반) |
| wonFormat() | String? | String | 정수형 문자열을 원화 포맷으로 변환 |
| wonFormat() | Int | String | 정수를 원화 포맷 문자열로 변환 |

#### ext.ViewExt
##### 역할
- View, EditText, ViewPager2, RecyclerView과 같은 UI 컴포넌트에 대한 공통 확장 함수 모음
- 클릭 이벤트 처리, 포커스에 따른 UI 상태 변경, 페이지 애니메이션, 중앙 정렬과 같은 UI 로직 간소화


##### 함수 상세 설명
- click()
    - View 클릭 리스너를 간결하게 등록하는 확장 함수  
- clicks()
    - 여러 View에 공통 클릭 이벤트를 등록할 수 있는 유틸 함수  
- EditText.setStateBackGround(uppercase, validationBlock, onTextChange)
    - 텍스트 변경 및 포커스 변화에 따라 배경 리소스를 자동으로 변경
    - 유효성 검사 기반으로 상태를 파란색, 회색, 빨간색으로 표현
    - uppercase = true일 경우 자동으로 대문자로 변환
- ViewPager2.paymentPageTransformer(pageMarginPx, offsetPx)
    - ViewPager2에 마진 및 오프셋 기반 슬라이딩 애니메이션 적용
-  RecyclerView.setCenterToPosition(position)
    - 특정 아이템을 가운데 정렬되도록 스크롤 위치를 자동 계산
    - itemWidth = 92dp, padding = 16dp 기준으로 중앙 오프셋 계산

##### 배경 리소스 조건 요약
|상황|조건|배경 리소스|
|:---:|:---:|:---:|
|포커스 O, 유효/빈칸|O|파란 테두리|
|포커스 O, 유효 X|X|빨간 테두리|
|포커스 X, 유효/빈칸|O|회색 테두리|
|포커스 X, 유효 X|X|빨간 테두리|

## al.module.db.dao
### 요약

### Code
#### module.db.dao.ALAirportDao
##### 역할
- Room Database의 DAO 인터페이스
- ALAirportEntity(공항명, 공항 코드, 공항 지역)의 조회, 삽입, 삭제 기능 제공

##### DAO 메서드 설명
| 함수명 | 반환 타입 | 설명 | 특징 |
|:--------:|:----------:|:------:|:------:|
| getList() | List<ALAirportEntity> | 전체 공항 목록을 조회 | 단순 조회 쿼리 |
| insert(codeList: List<ALAirportEntity>) | - | 공항 목록 삽입 | 같은 키가 있으면 덮어쓰기 (REPLACE 전략) |
| deleteAll() | - | 모든 공항 데이터 삭제 | 테이블 초기화 용도 |

#### module.db.dao.ALBaseCodeDao
##### 역할
- Room Database의 DAO 인터페이스
- ALBaseCodeEntity(코드 그룹, 코드, 코드명, 정렬 순서, 부가 속성 1~9)의 조회, 삽ㅇ입, 삭제 기능 제공

##### DAO 메서드 설명
| 함수명 | 반환 타입 | 설명 | 특징 |
|:--------:|:----------:|:------:|:------:|
| getList() | List<ALBaseCodeEntity> | 기초 코드 목록 조회 기능을 제공 | 단순 조회 쿼리 |
| insert(codeList: List<ALBaseCodeEntity>) | - | 기초 코드 목록 삽입 | 같은 키가 있으면 덮어쓰기 (REPLACE 전략) |
| deleteAll() | - | 모든 기초 코드 데이터 삭제 | 테이블 초기화 용도 |

#### moudle.db.dao.ALBookmarkDao
##### 역할
- 사용자의 즐겨찾기 노선 정보(Bookmarks)를 로컬 DB에 저장/조회/삭제하는 DAO
- 고정핀 기능 (isFixedPin) 우선 정렬을 통해 자주 사용하는 노선을 상단에 배치
- 고정핀 여부, 생성 시간 순으로 정렬하여 최대 30개 조회

##### 변수 및 함수
| 함수 | 매개변수 | 반환 타입 | 설명 |
|:----:|:--------:|:----------:|:------|
| getList() | - | List<ALBookmarkEntity> | 고정핀 여부를 확인한 후 생성 시간 순 정렬로 최대 30개 즐겨찾기 목록 조회 |
| insertList(list) | List<ALBookmarkEntity> | - | 즐겨찾기 목록을 일괄 저장 또는 갱신 |
| insertOrUpdate(bookmark) | ALBookmarkEntity | - | 단일 즐겨찾기 저장 또는 갱신 (OnConflictStrategy.REPLACE) |
| deleteAll() | - | - | 모든 즐겨찾기 데이터 삭제 |
| delete(bookmark) | ALBookmarkEntity | - | 특정 즐겨찾기 삭제 |

#### module.db.dao.ALPassengerDao
##### 역할
- 사용자가 자주 입력하는 탑승객 정보(인덱스, 생년월일, 국적 코드, 성별, 성, 이름)를 로컬에 저장/조회/삭제하는 DAO
- 예약 시 반복 입력 최소화를 위한 자동완성 기능 기반
Single과 Completable을 활용하여 비동기 작업 처리
Room 기반 DAO이며 중복 삽입 시 REPLACE 정책으로 덮어쓰기

##### 변수 및 함수
| 함수 | 매개변수 | 반환 타입 | 설명 |
|:----:|:--------:|:----------:|:------|
| getList() | - | Single<List<ALPassengerEntity>> | 저장된 탑승객 정보 전체 조회 |
| insert(recentRoute) | ALPassengerEntity | Completable | 단일 탑승객 정보 삽입 또는 갱신 |
| deleteAll() | - | - | 모든 탑승객 정보 삭제 |
| delete(recentRoute) | ALPassengerEntity | Completable | 특정 탑승객 정보 삭제 |

#### module.db.dao.ALRecentRouteDao
- 사용자가 검색한 최근 항공 노선 정보를 로컬에 저장 및 관리하는 DAO
- 최대 5개까지 저장하며 가장 최근 검색이 상단에 위치


##### 주요 함수 및 역할
| 함수 | 매개변수 | 반환 타입 | 설명 |
|:----:|:--------:|:----------:|:------|
| getList() | 없음 | Single<List<ALRecentRouteEntity>> | 최근 검색한 노선 목록을 최신순으로 5개 조회 |
| insert(recentRoute) | ALRecentRouteEntity | Completable | 단일 최근 검색 노선 저장 |
| insert(recentRouteList) | List<ALRecentRouteEntity> | Completable | 다중 최근 검색 노선 저장 |
| deleteAllAsync() | - | Completable | 모든 최근 노선 정보 비동기 삭제 |
| deleteAll() | - | - | 모든 최근 노선 정보 동기 삭제 |
| delete(recentRoute) | ALRecentRouteEntity | Completable | 특정 검색 노선 삭제 |

##### 특징 및 역할 요약
- 최근 검색 이력 자동 저장 기능 제공 (편도/다구간)
- LIMIT 5로 최대 5개까지 최근 순서로 유지
- Room Dao 기반이며 REPLACE 정책 사용하고 동일한 검색은 덮어쓰기
- 동기/비동기 삭제 모두 제공하여 상황에 따른 유연한 처리 가능

## al.module.db.entities
### 요약
### Codes
#### module.db.entities.ALAirportEntity
##### 역할
- 항공 시스템 내 공항 코드 및 관련 정보를 저장하는 로컬 Entity
- Room DB에 저장되며 기본 코드 데이터로 활용

##### Entity 정보

| 필드명 | 타입 | DB 컬럼명 | 설명 |
|:------:|:----:|:----------:|:-----|
| airportName | String | airportName | 공항의 이름 |
| airportCode | String | airportCode | 공항 코드 - @PrimaryKey |
| airportArea | String | airportArea | 공항이 속한 지역 구분 |

#### module.db.entities.ALBaseCodeEntity
##### 역할
- 앱 전반에서 사용하는 기초 코드 정보를 저장하는 로컬 Entity
- Room DB에 저장되며 코드 그룹 및 코드명으로 식별되는 전반적인 코드 데이터

##### Entity 정보

| 필드명 | 타입 | DB 컬럼명 | 설명 |
|:------:|:----:|:----------:|:-----|
| codeGroup | String | codeGroup | 코드 그룹 - @PrimaryKey |
| code | String | code | 코드 값 - @PrimaryKey |
| codeName | String | codeName | 코드명 - @PrimaryKey |
| sort | Int | sort | 정렬 순서 |
| isUse | Boolean | isUse | 사용 여부 |
| attr1 ~ attr9 | String | attr1 ~ attr9 | 부가 속성값 |

#### module.db.entities.ALBookmarkEntity
##### 역할
- 항공 노선 즐겨찾기(Bookmark) 정보를 저장하는 로컬 Entity
- Room DB에 저장되며 자주 이용하는 노선을 간편하게 재검색할 수 있도록 지원

##### Entity 정보
| 필드명 | 타입 | DB 컬럼명 | 설명 |
|:------:|:----:|:----------:|:-----|
| departureAirportName | String? | departureAirportName | 출발지 공항명 |
| departureAirport | String | departureAirport | 출발지 공항 코드 - @PrimaryKey |
| arrivalAirportName | String? | arrivalAirportName | 도착지 공항명 |
| arrivalAirport | String | arrivalAirport | 도착지 공항 코드 - @PrimaryKey |
| isFixedPin | Boolean | isFixedPin | 상단 고정 여부 |
| nickName | String | nickName | 사용자가 지정한 별칭 |
| createTime | Long | createTime | 생성 시각 |

#### module.db.entities.ALPassengerEntity
##### 역할
- 사용자가 입력한탑승객 정보를 로컬 DB에 저장하기 위한 로컬 Entity
- Room DB에 저장되며 항공권 예약 시 반복 입력을 줄이기 위해 활용
- @Parcelize를 통해 Android 컴포넌트 간 전달 가능
- 별도 식별자 id를 자동 생성 (@PrimaryKey(autoGenerate = true))

##### Entity 정보
| 필드명 | 타입 | DB 컬럼명 | 설명 |
|--------|------|-----------|------|
| id | Int | id | 자동 증가되는 식별자 - @PrimaryKey |
| dateOfBirth | String | dateOfBirth | 탑승객 생년월일 |
| nationalityCode | String | nationalityCode | 탑승객 국적 코드 |
| gender | String | gender | 탑승객 성별 |
| firstName | String | firstName | 탑승객 성 |
| lastName | String | lastName | 탑승객 이름 |

#### module.db.entities.ALRecentRouteEntity
##### 요악
- 사용자의 최근 검색 노선을 저장하기 위한 로컬 Entity
- Room DB에 저장되며 편도 또는 다구간 항공편 검색 시 재검색 편의성 제공

##### Entity 정보
| 필드명 | 타입 | DB 컬럼명 | 설명 |
|--------|------|-----------|------|
| departureAirportCode | String | departureAirportCode | 출발지 공항 코드 - @PrimaryKey |
| arrivalAirportCode | String | arrivalAirportCode | 도착지 공항 코드 - @PrimaryKey |
| createTime | Long | createTime | 생성 시간 |

## al.module.db
### Codes
#### module.db.ALDatabase
##### 역할
- 로컬 Room Database 클래스
- 항공 관련 기초정보, 즐겨찾기, 탑승객 정보, 최근 검색을 저장
- Room.databaseBuilder()로 생성
- TiaApp Application Context 기반 싱글톤 인스턴스 구성
- @Synchronized 사용으로 멀티스레딩 환경에서도 안정적

##### 주요 기능
| 메서드 | 반환 타입 | 설명 |
|--------|------------|------|
| baseCodeDao() | ALBaseCodeDao | 코드 정보 DAO 제공 |
| recentRoutDao() | ALRecentRouteDao | 최근 검색 노선 DAO 제공 |
| airportDao() | ALAirportDao | 공항 정보 DAO 제공 |
| passengerDao() | ALPassengerDao | 탑승객 정보 DAO 제공 |
| bookmarkDao() | ALBookmarkDao | 즐겨찾기 DAO 제공 |
| getInstance() | ALDatabase | 싱글톤 인스턴스 반환 |


##### 구성된 Entity 목록
| Entity 클래스명 | 설명 |
|-----------------|------|
| ALAirportEntity | 공항 정보 |
| ALBaseCodeEntity | 기초 코드 정보 |
| ALBookmarkEntity | 즐겨찾기 노선 정보 |
| ALPassengerEntity | 최근 입력한 탑승객 정보 |
| ALRecentRouteEntity | 최근 검색 노선 기록 |

##### Room 설정
| 항목 | 설정값 |
|------|--------|
| version | 2 |
| exportSchema | false |
| allowMainThreadQueries() | Main Thread에서도 DB 접근 허용 |

##### 마이그레이션(version 1 → 2)
- ALBaseCodeEntity에 부가속성 COLUMN(attr5~attr9) 추가
- SQL 적용 코드
    ```sql
    ALTER TABLE 'ALBaseCodeEntity' ADD COLUMN 'attr5' TEXT DEFAULT '' NOT NULL;
    ALTER TABLE 'ALBaseCodeEntity' ADD COLUMN 'attr6' TEXT DEFAULT '' NOT NULL;
    ALTER TABLE 'ALBaseCodeEntity' ADD COLUMN 'attr7' TEXT DEFAULT '' NOT NULL;
    ALTER TABLE 'ALBaseCodeEntity' ADD COLUMN 'attr8' TEXT DEFAULT '' NOT NULL;
    ALTER TABLE 'ALBaseCodeEntity' ADD COLUMN 'attr9' TEXT DEFAULT '' NOT NULL;
    ```

## al.module.network.service
### Codes
#### module.network.service.ALCommonService
##### 역할
- 공통 정보(광고, 배너, 카드, 결제 수단, 약관)를 조회하는 Retrofit 인터페이스
- 모든 API는 POST 방식, RxJava의 Single<Response<...>> 반환
- 공통 응답 구조인 ALBaseResponse<T>를 통해 일관된 데이터 수신

##### 메서드 목록

| 메서드명 | API 엔드포인트 | Request 타입 | Response 데이터 타입 | 설명 |
|:--|:--|:--|:--|:--|
| inqrAdvertStupInfo | api/mui/v2/inqrAdvrPupStupInf | ALPromotionBannerRequest | ALPromotionBannerResponse | 광고 배너 설정 정보 조회 |
| inqrAdvrPupStupInf | api/mui/v2/inqrAdvrPupStupInf | ALPromotionPopupRequest | ALPromotionPopupResponse | 광고 팝업 설정 정보 조회 |
| inqrMoappBnrList | api/mui/v2/inqrAdvrPupStupInf | ALRollingBannerRequest | ALRollingBannerResponse | 롤링 배너 정보 조회 |
| inqrPymMnsListAtPym | api/pym/v2/inqrPymMnsListAtPym | ALPaymentMethodRequest | ALPaymentMethodResponse | 결제 시 사용 가능한 결제 수단 목록 조회 |
| inqrPrpmAutPymStplAgrmYN | api/mbrs/v2/inqrStplList | ALTermsListRequest | ALTermsListResponse | 이용약관과 같은 약관 목록 조회 |
| inqrCardBinInfo | api/pym/v2/inqrCardBinInfo | ALCardInfoRequest | ALCardInfoResponse | 카드 BIN 정보 조회 |

#### 특징
- 동일한 API 경로를 사용하되 Request 객체의 타입으로 기능을 구분
- 서버 통신 실패/성공 여부를 포함한 ALBaseResponse<T> 래핑 구조
- 모든 응답은 Retrofit2의 Response<> 래핑 + RxJava의 Single로 감싸져 있음
- ViewModel 또는 Repository 단에서 처리 용이

#### module.network.service.ALService
##### 역할
- 항공 여정, 예약, 결제, 취소, 즐겨찾기와 AL(항공) 관련 주요 기능을 담당하는 Retrofit API 인터페이스
- 모든 메서드는 RxJava의 Single<Response<...>>를 통해 비동기 호출 처리

##### 메서드 목록
| 메서드명 | API 엔드포인트 | Request 타입 | Response 데이터 타입 | 설명 |
|:--|:--|:--|:--|:--|
| selectBasicCodeInfo | api/u/cmm/v1/selectBaseInfo | ALBaseCodeInfoRequest | ALBaseCodeInfoResponse | 기초 정보 코드 제공 |
| selectSchList | api/u/sch/v1/selectSchList | ALTravelListRequest | ALTravelListResponse | 여정 정보 제공 |
| selectPreFareRule | api/u/pnr/v1/selectPreFareRule | ALPreFareRuleRequest | ALFareRuleResponse | 예약 전 운임 규정 |
| selectFareDcRule | api/u/pnr/v1/selectFareDcRule | ALFareDiscountRuleRequest | ALFareDiscountRuleResponse | 신분 할인 목록 |
| requestPnrReservation | api/u/pnr/v1/requestPnrReservation | ALReservationRequest | ALRservationResponse | 항공권 예약 |
| requestPayment | api/u/pnr/v1/requestPayment | ALPaymentRequest | ALPaymentResponse | 항공권 결제 |
| selectPnrList | api/u/pnr/v1/selectPnrList | Any = Any() | ALRservationCompleteListResponse | 예매 완료 조회 |
| selectPnrInfo | api/u/pnr/v1/selectPnrInfo | ALReservationCompleteDetailRequest | ALReservationCompleteDetailResponse | 예매 상세 조회 |
| selectPostFareRule | api/u/pnr/v1/selectPostFareRule | ALPostFareRuleRequest | ALFareRuleResponse | 예약 후 운임 규정 |
| requestPnrRefund | api/u/pnr/v1/requestPnrRefund | ALReservationCancelRequest | ALReservationCancelResponse | 항공권 예매 취소 |
| requestSpcPnrRefund | api/u/pnr/v1/requestSpcPnrRefund | ALResrvationCancelByReasonRequest | ALReservationCancelByReasonResponse | 항공권 예매 취소 요청 |
| requestCurrentPnr | api/u/pnr/v1/requestCurrentPnr | Any = Any() | ALReservationCompleteListResponse | 항공권 초기화 |
| selectFavorRouteList | api/u/fvrt/v1/selectFavorRouteList | Any = Any() | ALBookmarkListResponse | 즐겨기 목록 조회 |
| saveFavorRoute | api/u/fvrt/v1/saveFavorRoute | ALBookmarkSaveRequest | ALBookmarkSaveResponse | 즐겨찾기 저장 |
| deleteFavorRoute | api/u/fvrt/v1/deleteFavorRoute | ALBookmarkDeleteRequest | ALBookmarkDeleteResponse | 즐겨찾기 삭제 |
| selectTicketHisList | api/u/cent/v1/selectTicketHisList | ALSelectTicketHisListRequest | ALUsingListResponse | 즐겨찾기 삭제 |

## al.module.network
### Codes
#### module.network.ALApiClient
##### 역할
- AL 모듈의 Retrofit 클라이언트를 구성하는 객체
- 항공 예매 관련 API 호출을 위한 HTTP 클라이언트 설정 및 인증 처리
- AccessToken 기반 인증 처리
- Chucker 및 LoggingInterceptor로 디버깅 지원
- UUID, OS, App Version의 API 요청 공통 헤더 삽입

##### 주요 함수 및 클래스

| 함수/클래스 | 반환타입 | 설명 |
|:---:|:---:|:---|
| getRetrofit(baseUrl: String) | Retrofit | 주어진 baseUrl에 맞는 Retrofit 인스턴스 생성 |
| client() | OkHttpClient | 인증, 타임아웃, 로깅, 인터셉터가 포함된 클라이언트 생성 |
| AppInterceptor | Interceptor | 공통 헤더 추가 및 토큰 처리용 커스텀 인터셉터 클래스 |

##### 공통 헤더 구성

| 헤더명 | 설명 |
|:---:|:---|
| Content-type | application/json;charset=UTF-8 |
| UUID | App 내 저장된 UUID |
| appCode | 고정 값 MIC |
| Connection | close |
| osKnd | A - Android |
| osVer | Android OS Version |
| appVer | 앱 버전 |
| Authorization | Bearer AccessToken - 로그인 시 |



## al.provider
### Codes
#### provider.ALProvider
##### 역할
- AppPreference(sharedPreferences)를 기반으로 항공권 예매 관련 설정값을 저장하고 조회하는 싱글톤 객체
- 주로 마지막 기초데이터 업데이트 시각 및 마지막 사용 결제수단 정보를 관리함
- 앱 실행 시 이전에 저장된 기초데이터 업데이트 시간을 확인하여 업데이트 여부 결정
- 예매 프로세스에서 마지막으로 선택한 결제 수단을 기본으로 보여주기 위한 설정값 제공

##### 변수 및 프로퍼티

| 프로퍼티명 | 타입 | 설명 | 기본값 또는 조건 |
|------------|------|------|------------------|
| baseDataLastData | String | 기초데이터 마지막 업데이트 일자 (형식: yyyyMMddHHmmss) | 기본값: ALConstants.DEFAULT_BASE_DATA_LAST_DATE |
| lastPaymentMethod | Int | 마지막으로 사용한 결제수단 코드 | 기본값: 1 |

##### 내부 구현 방법
- Android SharedPreferences를 사용해 putString, putInt, getString, getInt 방식으로 저장 및 조회
- AppPreference.get().sharedPreferences를 통해 전역 Preference 인스턴스를 참조함

## al.ui.bridge
### Codes
#### ui.bridge.ALBridgeActivity
##### 역할
- 항공 기능 진입 전 NetFunnel을 통한 접속 제어 및 딥링크에 따른 화면 분기를 담당하는 브릿지 액티비티
- 항공 메인 혹은 예매 상세 화면으로 라우팅 역할을 수행하며, ViewModel을 통해 필요한 초기 데이터를 가져온다

##### 주요 함수 및 클래스
| 메서드명 | 설명 |
|----------|------|
| onCreate() | NetFunnel 시작 및 initObserve() 실행 |
| init() | 딥링크 타입에 따라 항공 메인 진입 또는 예매 상세 진입 처리 |
| initObserve() | ViewModel 이벤트 관찰 및 화면 전환 처리 |
| showNetFunnel() | NetFunnel 로직 실행 후 onSuccess() 시 init() 호출 |
| orientation() | Android 8.0 제외 모든 OS 버전에서 세로 고정 설정 |
| finish() | 액티비티 종료 시 애니메이션 제거 |

##### ViewModel 연동
- ALBridgeViewModel의 이벤트를 관찰하여 화면 전환
  - onMain → ALMainActivity로 이동
  - onReservationDetail → ALReservationDetailActivity로 이동

##### 라우팅 함수
| 메서드 | 설명 |
|--------|------|
| startMain() | 항공 메인 화면으로 이동 |
| startMainWithAirportCode() | 출발지와 도착지 포함 항공 메인 이동 |
| startReservationDetail() | 특정 여정의 예매 상세 화면 이동 |
| startActivityForResultWithReservationDetail() | 예매 상세 화면으로 이동하며 결과 콜백 필요 시 사용 |



#### ui.bridge.ALBridgeViewModel
##### 역할
- 항공 메인 또는 예매 상세 진입을 위한 데이터 초기화와 라우팅 이벤트를 담당하는 ViewModel
- BaseInfo 및 ReservationDetail 정보를 네트워크를 통해 불러오고 전역 데이터로 저장

##### 주요 함수 및 클래스
| 메서드 | 설명 |
|--------|------|
| getBaseInfo() | 기초 코드 정보 호출 후 _onMain 이벤트 트리거 |
| getBaseInfoAndReservationDetail() | 기초 코드 + 예매 상세 정보 병렬 호출 후 _onReservationDetail 이벤트 트리거 |
| setBaseInfo() | 기초 코드 데이터를 필터링하여 ALGlobalData에 세팅 |
| addFilteredCodesToGlobalList() | 조건에 따라 특정 코드 그룹만 필터링해서 대상 리스트에 저장 |
| Factory | ViewModelProvider.Factory 구현으로 DI - Dependency Injection 대신 객체 생성 위임 |

##### LiveData
| 변수명 | 타입 | 설명 |
|--------|------|------|
| _onMain | MutableLiveData<Event<Any>> | 항공 메인 진입 신호 |
| onMain | LiveData<Event<Any>> | 외부 접근용 (항공 메인 화면 트리거) |
| _onReservationDetail | MutableLiveData<Event<ALReservationCompleteDetailModel>> | 예매 상세 진입 신호 |
| onReservationDetail | LiveData<Event<ALReservationCompleteDetailModel>> | 외부 접근용 (예매 상세 화면 트리거) |

##### 데이터 흐름 요약
1. ALBridgeActivity에서 딥링크 타입을 기반으로 getBaseInfo() 또는 getBaseInfoAndReservationDetail() 호출
2. BaseCode 및 ReservationDetail 데이터를 받아서 ALGlobalData에 저장
3. 이벤트 LiveData로 알려서 Activity에서 화면 전환 처리

##### 예외 및 상태 처리
- it.isDuplicateLogin(): 중복 로그인 시 상태 변경
- it.isSuccess().not(): 네트워크 오류 또는 실패 메시지 출력
- _uiState.value = Event(ALUiState.Alert(...)): 전역 상태 알림용

## al.ui.common
### Codes

#### ui.common.dialog.ALCommonDialog
##### 역할
- 공통 다이얼로그 UI를 제공하는 커스텀 Dialog 클래스
- 타이틀, 메시지, 확인과 취소 버튼을 유연하게 구성하고 클릭 이벤트를 처리함
- 다양한 상황에서 재사용 가능한 모듈형 다이얼로그를 제공함

##### 주요 함수 및 클래스
| 메서드 | 설명 |
|--------|------|
| onCreateDialog() | 다이얼로그의 타이틀 제거 및 외부 터치 시 닫힘 여부 설정 |
| onCreateView() | ViewBinding을 사용하여 레이아웃을 inflate |
| onViewCreated() | 레이아웃 크기 설정 및 initView() 호출 |
| initView() | 전달된 title, message, 버튼 텍스트 및 클릭 리스너를 바인딩 |
| title() | 다이얼로그 타이틀 설정 (String 또는 리소스 ID 사용 가능) |
| message() | 다이얼로그 메시지 설정 (String 또는 리소스 ID 사용 가능) |
| positive() | 확인 버튼 텍스트와 클릭 동작 설정 |
| negative() | 취소 버튼 텍스트와 클릭 동작 설정 |
| onDestroyView() | ViewBinding 해제하여 메모리 누수 방지 |

##### 주요 변수
| 변수명 | 타입 | 설명 |
|--------|------|------|
| _binding | AlDialogCommonBinding? | ViewBinding을 위한 내부 변수 |
| title, message | String? | 다이얼로그에 표시할 타이틀과 메시지 |
| positiveButton, negativeButton | String? | 버튼에 표시할 텍스트 |
| positiveClick, negativeClick | (() -> Unit)? | 클릭 시 실행할 람다 함수 |

##### 동작 흐름
1. 다이얼로그가 생성되면 title, message, 버튼 텍스트를 View에 반영
2. 버튼 클릭 시 등록된 람다(positiveClick, negativeClick) 실행 후 다이얼로그 닫힘
3. View가 파괴되면 _binding을 null로 설정하여 메모리 정리

#### ui.common.dialog.ALProgressDialog
##### 역할
- 사용자 인터랙션을 차단하고 로딩 상태를 보여주는 공통 진행 다이얼로그
- ProgressBar를 통해 작업 중임을 시각적으로 표시
- isCancelable = false 설정으로 취소 불가 다이얼로그

###### 주요 함수 및 동작
| 메서드 | 설명 |
|--------|------|
| onCreateView() | 레이아웃 바인딩 후 다이얼로그를 생성하고 외부 클릭 취소 비활성화 처리 |
| onViewCreated() | 애니메이션 제거 스타일 적용 및 초기화 함수 호출 |
| init() | 뒷배경 어둡게 처리 제거 + 뷰 바인딩 + 프로그레스 시작 |
| dismiss() | 프로그레스바 종료 후 다이얼로그 닫기 - 예외 처리 포함 |
| onDestroyView() | ViewBinding 해제 - 메모리 누수 방지 |

##### 주요 변수

| 변수명 | 타입 | 설명 |
|--------|------|------|
| _binding | DialogAlProgressBinding? | ViewBinding 객체의 내부 참조 |
| binding | DialogAlProgressBinding | 널이 아닌 뷰 바인딩 접근자 |

##### 동작 흐름 요약

1. 다이얼로그가 생성되면 isCancelable = false로 설정되어 사용자가 닫을 수 없음
2. onViewCreated()에서 다이얼로그 애니메이션 제거 및 init() 호출
3. init()에서는 배경 흐림 제거 및 ProgressBar 표시
4. 외부에서 dismiss() 호출 시 로딩 종료 후 안전하게 닫힘 처리
5. View가 파괴되면 _binding을 null로 설정하여 메모리 해제

##### 특징

- dialog?.window?.setWindowAnimations(R.style.ALAlertNoAnimation): 다이얼로그 애니메이션 비활성화

- dialog?.window?.clearFlags(WindowManager.LayoutParams.FLAG_DIM_BEHIND): 배경 어두워지는 효과 제거하여 배경과 함께 로딩 표시 가능

- binding.progress.show() / binding.progress.hide(): 커스텀 로딩 UI 표시와 제거

#### ui.common.view.ALActionBar
##### 역할
- 항공 서비스 전용 커스텀 액션바 컴포넌트
- 좌측 뒤로가기 버튼, 중앙 타이틀, 우측 텍스트 버튼으로 구성
- XML 속성(alActionBarTitle, alLeftSrc, alRightText)을 통해 동적 커스터마이징 가능

##### 주요 함수 및 속성
| 함수/변수 | 설명 |
|-----------|------|
| initAttributes(attrs) | XML에서 설정한 커스텀 속성들을 초기화하고 뷰에 반영 |
| initListener() | 뒤로가기 버튼과 우측 버튼의 클릭 리스너 설정 |
| setTitle(title: String) | 타이틀 텍스트를 코드로 동적으로 변경 |
| onBackPress | 뒤로가기 버튼 클릭 시 실행할 콜백 |
| onRightPress | 우측 텍스트 버튼 클릭 시 실행할 콜백 |

##### XML 커스텀 속성 (res/values/attrs.xml 기준)
| 속성명 | 타입 | 설명 |
|--------|------|------|
| alActionBarTitle | string | 타이틀 텍스트 |
| alLeftSrc | reference | 좌측 아이콘(drawable) |
| alLeftString | string | 좌측 아이콘의 접근성 설명 |
| alRightText | string | 우측 텍스트 버튼 내용 |
| alIsMain | boolean | 메인화면 여부 - 배경색 투명 처리 |

##### 뷰 구조
- tvTitle: 중앙 타이틀 텍스트
- ivBack: 좌측 뒤로가기 아이콘 버튼
- tvRight: 우측 텍스트 버튼
- flRoot: 루트 레이아웃 (메인 화면일 경우 투명 배경)

##### 동작 흐름
1. 생성자에서 initAttributes()로 XML 속성 초기화
2. tvTitle, ivBack, tvRight와 같은 구성요소에 값 반영
3. initListener()로 버튼 클릭 이벤트 연결
4. 외부에서 onBackPress, onRightPress 람다 할당 가능
5. 필요시 setTitle()로 타이틀 동적 변경 가능



#### ui.common.view.ALRecentRouteView
##### 역할
- 최근 검색한 항공 출발지와 도착지을 표시하는 커스텀 뷰
- 삭제 버튼 포함, setData()를 통해 출발지와 도착지 텍스트 설정 가능
- 항공 메인 또는 검색 화면에서 최근 검색 경로 리스트 UI 구성 시 사용

##### 주요 함수 및 속성
| 함수/변수 | 설명 |
|-----------|------|
| setData(departure: String, arrival: String) | 출발지와 도착지를 텍스트 뷰에 설정하고 접근성 설명 추가 |
| onCloseClick: () -> Unit | 닫기 버튼 클릭 시 실행될 콜백 - 람다 |
| init() | 닫기 버튼(ivClose)에 클릭 리스너 등록 |

##### 뷰 구성 요소
| ID | 역할 |
|----|------|
| tvDeparture | 출발지 텍스트 표시 |
| tvArrival | 도착지 텍스트 표시 |
| ivClose | 닫기 버튼 |

##### 동작 흐름 요약
1. 생성자에서 View Binding을 통해 XML 레이아웃 inflate
2. init() 내에서 닫기 버튼 클릭 리스너 연결
3. setData() 호출 시 출발지와 도착지 텍스트 설정 및 contentDescription 지정
4. 사용자가 닫기 버튼을 누르면 외부에서 지정한 onCloseClick() 콜백 실행

##### 접근성 처리
- 닫기 버튼의 contentDescription은 출발지+도착지 삭제 형식의 문자열을 세팅
- al_param_delete는 접근성 문자열 리소스로, 화면리더 사용자를 위한 삭제 안내 역할


#### ui.common.BindingAdapter
##### 역할
- 레이아웃 XML에서 직접 사용할 수 있는 데이터 바인딩 어댑터 정의
- 이미지 URL 로드 및 공항 코드와 지역 정보를 하나의 문자열로 표시하는 기능 제공

##### 주요 함수 및 설명
| 함수명 | 설명 |
|--------|------|
| alLoadImage | ImageView에 Glide를 이용하여 이미지 로드 - 디스크 캐시 포함 |
| alAirportName | 공항 지역명과 코드명을 결합하여 TextView에 출력 |

## al.ui.easypay
### Codes
#### ui.easypay.ALEasyPayActivity
##### 역할
- WebView를 통한 간편결제 페이지 표시 및 결제 결과 수신 처리
- 결제 성공 시 Intent를 통해 결과 반환
- result:// URL 스킴을 통해 결제 결과 수신 후 Intent로 결과 전달
- 외부 앱 링크도 처리하며 미설치 시 마켓으로 유도
- WebView에 JavaScript 설정 및 Alert 대체 다이얼로그 사용
- POST 방식으로 인코딩된 파라미터로 결제 URL 요청

##### 주요 함수 및 설명
| 함수 | 설명 |
|----------|------|
| init() | 백버튼 누르면 WebView 뒤로가기 또는 액티비티 종료 |
| initView() | WebView 설정 및 결제 페이지 로딩, URL 스킴 처리 |
| shouldOverrideUrlLoading() | 결제 결과 처리 또는 외부 앱 실행, 마켓 연결과 같은 URL 스킴 제어 |
| onJsAlert() | JS alert 호출 시 AlertDialogFragment로 대체 |
| postUrl() | 결제 요청 파라미터 생성 후 서버에 전송 |

## al.ui.farerules
### Codes
#### ui.farerules.adapter.ALFareRulesAdapter
##### 역할
- ALFareRuleModel.FareRule 데이터를 기반으로 RecyclerView에서 항공 운임 규칙 정보를 표시하는 Adapter
- 제목과 설명을 표시하며 설명은 HTML 형태로 파싱되어 표시됨

##### 구성 요약
| 구성 요소 | 설명 |
|----------|------|
| rollingBanner | 운임 규칙 데이터 리스트 |
| FareRulesHolder | 뷰 홀더 클래스, 바인딩 객체를 통해 데이터 표시 |
| Html.fromHtml() | 설명 텍스트가 HTML 형식일 수 있으므로 이를 텍스트로 변환 |

##### 데이터 흐름
- 외부에서 ALFareRuleModel.FareRule 리스트를 받아와 Adapter에 전달
- onCreateViewHolder()에서 뷰 바인딩 객체 생성
- onBindViewHolder()에서 해당 포지션의 아이템을 bind()에 전달
- bind() 함수에서 텍스트 뷰에 운임 규칙 정보 설정 - HTML 변환 포함

#### ui.farerules.ALFareRulesActivity
##### 역할
- 항공 운임 규칙 리스트를 보여주는 단순 화면
- ALFareRulesAdapter를 통해 전달받은 운임 규칙 데이터를 RecyclerView에 출력

##### 주요 구성 및 설명
| 구성 요소 | 설명 |
|-----------|------|
| ALFareRulesAdapter | RecyclerView에 운임 규칙 아이템을 표시 |
| actionBar.onBackPress | 뒤로 가기 버튼 눌렀을 때 finish() 처리 |
| intent.getParcelableArrayListExtra() | 인텐트를 통해 운임 규칙 리스트를 받아옴 |
| viewModel() | 별도의 ViewModel 없음 (null 반환) |

##### 버전 분기 처리
- Android 33+ (TIRAMISU)부터 getParcelableArrayListExtra()에 타입 명시 필요
- 그 이하 버전에서는 명시 없이 사용 가능

##### 데이터 흐름 요약
- onCreate() → initView() 호출
- 인텐트에서 운임 규칙 리스트 추출
- ALFareRulesAdapter에 전달해 RecyclerView에 바인딩
- 사용자 뒤로가기 버튼 누르면 finish() 호출로 액티비티 종료

## al.ui.main
### Codes
#### ui.main.adapter.ALAirportAdapter
##### 역할
- ALBaseCodeInfoModel.Airport 리스트를 기반으로 공항 선택 화면을 구성하는 RecyclerView Adapter
- 선택된 공항은 강조 표시되고 선택 시 외부 콜백(onAirportSelect)을 통해 알림

##### 주요 구성 및 설명
| 구성 요소 | 설명 |
|-----------|------|
| airportList | 공항 데이터 리스트 |
| currentAirportCode | 현재 선택된 공항 코드 (선택 여부 표시 용도) |
| onAirportSelect | 사용자가 공항을 선택했을 때 호출되는 콜백 |
| AirportHolder | ViewHolder 클래스, 각 공항 정보를 레이아웃에 바인딩 |

##### 주요 함수
| 함수 | 설명 |
|------|------|
| onCreateViewHolder() | ViewHolder 및 바인딩 객체 생성 |
| getItemCount() | 공항 리스트 개수 반환 |
| onBindViewHolder() | 포지션에 해당하는 ViewHolder에 데이터 바인딩 |
| AirportHolder.bind() | 공항 이름과 공항 코드를 표시하고 현재 선택 여부에 따라 스타일 변경 및 클릭 이벤트 처리 |

##### UI 처리 로직
- 공항명이 존재하면 지역 + 코드 형태로 텍스트 표시하고 없으면 지역명만 표시
- airport.airportCode == currentAirportCode일 경우
  - 텍스트 색상: 마스코트 컬러 - 자주색(al_48004a)
  - 선택 표시 뷰(vSelector): VISIBLE
  - 선택 상태: true
- 그 외
  - 텍스트 색상: 일반 회색(al_555555)
  - 선택 표시 뷰: GONE
  - 선택 상태: false

##### 접근성 처리
- tvAirportArea.contentDescription에 접근성용 공항명 제공 - 공항 코드 철자 분리

##### 데이터 흐름 요약
1. 외부에서 공항 리스트와 현재 선택된 공항 코드 전달
2. onCreateViewHolder → ViewHolder 생성
3. onBindViewHolder → ViewHolder의 bind() 실행
4. bind() 내에서 공항 정보 표시 및 선택 스타일 적용
5. 사용자 클릭 시 onAirportSelect 콜백 호출


#### ui.main.adapter.ALBookmarkAdapter
##### 역할
- ALBookmarkEntity 리스트를 기반으로 즐겨찾기 항공 노선을 RecyclerView로 표시
- 출발과 도착지와 닉네임, 고정 여부를 표현하며 항목 클릭 시 콜백으로 이벤트 전달

##### 주요 구성 및 설명
| 구성 요소 | 설명 |
|-----------|------|
| bookmarkList | 즐겨찾기 항공 노선 정보 리스트 |
| onItemClick | 항목 클릭 시 실행되는 콜백 함수 |
| BookmarkHolder | ViewHolder 클래스, 항목 UI와 데이터 바인딩 수행 |

##### 주요 함수
| 함수 | 설명 |
|------|------|
| onCreateViewHolder() | ViewHolder 및 바인딩 객체 생성 |
| onBindViewHolder() | 각 항목에 해당하는 데이터를 ViewHolder에 바인딩 |
| getItemCount() | 전체 즐겨찾기 항목 수 반환 |
| BookmarkHolder.bind() | 항목에 포함된 닉네임과 출도착지와 고정핀 표시 및 클릭 처리 |

##### UI 처리 로직
- nickName 존재 여부에 따라 표시 여부 제어
  - 있으면 tvNickname에 표시하고 VISIBLE
  - 없으면 GONE 처리
- tvDeparture, tvArrival에 출발지와 도착지 이름 표시
- ivPin은 isFixedPin == true일 때만 VISIBLE
- 전체 항목 클릭 시 onItemClick(item) 실행

##### 데이터 흐름 요약
1. 외부에서 bookmarkList와 onItemClick 전달
2. onCreateViewHolder() → ViewHolder 생성
3. onBindViewHolder() → bind() 호출하여 데이터 바인딩
4. View 내부에 노선 정보와 상태 UI 표시
5. 클릭 이벤트 시 onItemClick(item) 콜백 전달

#### ui.main.adapter.ALCalendarAdapter
##### 역할
- 항공권 예약을 위한 달력 선택 기능을 제공하는 RecyclerView Adapter
- 월별 헤더와 일자 아이템을 혼합하여 표현
- 편도 및 왕복에 따라 날짜 선택 및 UI 구성이 동적으로 달라짐
- AsyncListDiffer를 활용해 달력 리스트를 효율적으로 관리

##### 주요 구성 및 설명
| 구성 요소 | 설명 |
|-----------|------|
| journeyType | 편도, 왕복과 같은 여정 종류에 따라 UI 처리 방식 결정 |
| holidayList | 공휴일 데이터를 기반으로 날짜 색상 처리 |
| onDayClick | 날짜 선택 시 외부에 선택 결과 전달 |
| callback | 이전 달과 다음 달 이동 시 사용 |
| departureCalendar, arrivalCalendar | 선택된 출발일과 도착일을 저장 |
| ITEM_TYPE_HEADER, ITEM_TYPE_DAY | 아이템 구분 (헤더: 년과 월, 날짜) |

##### ViewHolder 구성
| 뷰홀더 | 역할 |
|--------|------|
| CalendarHolder | 날짜 셀 UI를 그리며 선택 상태, 공휴일, 왕복과 편도 표시 |
| HeaderHolder | 년과 월 표시 + 이전 달과 다음 달 이동 처리 |

##### 날짜 표시 처리
- today, oneYearLater 범위 내에서만 선택 가능
- 공휴일은 holidayList.pbhlDt 기준으로 색상 다르게 처리
- 요일에 따라 일요일, 토요일은 색상 강조
- 선택된 날짜(가는날/오는날)일 경우 배경색 및 타이틀 강조

##### 날짜 선택 처리 로직 (왕복)
1. 출발일만 선택된 경우 → 클릭한 날짜가 오는날
2. 출발과 도착 모두 선택된 경우 → 클릭한 날짜로 출발일 초기화, 도착일 초기화
3. 같은 날짜 선택 시 → 당일 왕복 처리

##### 날짜 선택 처리 로직 (편도)
- 클릭한 날짜를 출발일로 저장

##### 접근성 처리
- 날짜 셀에 접근성 설명 추가 (tvTitle.text + yyyyMMdd 형식)
- 선택 가능 여부도 접근성 노드에 설정

##### 이전 달과 다음 달 이동
- Header에서 leftThinIcon, rightThinIcon 클릭 시 callback(true/false, Calendar) 호출
- 오늘보다 이전 달 또는 1년 이후 달은 이동 버튼 비활성화

##### 데이터 흐름 요약
1. 외부에서 달력 데이터 리스트 전달 (updateList)
2. ViewType에 따라 Header/Day ViewHolder 생성
3. CalendarHolder.bind()에서 날짜 상태에 따라 UI 처리
4. 날짜 클릭 시 onDayClick(departure, arrival) 전달
5. 월 이동 시 callback(true/false, Calendar) 실행

#### ui.main.adapter.ALReservationCompleteAdapter
##### 역할
- 예약 완료 항공 여정 정보를 리스트 형태로 표시하는 RecyclerView Adapter
- 각 여정(Journey)에 대한 항공사, 출발과 도착 공항, 시간, 예약 상태의 정보를 카드 형태로 구성
- 사용자가 특정 예약을 선택할 수 있도록 클릭 이벤트를 제공

#### ui.main.bottomsheet.ALAirportSelectBottomSheet
##### 역할
- 공항 선택 기능을 제공하는 BottomSheetDialogFragment 클래스
- 최근 검색한 노선 목록과 전체 공항 리스트를 Grid 형태로 보여줌
- 사용자가 공항을 선택하거나 최근 노선을 클릭하면 ViewModel에 정보 전달 후 BottomSheet 종료
- ViewModel(ALMainViewModel)을 공유하여 선택된 공항 상태를 관리함

#### ui.main.bottomsheet.ALBookmarkBottomSheet
##### 역할
- 즐겨찾기(Bookmark) 공항 정보를 보여주는 BottomSheetDialogFragment 클래스
- ALMainViewModel의 bookmarkList를 기반으로 공항 목록을 표시하며, 항목 선택 시 출발지와 도착지를 설정 후 BottomSheet 종료
- 즐겨찾기 항목이 없을 경우 빈 화면 뷰를 보여줌
- 접근성 사용자를 위한 focus 제어 및 안내 문구 설정 포함

#### ui.main.bottomsheet.ALDateSelectBottomSheet
##### 역할
- 사용자가 항공 여정 날짜를 선택할 수 있는 BottomSheetDialogFragment
- 여정 타입(편도와 왕복)에 따라 달력 UI 구성 및 선택 로직을 제어함
- ALCalendarAdapter를 통해 커스텀 달력 목록을 렌더링하며 날짜 선택 시 출발일과 도착일을 ViewModel에 반영함
- 접근성 사용자를 위한 focus 제어 및 안내 문구 설정 포함

#### ui.main.bottomsheet.ALInfantChildInfoBottomSheet
##### 역할
- 유아 및 어린이 관련 안내 정보를 제공하는 BottomSheetDialogFragment
- 단순 안내 창 형태로 확인 버튼(btnConfirm) 클릭 시 BottomSheet가 닫힘

#### ui.main.bottomsheet.ALPassengerSelectBottomSheet
##### 역할
- 탑승객 수(성인/소아/유아) 선택을 위한 BottomSheetDialogFragment
- ALMainViewModel의 임시 승객 정보(initTempPassengerInfo)를 초기화
- 확인 버튼 클릭 시 setPassengerInfo()로 선택 내용 반영
- 취소 시 clearTempPassengerInfo() 호출로 변경 내용 폐기

#### ui.main.ALMainActivity
##### 역할
- 항공권 예매 메인 화면을 담당하는 액티비티
- 검색 조건(출발지와 도착지, 날짜, 인원) 설정 및 최근 예약 리스트, 광고 배너, 롤링 배너를 표시
- ViewModel과 바인딩되어 사용자의 입력과 이벤트를 관찰하고 처리
- 다양한 BottomSheet 호출 및 예약 결과 페이지로 이동하는 역할 수행

##### 주요 구성 요소
| 구성 요소 | 설명 |
|----------|------|
| ALMainViewModel | 메인 화면 관련 상태 및 데이터 관리 |
| ALReservationCompleteAdapter | 예약 완료 리스트를 ViewPager2로 표시 |
| 다양한 BottomSheet들 | 공항 선택, 날짜 선택, 탑승객 수 선택, 즐겨찾기 |
| Glide | 광고 이미지 및 배너 로딩 |
| AirbridgeManager | 광고 클릭 트래킹 처리 |
| TIAToastBuilder | 날짜 미선택 시 사용자 알림 |

##### 데이터 흐름 요약
- onCreate()에서 init(), deepLink(), initView(), initObserver() 호출
- Intent로 전달된 출/도착 공항 코드 처리 - deepLink
- 버튼 클릭 및 각 UI 이벤트에서 ViewModel 데이터 변경
- ViewModel의 LiveData를 observe 하여 UI 자동 갱신
- 최근 예약 리스트, 광고 배너, 롤링 배너, 프로모션 팝업과 같은 동적 UI 처리

##### ViewModel 연동 및 처리
- showBottomSheet: 사용자 액션에 따라 적절한 BottomSheet 호출
- rollingBanner: 롤링 배너 데이터 변경 시 ViewPager 갱신
- promotionPopup: 활성화된 프로모션이 있을 경우 팝업 노출
- reservationCompleteList: 최근 예약 정보 바인딩
- promotionBanner: 상단 광고 배너 이미지 및 설명 표시

#### ui.main.ALMainViewModel
##### 역할
- ALMainActivity에서 사용되는 메인 ViewModel로, 항공권 여정 정보, 프로모션, 예약 리스트, 최근 경로를 관리
- UI에서 발생하는 이벤트를 처리하고 관련 데이터를 구성하여 UI로 전달
- BottomSheet 호출 이벤트와 여정 정보 커밋, 데이터 초기화와 같은 다양한 로직 수행

##### 주요 LiveData 및 변수 요약

| 변수명 | 타입 | 설명 |
|--------|------|------|
| _journeyInfo, _journeyInfoMultiWay | MutableLiveData<ALAppJourneyInfo> | 여정 정보 (일반 경로와 다구간) |
| _promotionBanner, _promotionPopup, _rollingBanner | MutableLiveData<Model> | 각종 배너 및 팝업 관련 데이터 |
| _reservationCompleteList | MutableLiveData<ALReservationCompleteListModel> | 예약 완료 리스트 |
| _recentRouteList | MutableLiveData<List<ALRecentRouteEntity>> | 최근 검색 경로 리스트 |
| _showBottomSheet | MutableLiveData<Event<MainBottomSheet>> | 바텀시트 열기 트리거 |
| _passengerInfo, _tempPassengerInfo | MutableLiveData<ALAppPassengerInfo> | 여정 인원 정보 |
| bookmarkList, yearCalendar | ArrayList | 즐겨찾기 목록, 연간 달력 데이터 |
| multiWayType, airportSelectType | enum | 현재 다구간 타입, 공항 선택 위치 |

##### 핵심 함수 및 역할

| 함수명 | 역할 |
|--------|------|
| init() | 초기 데이터 병렬 호출 (배너, 예약, 휴일) |
| onJourneyClick() | 여정 타입(편도/왕복/다구간) 클릭 시 UI 상태 변경 |
| onDateClick(), setDateInfo() | 날짜 바텀시트 호출 및 날짜 정보 설정 |
| onAirportClick(), setAirportInfo() | 공항 선택 및 저장 처리 |
| onPassengerInfoClick() 외 | 인원 정보 관련 BottomSheet 호출 및 변경 |
| checkDuplicateAirport() | 출발/도착 공항 중복 확인 |
| getSelectedDateInfo(), getCurrentAirportCode() | 현재 선택된 날짜 또는 공항코드 반환 |
| commitJourneyInfo() | 여정 정보 글로벌 저장 및 최근 경로 로컬 저장 |
| getReservationList(), reservationListReloadClick() | 예약 리스트 불러오기 및 갱신 |
| getRecentRouteList(), deleteRecentRouteList() | 최근 경로 로컬 조회 및 삭제 |
| onSwapFirstRouteClick(), onSwapSecondRouteClick() | 출도착지 교환 처리 |
| getYearCalendar() | 달력 렌더링용 연간 날짜 및 더미 데이터 생성 |

##### 상태 및 동작 흐름 요약
- init() 시 Single.zip을 사용하여 병렬 API 호출 → 결과는 각각의 LiveData에 반영됨
- 사용자 인터랙션 발생 시 LiveData 변경 → UI 자동 반응 (데이터 바인딩 기반)
- 공항, 날짜, 인원, 예약은 BottomSheet를 통해 설정하며 Event<MainBottomSheet>로 View에 통지
- 여정 정보는 ALGlobalData에 저장 후 검색 진행
- 중복 공항 체크, 날짜 유효성 체크와 같은 유효성 검사 포함
- 비동기 작업은 RxJava 기반, subscribeOn 및 observeOn 구성 사용

## al.ui.passengerinfoadd
### Codes

#### passengerinfoadd.bottomsheet.ALNationalitySelectForAddBottomSheet
##### 역할
- 여권정보 입력 시 국적 선택을 위한 BottomSheet
- ALPassengerInfoAddViewModel의 국적 리스트를 가져와 ALNationalityAdapter를 통해 리스트 표시
- 사용자가 국적을 선택하면 콜백을 통해 선택 값을 전달하고 BottomSheet를 닫음

#### passengerinfoadd.ALPassengerInfoAddActivity
##### 역할
- 항공 여정 예약 시 필요한 탑승객 정보(이름, 생년월일, 성별, 국적)를 입력받는 Activity
- 사용자 입력값에 대한 유효성 검증, ViewModel 바인딩, 국적 선택 BottomSheet 연동을 수행
- ViewModel(ALPassengerInfoAddViewModel)을 통해 입력된 데이터를 저장하고 다음 단계로 넘김

#### passengerinfoadd.ALPassengerInfoAddViewModel
##### 역할
- 탑승자 정보 추가 화면(ALPassengerInfoAddActivity)에서 사용자의 입력값을 저장 및 검증하는 ViewModel
- 탑승자 성명, 생년월일, 국적, 성별의 정보를 수집하고 유효성 검사 수행
- 국적에 따라 한글과 영문 이름 형식 체크 및 로컬 DB(ALPassengerInfoRepository)에 저장
- 저장 성공 시 다음 화면 이동 이벤트(_onNextStep)를 발생시킴

## al.ui.passengerinfoinput
### Codes
#### passengerinfoinput.adapter.ALFareDiscountRuleAdapter
##### 역할
- 할인 규칙 목록(FareDiscountRule)을 보여주는 RecyclerView Adapter
- 현재 선택된 할인 코드(currentPtcCode)와 비교하여 체크 표시를 표시하고 숨김 처리
- 각 항목 클릭 시 콜백을 통해 선택 항목을 외부로 전달
- 접근성 향상을 위해 accessibilityDelegate를 활용해 선택 여부 표시

#### passengerinfoinput.adapter.ALNationalityAdapter
##### 역할
- 국적 목록(nationalityList)을 표시하는 RecyclerView Adapter
- 현재 선택된 국적(currentNationalityCode)과 비교하여 선택 상태를 시각적으로 표시
- 항목 클릭 시 외부 콜백을 통해 선택된 항목 전달
- 접근성을 위해 accessibilityDelegate를 통해 선택 상태를 알림

#### passengerinfoinput.adapter.ALPassengerInfoInputAdapter
##### 역할
- 탑승자 정보를 입력받는 RecyclerView Adapter
- 첫 번째 아이템은 예약자 정보를 입력하는 뷰, 나머지는 개별 탑승자 정보를 입력하는 뷰
- 탑승자 정보는 이름, 성, 생년월일, 성별, 국적, 운임 할인 정보를 포함
- 예약자 정보와 동일 체크 시 자동으로 입력 필드 채우기 가능
- ViewModel과 연결되어 실시간 데이터 반영 및 유효성 검사 처리
- 국적 및 할인 정보는 바텀시트로 선택 가능

#### passengerinfoinput.bottomsheet.ALFareDiscountRuleBottomSheet
##### 역할
- 운임 할인 규칙을 선택하는 바텀시트 UI 컴포넌트
- ALMultiWayType에 따라 첫 번째 여정 또는 두 번째 여정에 적용할 수 있는 할인 규칙 목록을 표시하고 선택 가능
- 사용자가 항목을 선택하면 ViewModel에 할인 정보 저장 후 바텀시트 닫힘
- 선택된 할인 정보는 콜백으로 호출자에게 전달 가능+

#### passengerinfoinput.bottomsheet.ALNationalitySelectBottomSheet
##### 역할
- 탑승자 국적을 선택할 수 있는 국적 선택 바텀시트
- RecyclerView를 통해 국적 리스트를 제공하고 선택된 항목을 ViewModel에 저장함
- 선택 후 콜백을 통해 호출 측에 선택된 국적명을 전달하고 바텀시트 종료

#### passengerinfoinput.ALPassengerInfoInputActivity

##### 역할
- 사용자가 입력한 여러 탑승자 정보를 확인 및 수정할 수 있는 화면 - Activity
- RecyclerView를 이용해 예약자 및 동반 탑승자 정보를 입력받고 검증
- 승객 선택 화면, 결제 화면으로의 이동 처리
- ViewModel(ALPassengerInfoInputViewModel)과 바인딩하여 입력 상태를 실시간 반영

#### passengerinfoinput.ALPassengerInfoInputViewModel
##### 역할
- 여러 탑승자의 정보 입력 및 검증을 관리하는 ViewModel
- 운임 할인 규칙 조회, 탑승자 정보 유효성 검사, 예약 요청 데이터 구성, 탑승자별 요금 설정을 담당
- UI의 입력 필드와 연결되어 실시간 검증 및 상태 업데이트 제공

##### 예약 진행 흐름 요약
1. getPassengerList()로 탑승자 초기 데이터 구성  
2. 사용자 입력 → setXXX()로 passengerInfo 데이터 갱신  
3. getFareDiscountRule()로 운임 할인 규칙 조회  
4. UI에서 할인 선택 시 → setFirstFareInfo() / setSecondFareInfo() 호출  
5. 유효성 체크 (checkValidation()) 통과하면 → onInputCompleteClick()에서 예약 요청 구성  
6. ALGlobalData.reservationInfo에 최종 예약 데이터 저장 후 결제화면으로 이동

##### 유효성 기준
| 항목 | 기준 |
|------|------|
| 이름/성 | nameValidation() 통과 |
| 생년월일 | dateOfBirthValidation2() 통과 |
| 나이 | 성인/소아/유아 기준 개월 수 부합 여부 (checkAge()) |
| 국적 | 한국: 한글 성명, 외국: 영문 대문자만 허용 |
| 성별 | 입력 여부 필수 |
| 이메일 | emailValidation() 통과 |

## al.ui.passengerinfolist
### Codes
#### passengerinfolist.adapter.ALPassengerInfoListAdapter
##### 역할
- 탑승자 정보 불러오기 선택 시 목록 및 추가 항목을 표시하는 RecyclerView 어댑터
- 기존 저장된 ALPassengerEntity 목록을 보여주고 선택 또는 삭제 기능 제공
- 탑승자 수가 9명 미만일 경우 마지막 항목에 추가하기 뷰를 함께 표시

#### passengerinfolist.ALPassengerInfoListActivity
##### 역할
- 저장된 탑승자 정보 목록을 보여주고 선택하거나 새로 추가할 수 있는 화면
- 사용자가 특정 타입의 탑승자(성인/소아/유아)를 선택하고 완료하면 결과를 호출한 액티비티로 반환
- 삭제, 추가, 선택, 연령 체크의 UI 이벤트와 ViewModel 연동을 처리

#### passengerinfolist.ALPassengerInfoListViewModel
##### 역할
- 저장된 탑승자 목록(ALPassengerEntity) 조회, 삭제, 선택 처리
- 좌석 유형(성인/소아/유아)에 따른 나이 조건 체크 로직

##### 나이 체크 로직
| 나이 구간 | 개월 수 | 좌석 타입 허용 |
|-----------|---------|----------------|
| 성인      | 156개월 이상 (만 13세 이상) | ADULT만 가능 |
| 소아      | 24개월 이상 ~ 156개월 미만 (만 2세 이상 ~ 13세 미만) | CHILD만 가능 |
| 유아      | 24개월 미만 | INFANT, 단 CHILD 좌석 예매는 허용 |

##### 특징
- Factory 객체를 통해 Activity에서 ViewModel 생성 시 ALPassengerInfoRepository 주입
- 의존성 주입을 위한 커스텀 팩토리 패턴 활용

## al.ui.payment
### Codes
#### payment.adapter.ALFareDetailAdapter
##### 역할
- 결제 화면에서 탑승자별 운임 상세 정보를 표시하는 RecyclerView 어댑터
- 운임, 세금, 유류할증료, 발권수수료 및 할인 금액을 계산 및 표시
- ALPaymentViewModel을 참조하여 원래 내야하는 운임 및 총합 계산 수행


##### 할인 계산 및 표시
| 항목 | 로직 |
|------|------|
| discountAirFare | 기본운임 |
| discountAirTax | 기본세금 |
| discountFuelCharge | 기본유류할증료 |
| discountTkFee | 기본발권수수료 |

- 위 값이 0이 아닌 경우에만 할인 금액 및 뷰(cl4, cl6, cl8, cl10)를 VISIBLE 처리  
- 모든 할인값이 0인 경우엔 할인 태그(tvDiscountTag) 숨김


#### payment.adapter.ALPayemntCardAdapter
##### 역할
- 결제 카드 리스트(등록된 카드 및 새 카드 등록)를 RecyclerView에 표시하는 Adapter
- 카드 정보(ALPaymentCardModel.PaymentCard)를 뷰에 바인딩하고 화면 너비에 따라 카드번호 폰트 크기를 동적으로 조절
- 마지막 항목에 카드 등록하기 버튼을 포함하여 클릭 시 onCardResister() 콜백 실행


#### payment.adapter.ALTermsAdapter
##### 역할
- 결제 화면에서 약관 및 운임규정 항목을 RecyclerView에 표시하는 Adapter
- 각 항목은 일반 약관(termsList), 여정 1의 운임규정, 여정 2의 운임규정(왕복일 경우에만) 중 하나
- 약관 항목을 클릭 시 onTermsClick(item) 실행하며 운임규정 항목을 클릭 시 onFareRuleClick(Boolean 값) 실행
- termsAgreeList를 통해 약관 동의 여부 상태를 체크하고 토글
- 시각 장애인 접근성을 고려하여 accessibilityDelegate에서 contentDescription, isChecked 설정


#### payment.bottomsheet.ALFareDetailBottomSheet
##### 역할
- 항공 운임 상세 내역을 사용자에게 보여주는 BottomSheetDialog
- ALPaymentViewModel에서 제공하는 전체 운임 정보를 ALFareDetailAdapter로 바인딩하여 RecyclerView에 표시
- 바텀시트 UI는 반쯤 열려 있는 상태에서 시작되며 슬라이드 시 더 확장 가능
- 하단 여백을 슬라이드 비율에 따라 실시간으로 조절하여 하단 뷰가 화면 밖으로 밀려나는 문제를 방지

#### payment.bottomsheet.ALKoreanAirPaymentGuideBottomSheet
##### 역할
- 대한항공 결제 전 안내 메시지를 보여주는 BottomSheetDialog
- 사용자에게 결제 유의사항을 강조된 텍스트 스타일로 안내하고 확인 시 onPayment 콜백을 실행하여 결제 프로세스를 진행
- Android 버전에 따라 TypefaceSpan 또는 커스텀 MetricAffectingSpan을 적용하여 텍스트 폰트 스타일링 처리

#### payment.ALPaymentActivity
##### 역할
- 항공 결제 프로세스 전체를 담당하는 Activity
- 결제 수단 선택, 약관 동의, 카드 선택 및 등록, 결제 방식(일반과 이지페이) 처리
- 대한항공 결제 시 별도 가이드 바텀시트 제공
- 결제 성공와 실패 여부에 따라 ALPaymentSuccessActivity로 전환
- 카드 등록 및 결제 시 외부 WebView 및 TransKey 암호 키패드와 연동
- ViewModel과 데이터 바인딩, 각종 Observer를 통해 UI 상태를 실시간 반영

#### payment.ALPaymentViewModel
##### 역할
- AL 항공권 결제 전용 액티비티
- 결제 방법 선택(일반 결제와 이지페이) 및 카드 선택과 등록을 처리함
- 약관 동의, 결제 가능 체크, 운임 규정 확인, 할부 설정과 같은 결제 관련 모든 UI 및 로직을 담당
- 대한항공 결제의 경우 전용 안내 바텀시트를 표시하고 분기 처리
- ALPaymentViewModel과 연동하여 결제 상태를 관찰하고 결제 성공과 실패 시 다음 화면으로 이동
- 외부 WebView(카드 등록, 일반 결제) 및 TransKey 암호화 키패드와 연동
- 백버튼 및 액션바 이벤트를 통해 예약 취소 안내 다이얼로그 표시

## al.ui.paymentsuccess
### Codes
#### paymentsuccess.adapter.ALFareDetailForPaymentSuccessAdapter
##### 역할
- 결제 성공 화면에서 여정별과 승객별 운임 상세 정보를 RecyclerView에 표시하는 어댑터
- ALAppFareInfo 리스트를 기반으로 실제 요금, 할인 금액, 총 합계을 계산하여 UI에 반영
- 여정1과 여정2 구분 및 성인, 소아, 유아와 같은 승객 타입에 따른 텍스트 설정
- 할인 항목(항공운임, 공항세, 유류할증료, 발권수수료)이 존재하는 경우에만 해당 UI를 표시
- 마지막 아이템의 경우 전체 여정 총 결제 금액(viewModel.totalTravelFare)을 하단에 노출

#### paymentsuccess.bottomsheet.ALFareDetailForPaymentSuccessBottomSheet
##### 역할
- 항공 결제 성공 화면에서 여정별 요금 상세 정보를 바텀시트 형태로 보여주는 UI 컴포넌트
- ALPaymentSuccessViewModel에서 받은 운임 정보를 ALFareDetailForPaymentSuccessAdapter를 통해 바인딩
- BottomSheetBehavior를 이용해 STATE_COLLAPSED 상태에서 60% 화면 비율로 보이도록 설정
- 유저가 바텀시트를 스와이프할 경우 아래 여백 영역 높이를 실시간으로 조정해 부드러운 UX 제공
- 확인 버튼 클릭 시 바텀시트 종료

#### paymentsuccess.ALPaymentSuccessActivity
##### 역할
- 결제 완료 후 결과(성공과 실패)를 보여주는 화면을 구성하는 액티비티
- 결제 성공 여부에 따라 서로 다른 UI, 메시지, 광고 배너, 알림을 설정
- 결제 성공 시 성공 이미지 및 메시지 출력, 운임 상세 정보 버튼 및 배너 표시, 마케팅 알림 동의 바텀시트 호출, Braze와 Airbridge 추적 정보 전송
- 결제 실패 시 실패 이미지 및 메시지 출력, 실패 사유가 존재할 경우 UI에 표시
- 프로모션 배너 이미지를 Glide로 비동기 로딩하고 클릭 시 외부 링크 이동
- 뒤로가기 시 메인 화면으로 이동하도록 콜백 오버라이드

#### paymentsuccess.ALPaymentSuccessViewModel
##### 역할
- 항공권 결제 성공 화면을 위한 상태 관리 및 데이터 가공 로직을 담당하는 ViewModel
- ALGlobalData로부터 여정 및 탑승객 데이터를 받아 LiveData로 가공해 UI에 제공
##### 기능 상세 설명
| 기능 | 설명 |
|------|------|
| 금액 계산 (travelAmount) | 여정별 탑승객 운임 정보를 합산하여 총 운임 계산 및 포맷 적용 |
| 탑승객 정보 구성 (travelPassenger) | 탑승객 수, 좌석 타입과 같은 정보를 문자열로 구성 |
| 요금 상세 정보 리스트화 (getAllFareInfoList) | ALAppFareInfo 리스트로 여정1과 여정2 요금 및 인원별 요금 세부 정보 구성 |
| 즐겨찾기 동기화 (getBookmarkList, onBookmarkClick) | 즐겨찾기 여부 확인 및 추가와 삭제 기능 |
| 수하물 정보 정리 (setBaggageInfo) | 여정별 무료 수하물 정보 정리 및 문자열 구성 |
| 프로모션 배너 불러오기 (callPromotionBanner) | 결제 성공 화면의 광고 배너 API 호출 |
| 추적 정보 전송 (sendBrazeAirbridge) | Braze, Airbridge에 탑승지와 도착지, 금액, 탑승일 정보 전송 |
| 원가 요금 조회 (getFirstOriginFare, getSecondOriginFare) | 탑승객 타입별 원 운임 정보 반환 - 할인 전 가격 대비 목적 |
| 서비스 연동 (linkService) | 항공 연동 서비스 등록 여부 확인 및 처리 |

## al.ui.reservationcancel
### Codes
#### reservationcancel.bottomsheet.ALAdvanceCheckInGuideBottomSheet
##### 역할
- 사전 체크인 관련 안내 메시지를 하단 바텀시트 형태로 보여주는 UI 컴포넌트
- 사용자에게 안내 메시지와 함께 확인 또는 취소 선택을 받을 수 있도록 양방향 버튼 제공
- 각 버튼 클릭 시 전달받은 콜백 함수를 실행하고 바텀시트를 닫는 구조
- 체크인 유무에 따른 사용자 선택을 유도

#### reservationcancel.ALReservationCancelActivity
##### 역할
- 항공 예약 취소를 위한 UI와 로직을 담당
- ViewModel로부터 전달받은 운임 규정 정보 리스트를 RecyclerView에 바인딩하여 사용자에게 제공  
- 사전 체크인이 완료된 경우 안내 바텀시트를 통해 추가 안내 및 웹 브라우저 연결 기능 제공  
- 예약 취소 진행 시 AlertDialog 또는 다음 화면으로 이동하는 분기 처리 수행  
- 체크박스를 통해 사용자 동의를 받고 확인 버튼을 활성화

#### reservationcancel.ALReservationCancelViewModel
##### 역할
- 항공 예약 취소 화면의 데이터 처리 및 비즈니스 로직을 담당하는 ViewModel
- 예약 상세 정보 및 취소 사유와 운임 정보를 초기화하고 LiveData로 UI에 전달
- 즐겨찾기 등록 여부 확인 및 추가와 삭제 기능 제공
- 예약 취소 요청 시 ALReservationCancelRequest를 구성하여 서버에 전달하고 결과에 따라 다음 단계 분기 처리
- 사전 체크인 완료된 예약에 대해서는 별도의 URL(웹뷰) 안내를 위한 이벤트 발생 처리 포함
- 성인, 소아, 유아별 운임 금액 계산 및 UI에 전달

## al.ui.reservationcancelbyreason
### Codes
#### reservationcancelbyreason.adapter.ALReservationCancelGuideAdapter
##### 역할
- 항공 예약 취소 사유에 따른 안내 문구들을 RecyclerView로 표시하는 어댑터
- 첫 번째 항목에만 타이틀을 표시하고 이후 항목에는 숨기는 방식으로 UI 구성
- CharSequence로 전달된 안내 문구 리스트를 순차적으로 바인딩하여 텍스트뷰에 출력
- 뷰 홀더 내부에서 타이틀과 설명을 동시에 설정

#### reservationcancelbyreason.ALReservationCancelByReasonActivity
##### 역할
- 항공 예약 취소 사유에 따라 사용자에게 안내 문구를 제공하는 화면
- Intent에서 사유 코드를 받아 해당 사유에 맞는 안내 리스트를 구성하고 RecyclerView에 바인딩
- 특정 텍스트에 색상 강조 및 굵기를 적용하여 시각적으로 강조된 메시지 제공
- 체크박스 동의 여부에 따라 취소 요청 버튼 활성화
- 예약 취소 요청이 완료되면 AlertDialog로 완료 안내 후 예약 관련 액티비티 스택 종료

#### reservationcancelbyreason.ALReservationCancelByReasonViewModel
##### 역할
- 항공 예약 취소 사유가 있는 경우 해당 사유에 따라 서버에 취소 요청을 보내는 ViewModel
- 예약 상세 정보와 취소 사유 코드를 Intent로부터 전달받아 내부 변수로 저장
- 예약 취소 버튼 클릭 시 ALReservationCancelByReasonRequest 객체를 구성하여 서버에 API 요청 수행
- 예약 취소 완료 시 LiveData 이벤트를 통해 UI에 알리고 여정 동기화를 위한 MJManager.syncMyJourneyWithServer() 호출

## al.ui.reservationdetail
### Codes
#### reservationdetail.adapter.ALCancelReasonAdapter
##### 역할
- 예약 상세 화면에서 취소 사유 선택을 위한 RecyclerView 어댑터
- ALBaseCodeInfoModel.Code 리스트를 받아 각 항목의 codeName을 UI에 표시
- 항목 클릭 시 전달받은 onItemClick 콜백을 통해 선택된 사유를 외부로 전달
- 사용자 선택 이벤트를 처리하기 위해 각 아이템에 click 확장 함수 적용

#### reservationdetail.adapter.ALFareDetailForReservationAdapter
##### 역할
- 항공 예약 상세 화면에서 승객별 운임 정보를 표시하는 RecyclerView 어댑터
- ALAppFareInfo 리스트를 기반으로 항공 운임, 공항세, 유류할증료, 발권수수료, 총 합계를 표시
- 성인, 소아, 유아 구분에 따른 타입별 텍스트를 설정하고 이름 및 운임 정보를 바인딩
- 리스트의 마지막 아이템일 경우 ViewModel에서 전달된 총 결제 금액을 하단에 표시
- 여정 시작 여부(isFirstIndex)에 따라 구분선과 타이틀 표시 여부를 조정

#### reservationdetail.adapter.ALPassengerInfoAdapter
##### 역할
- 예약 상세 화면에서 탑승객의 정보를 RecyclerView 형태로 보여주는 어댑터
- 성명, 국적, 생년월일, 성별 정보를 ALReservationCompleteDetailModel.PassengerInfo로부터 바인딩
- 생년월일은 yyyy-MM-dd 포맷 문자열을 파싱 후 사용자 친화적인 형태로 변환하여 표시
- 성별은 M과 F 값을 성별 문자열로 매핑하여 출력

#### reservationdetail.bottomsheet.ALCancelReasonSelectBottomSheet
##### 역할
- 항공 예약 취소 사유를 선택할 수 있도록 제공하는 바텀시트 UI 컴포넌트
- 항공사 코드에 따라 코로나 관련 취소 사유 항목을 포함 여부를 조건부로 결정
- 사용자 선택 시 콜백으로 선택된 사유 데이터를 전달하고 바텀시트 종료
- 유의사항 텍스트 클릭 시 별도 안내 바텀시트(ALReservationCancelNoticeBottomSheet) 호출

#### reservationdetail.bottomsheet.ALFareDetailForReservationBottomSheet
##### 역할
- 예약 상세 화면에서 여정별 및 승객별 운임 정보를 바텀시트 형태로 보여주는 UI 컴포넌트
- RecyclerView를 통해 ALFareDetailForReservationAdapter를 활용하여 요금 항목 리스트 표시
- BottomSheetBehavior를 사용해 1:0.6 비율로 노출되며 slideOffset에 따라 하단 여백(vDummy) 높이를 동적으로 조절
- 버튼 클릭 시 바텀시트를 종료

#### reservationdetail.bottomsheet.ALReservationCancelNoticeBottomSheet
##### 역할
- 항공 예약 취소 시 유의사항을 안내하는 바텀시트 UI 컴포넌트
- 확인 버튼(btnConfirm) 클릭 시 바텀시트 종료

#### reservationdetail.ALReservationDetailActivity
##### 역할
- 항공 예약 상세 정보를 표시하는 화면으로, 예약 상태, 승객 정보, 운임 정보를 전반적으로 확인 가능
- 예약 상세 정보 초기화 및 관리를 위한 ALReservationDetailViewModel과 연동
- 예약 취소 사유 선택, 운임 상세 보기, 운임규정 조회, 배너 링크 이동의 다양한 사용자 반응을 처리
- 예매 취소 사유 코드(NS - 일반 예매 취소, DC - 결항 및 지연 취소, CI - 코로나19 취소)에 따라 일반 예매 취소 화면 또는 사유 기반 예매 취소 화면으로 분기
- 예약 상태 코드가 변경된 경우 결과 코드 설정 후 종료하여 예약 리스트 리프레시
- Glide를 활용한 광고 배너 이미지 로딩 및 접근성(contentDescription) 설정

#### reservationdetail.ALReservationDetailViewModel
##### 역할
- 항공 예약 상세 화면의 데이터 로직을 담당하는 ViewModel
- 예약 상세 정보, 배너, 즐겨찾기 여부와 같은 다양한 UI 상태를 LiveData로 관리
- 예약 번호 기반 예약 상세 정보 및 프로모션 배너 동시 조회 및 초기화 처리
- 운임 정보 리스트 계산 및 탑승객 운임 총액, 취소 금액, 취소 수수료 계산
- 운임규정 정보가 있으면 바로 이동하고 없으면 비동기 조회 후 화면 전환 유도
- 예약 상태와 도착 시간에 따라 취소 가능 여부 판단 및 예약 취소 버튼 노출 제어
- 즐겨찾기 목록 불러오기 및 즐겨찾기 추가와 삭제 처리

## al.ui.searchloading
### Codes
#### searchloading.ALSearchLoadingActivity
##### 역할
- 항공편 검색 후 로딩 애니메이션을 보여주는 액티비티
- ViewModel의 onNextStep 이벤트를 관찰하여 결과 화면인 ALTravelListActivity로 이동
- 뒤로가기 버튼 설정 및 결과 반환 처리
- ALTravelListActivity에서 돌아왔을 때 예약 리스트 갱신 여부에 따라 결과 코드 설정 후 종료

#### searchloading.ALSearchLoadingViewModel
##### 역할
- 항공편 검색 요청을 수행하는 ViewModel
- ALGlobalData.journeyInfo의 여정 정보와 승객 정보를 바탕으로 ALTravelListRequest 구성
- 왕복, 편도, 다구간 여부에 따라 여정 정보를 다르게 설정
- travelRepository를 통해 항공편 조회 API 호출
- 조회 성공 시 ALGlobalData.travelInfo에 결과 저장 후 onNextStep 이벤트 발생시켜 화면 전환 유도

## al.ui.summary
### Codes
#### summary.adapter.ALFareDetailForSummaryAdapter
##### 역할
- 여정 요약 화면에서 탑승객별 상세 운임 정보를 표시하는 RecyclerView Adapter
- 각 여정 및 탑승객 타입에 따른 운임 구성 요소(운임, 공항세, 유류할증료, 발권 수수료)를 ViewModel에서 조회하여 바인딩
- 첫 번째 여정 여부, 탑승객 인덱스를 기반으로 타이틀 및 구분선 출력 제어
- 마지막 아이템일 경우 총 결제 금액도 함께 표시

#### summary.adapter.ALFreeBaggageAdapter
##### 역할
- 여정 요약 화면에서 무료 수하물 정보를 표시하는 RecyclerView Adapter
- ALTravelListModel.FreeBaggage 데이터를 기반으로 수하물 서비스 이름, 수치, 단위를 조합해 항목별 텍스트로 바인딩
- 수하물 단위명은 글로벌 코드 리스트(ALGlobalData.unitList)에서 조회하여 출력

#### summary.bottomsheet.ALFareDetailForSummaryBottomSheet
##### 역할
- 여정 요약 화면에서 운임 상세 정보를 보여주는 BottomSheet
- BottomSheetBehavior의 COLLAPSED를 활용해 반쯤 펼쳐진 상태처럼 보이도록 설정
- 슬라이드 이벤트에 따라 바텀시트 내부 더미 뷰(vDummy)의 높이를 조정해 하단 위치를 유지
- ViewModel에서 가져온 운임 정보 리스트를 어댑터에 연결하여 RecyclerView로 출력
- 확인 버튼 클릭 시 바텀시트 닫기 동작 수행

#### summary.ALSummaryActivity
##### 역할
- 항공 여정 요약 화면을 구성하는 Activity
- 탑승 정보, 운임 규정, 운임 상세 바텀시트를 표시하고 사용자 이벤트를 처리
- 뒤로가기 또는 UI에서 가는편과 오는편 재선택 시 알림 다이얼로그를 통해 확인 후 종료 또는 재검색 처리
- 운임 규정 클릭 시 ViewModel에서 규정 정보를 조회한 후 ALFareRulesActivity로 이동
- 운임 상세 보기 클릭 시 ALFareDetailForSummaryBottomSheet 표시
- 탑승객 정보 입력 버튼 클릭 시 ALPassengerInfoInputActivity로 이동

#### summary.ALSummaryViewModel
##### 역할
- 여정 정보, 운임 규정, 운임 총액, 탑승객 정보, 수하물 정보의 요약 데이터를 관리하는 ViewModel
- ALGlobalData에서 여정 및 운임 데이터를 불러와 LiveData로 바인딩
- 총 운임 계산 및 포맷팅 처리 (totalFare)
- 편도, 왕복, 다구간 여정의 탑승객별 운임 정보를 리스트로 생성 (getAllFareInfoList)
- 각 여정의 운임 규정 정보 조회 (getFareRule)
- 탑승객 인원 및 좌석 타입에 따른 텍스트 설정 (travelPassenger)
- 각 여정에 포함된 무료 수하물 정보를 문자열로 저장 (setBaggageInfo)
- 여정 내 승객 타입(성인, 소아, 유아)에 따른 요금 정보 제공 (getFirstOriginFare, getSecondOriginFare)

## al.ui.travellist
### Codes
#### travellist.adapter.ALTravelDateSelectAdapter
##### 역할
- 여정 날짜 선택 UI를 구성하는 RecyclerView.Adapter
- 사용자가 선택 가능한 날짜 리스트를 전달받아 각 날짜를 리스트로 렌더링
- 선택된 날짜를 기준으로 스타일을 동적으로 변경 (선택 날짜는 Bold, 배경 색상 변경)
- 날짜 클릭 시 onItemClick(Calendar) 콜백 호출 후 선택 상태 갱신 (notifyDataSetChanged)
- 내부적으로 날짜 포맷 변환을 위해 displayDatePattern2, yyyyMMddPattern와 같은 확장 함수 사용

#### travellist.adapter.ALTravelFilterAirlineListAdapter
##### 역할
- 항공사 필터링 리스트를 위한 RecyclerView Adapter
- 항공사 목록을 체크박스 형태로 표시
- 각 항목의 체크 상태(isChecked)는 ALBaseCodeInfoModel.Code의 isChecked 필드를 사용해 초기화 및 갱신
- 체크 변경 이벤트 발생 시 onChecked 콜백 호출
- 접근성 이벤트 대응 - TYPE_VIEW_CLICKED 이벤트를 수신하면 상태 동기화

#### al.ui.travellist.adapter.ALTravelListAdapter  
##### 역할  
- 항공편 리스트 화면을 구성하는 RecyclerView Adapter  
- ALTravelListModel.Travel 리스트 데이터를 받아 각 항목을 렌더링  
- 항공사 로고, 항공편명, 출도착 시간, 비행시간, 좌석 정보, 요금의 UI 바인딩 처리  
- Glide를 이용해 항공사 로고를 이미지뷰에 로딩
- 공동운항 여부에 따라 각각의 UI 컴포넌트 제어  
- bookingClassType 값에 따라 일반, 할인, 특가 스타일 적용  
- cabinClass 값이 일반석이 아닌 경우 좌석명을 표시  
- 사용자가 항공편 항목을 클릭하면 onItemClick 콜백을 통해 이벤트 전달  
- 항공요금은 wonFormat 함수로 형식화하여 표시  
- 출도착 시간은 hhmmPattern 확장 함수를 사용해 시:분 형태로 포맷팅  
- 항공사 코드와 항공편 번호를 결합해 항공편명을 생성

#### al.ui.travellist.adapter.ALTravelSortAdapter  
##### 역할  
- 항공편 정렬 기준을 표시하는 RecyclerView Adapter  
- sortList로 전달받은 문자열 리스트 생성
- 선택된 항목은 글자 색상, 배경색, 폰트를 다르게 적용하고 체크 이미지 표시  
- 비선택 항목은 일반 스타일로 렌더링  
- clSpinnerContainer에 접근성 속성(accessibilityDelegate)을 설정해 스크린 리더가 선택 여부를 인식 가능하도록 처리  
- 사용자 클릭 시 onItemClick 콜백을 통해 항목 문자열과 위치(adapterPosition) 전달

#### al.ui.travellist.bottomsheet.ALNumberPickerBottomSheet  
##### 역할  
- 사용자가 금액을 선택할 수 있도록 NumberPicker 형태로 구성된 BottomSheetDialogFragment  
- 타이틀, 현재 금액, 시작 금액, 종료 금액, 증가 단위를 인자로 전달받아 숫자 목록을 생성  
- 생성된 숫자 리스트는 통화 포맷(###,###)을 적용해 화면에 표시  
- 선택된 금액은 currentAmount() 함수로 가져오며 confirm 버튼을 눌렀을 때 콜백으로 전달됨  
- confirm 버튼은 유효한 범위 내 금액을 선택한 경우에만 활성화  
- 접근성 대응을 위해 NumberPicker 값 변경 시 contentDescription을 업데이트하고 이벤트를 강제로 전송  
- confirm 버튼, close 버튼 클릭 시 각각 콜백 호출 또는 BottomSheet 종료 처리  
- 금액 문자열을 Int로 변환하기 위해 fromDecimalToIntOrNull 함수 사용  

#### al.ui.travellist.bottomsheet.ALTimePickerBottomSheet  
##### 역할  
- 사용자가 시간을 선택할 수 있도록 시 및 분 단위 NumberPicker를 제공하는 BottomSheetDialogFragment
- 시작 시간과 종료 시간 범위 내에서 선택 가능한 시각 데이터를 동적으로 생성
- 시는 createHourArray 함수에서 2자리 문자열 리스트로 생성  
- 분은 고정된 7단계(00, 10, 20, 30, 40, 50, 59)로 구성됨  
- currentTime을 기반으로 초기 선택 값을 설정하고 사용자 변경 시 contentDescription과 접근성 이벤트 전송  
- 선택된 시각 값은 currentHour, currentMinute 함수로 추출하여 콜백에 전달  
- 유효한 시간 범위 내에 있을 경우에만 확인 버튼 활성화 및 접근성 문자열 업데이트  
- 확인 버튼 클릭 시 선택된 시간(hour, minute)을 콜백으로 전달하고 BottomSheet 종료  
- companion object의 newInstance를 통해 제목, 현재 시간, 시작/종료 시간 인자를 넘겨 동적 생성  


#### al.ui.travellist.bottomsheet.ALTravelFilterBottomSheet
##### 역할
- 항공편 검색 시 적용할 필터 조건(항공사, 출발 시간, 운임 가격)을 설정하는 BottomSheetDialogFragment
- 항공사 필터링
  - ALGlobalData.airlineList 기반으로 체크박스 리스트 생성
  - 선택 상태는 ViewModel의 getTravelFilter().airlineList 값과 동기화
  - 전체 선택 체크박스를 통해 전체 항목 상태 일괄 변경
  - 접근성 이벤트(TYPE_VIEW_CLICKED) 발생 시 상태를 업데이트하여 스크린 리더 지원
- 출발 시간 필터링
  - 시작/종료 시간 선택 UI 제공 (ALTimePickerBottomSheet 호출)
  - 선택된 시간은 HH:mm 형식으로 표시 및 contentDescription 업데이트
  - ViewModel의 임시 값(tempFilterStartTime, tempFilterEndTime)과 동기화
- 운임 가격 필터링
  - 시작/종료 금액 선택 UI 제공 (ALNumberPickerBottomSheet 호출)
  - 선택된 금액은 통화 형식(###,###)으로 표시 및 contentDescription 업데이트
  - ViewModel의 임시 값(tempFilterStartAmount, tempFilterEndAmount)과 동기화
- 초기화 기능
  - lInit 클릭 시 모든 항공사 선택, 시간/금액 범위를 기본값으로 재설정
- 적용하기 버튼
  - 선택된 필터 조건을 ViewModel에 저장(setTravelFilter)하고 BottomSheet 닫기
- UI 및 접근성
  - BottomSheet 초기 상태를 확장 모드로 설정, 상단 여백 조절
  - GridLayoutManager(2열)로 항공사 목록 표시
  - 선택 값 변경 시 버튼 활성고ㅏ 활성 상태 갱신

#### al.ui.travellist.bottomsheet.ALTravelSortBottomSheet
##### 역할
- 항공편 리스트의 정렬 기준을 선택하는 BottomSheetDialogFragment
- 정렬 기준 목록은 문자열 리소스에서 불러와 RecyclerView로 표시 (ALTravelSortAdapter 사용)
- 현재 선택된 정렬 타입은 ViewModel의 currentSortedType을 기반으로 표시
- 사용자가 항목을 클릭하면 선택된 정렬 기준과 위치를 onItemClick 콜백으로 전달 후 BottomSheet 종료
- 닫기 버튼(ivClose) 클릭 시 BottomSheet 종료
- BottomSheet 초기 상태를 확장 모드로 설정하고 접힘 상태를 건너뛰도록 구성

#### al.ui.travellist.ALTravelListActivity
##### 역할
- 항공편 여정 목록 화면의 메인 액티비티
- ViewModel 바인딩 및 뒤로가기 콜백 등록, 액션바 뒤로가기 처리
- 정렬 바텀시트(ALTravelSortBottomSheet) 호출 및 선택 결과로 리스트 정렬 실행
- 필터 바텀시트(ALTravelFilterBottomSheet) 호출로 항공사 조건, 시간 조건, 운임 조건 설정
- 조건 변경 트리거(tvChange1, tvChange2, llConditionChange) 클릭 시 재선택 경고 AlertDialog 노출 후 흐름 분기
- selectFirstTravel 관찰
  - 출도착 시간(hhmmPattern), 총 운임(wonFormat) UI 반영 및 접근성(contentDescription) 세팅
  - 재선택 포커스 안내(focusTalkBack)
- nearDateList 관찰
  - ALTravelDateSelectAdapter로 날짜 리스트 표시 및 선택 시 재조회(requestTravelList)
  - 선택 기준일과 동일한 날짜 인덱스를 가운데 정렬(setCenterToPosition)
- travelList 관찰
  - Lottie 로딩 애니메이션 정리 후 ALTravelListAdapter로 항목 표시
  - 항목 클릭 시 로그인 여부 체크, 미로그인 시 로그인 플로우 호출(TGoLoginManager)
- onNextStep(Event) 관찰
  - ALSummaryActivity로 이동(startActivityForResult, REQ_CODE_SUMMARY)
- 결과 처리(onActivityResult)
  - 요약 화면 완료 시 재선택 코드 처리(가는편과 오는편) 또는 종료 흐름
- finishAndLoginCheck
  - 직전 로그인 시도 후 로그인 성공 상태라면 예약 목록 갱신 코드 setResult 전달
  - 액티비티 종료
- 기타
  - ViewModel 인스턴스 팩토리로 생성 및 액티비티 범위 공유
  - 클릭 편의 확장(click, clicks) 사용으로 View 바인딩 간결화

#### al.ui.travellist.ALTravelListViewModel
##### 역할
- 항공편 여정 목록 화면(ALTravelListActivity)의 ViewModel
- ALGlobalData에 저장된 여정 정보와 검색 결과를 기반으로 화면 상태 관리
- LiveData 관리
  - journeyInfo: 현재 여정 정보
  - travelList: 현재 표시 중인 항공편 리스트
  - selectFirstTravel: 선택된 첫 번째 여정(가는편)
  - onNextStep: 다음 단계 이동 이벤트
  - isReSearching: 재조회 진행 여부
  - nearDateList: 출발일 기준 ±5일의 날짜 리스트
  - journeyTitle: 여정 단계별 제목
- 여정 재조회(requestTravelList)
  - 선택된 날짜를 여정 정보에 반영
  - 여정 타입(왕복, 편도, 다구간)에 따라 요청 파라미터(ALTravelListRequest) 구성
  - API 호출을 통해 항공편 리스트를 새로 불러오고 정렬·필터 적용 후 LiveData 갱신
- setTravelNearDateList
  - 현재 여정 단계와 조건에 따라 날짜 범위 계산 후 ±5일 리스트 생성
- sortTravelList
  - 전달받은 정렬 타입(currentSortedType)에 따라 항공편 리스트 재정렬
- sortedList
  - 출발 시간, 가격, 소요 시간 기준으로 리스트 정렬
- filteredList
  - 출발 시간 범위, 운임 가격 범위, 항공사 조건에 따라 리스트 필터링
- setTravelInfo
  - 편도: 선택 항공편 저장 후 다음 단계 이벤트 발생
  - 왕복/다구간: 가는편 미선택 시 selectFirstTravel로 저장, 이미 선택된 경우 가는편·오는편 저장 후 다음 단계 이벤트 발생
  - 선택 후 날짜 리스트 및 여정 제목 갱신
- setJourneyTypeTitle
  - 여정 타입 및 단계에 따라 상단 제목(journeyTitle) 갱신
- onFirstReSelectClick
  - 가는편 재선택 처리(첫 번째 여정 선택 해제 후 초기 리스트 재설정)
- getTravelFilter와 setTravelFilter
  - 현재 여정 단계에 해당하는 필터(ALAppTravelFilter) 반환·설정
  - 설정된 필터 조건(출발 시간 범위, 운임 가격 범위, 항공사)에 따라 리스트 필터링
- Factory
  - ViewModelProvider.Factory 구현으로 ALProvider.travelRepository 주입

#### al.util.TalkBackUtils
##### 역할
- TalkBack 관련 접근성 유틸리티 제공 객체
- Context 확장 함수
  - isTalkBackOn()
    - 접근성 서비스가 활성화되고 터치 탐색이 켜져 있는지 여부 반환
  - isTouchExplorationOn()
    - 터치 탐색(Touch Exploration) 기능 활성 여부 반환
- View 확장 함수
  - focusTalkBack()
    - View에 접근성 포커스를 주기 위해 TYPE_VIEW_FOCUSED 이벤트 전송 및 ACTION_ACCESSIBILITY_FOCUS 수행
    - FragmentActivity의 lifecycleScope를 사용해 500ms 지연 후 실행
  - sendTalkBack(afterView: View)
    - 현재 View의 접근성 탐색 순서를 afterView 이전으로 설정(setAccessibilityTraversalBefore)

#### al.util (displayDatePatternPair, plainString)
##### 역할
- displayDatePatternPair(calendar1, calendar2)
  - 두 개의 Calendar 객체를 받아 yyyy.MM.dd 형식의 날짜 범위 문자열로 변환
  - 두 인자가 null이면 빈 문자열 반환
  - 날짜1 ~ 날짜2 형식으로 반환
- plainString(encStr, length)
  - 암호화된 문자열(encStr)을 복호화하여 반환
  - TransKeyCipher(SEED 알고리즘) 사용
  - Util.getSecureKey(false)로 보안 키를 생성 후 복호화 수행
  - 복호화 성공 시 ByteArray에서 String으로 변환 후 원본 ByteArray를 초기화
  - 복호화 실패 또는 예외 발생 시 빈 문자열 반환
  - StringUtil.emptyIfNull로 null 안전 처리

#### al.ALConstants
##### 역할
- 항공편 예약, 결제, 조회 등 AL 모듈 전역에서 사용하는 상수 정의 객체
- SharedPreferences 키 상수
  - PREF_AL_BASE_DATA_LAST_DATE, PREF_AL_LAST_PAYMENT_METHOD 등
- 기본값 상수
  - DEFAULT_BASE_DATA_LAST_DATE (yyyyMMddhhmmss 형식)
- URL 상수
  - EASY_PAY_URL: 간편 결제 요청 웹뷰 URL(BuildConfig.TIA_TR_URL 기반)
- Intent Extra Key 상수
  - 항공편 규정, 승객 정보, 여정 상태, 예약 상세, 결제 관련 데이터 전달 키(EXTRA_KEY_...)
- 결제 파라미터 상수
  - EASY_PAY_PARAM_RESULT, EASY_PAY_PARAM_ENCRYPT_DATA 등
- 코드 그룹명 상수
  - 항공사 코드 그룹명(BASE_INFO_AIRLINE_CODE_GROUP_NAME) 등 코드 조회에 사용되는 그룹 식별자
- 요청 코드 상수
  - REQ_CODE_SUMMARY, REQ_CODE_TRAVEL_LIST 등 startActivityForResult 호출 시 구분자
- 결과 코드 상수
  - RES_CODE_FIRST_RE_SELECT, RES_CODE_IS_REFRESH_RESERVATION_LIST 등 Activity 결과 처리용
- 에러 코드 상수
  - ERROR_CODE_TS424: 특정 오류 식별 코드

#### al.ALGlobalData
##### 역할
- AL 모듈 전역에서 공유되는 항공편 예약과 조회 관련 전역 데이터 저장 객체(Singleton)
- 공통 코드·기본 데이터
  - airportList: 공항 리스트
  - airlineList: 항공사 리스트
  - nationalityList: 국적 리스트
  - seatList: 좌석 리스트
  - paxTypeList: 승객 구분 코드 리스트
  - unitList: 서비스 단위 리스트
  - genderList: 성별 리스트
  - cancelReasonList: 취소 사유 리스트
  - airlineLogoBaseUrl: 항공사 로고 Base URL
- 여정과 예약 관련 데이터
  - journeyInfo: 사용자가 입력한 여정 정보(ALAppJourneyInfo)
  - travelInfo: 여정 조회 결과(ALTravelListModel)
  - firstTravelInfo: 선택된 가는편 항공편 정보
  - secondTravelInfo: 선택된 오는편 항공편 정보(왕복 시)
  - reservationInfo: 예약 요청 정보(ALReservationRequest)
- 데이터 초기화 메서드
  - clear(): 전체 데이터 초기화(기본 데이터, 여정 데이터, 기타 데이터 모두 초기화)
  - clearBaseData(): 공항, 항공사, 코드 리스트 및 로고 URL 초기화
  - clearJourneyData(): 여정 정보 초기화
  - clearEtcData(): 여정 정보, 조회 결과, 선택 항공편, 예약 정보 초기화
