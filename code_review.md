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
|deptAirport/arrAirport|출발/도착 공항 코드|
|departureDate/arrivalDate|출발/도착 일시|
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
- 항공권 결제 요청 시 필요한 여정, 탑승객, 카드, 운임 정보 등을 서버에 전달하기 위한 요청
- 성인/아동 운임, 프로모션 할인 금액, 카드정보, 결제 수단, 이지페이 정보 등 결제에 필요한 모든 데이터를 포함

|필드명|설명|
|:-----:|:---:|
|orderKey|주문번호|
|journeys(CardInfo)|각 여정별 결제 수단|
|pnrInfos|여정 예약 정보|
|passengerInfos|탑승객 정보|
|totalAirFare, totalFare|운임 총액 (세금 포함 여부에 따라 구분)|
|totalPromotionDiscountAmount 외|다양한 할인 금액 합계|
|CardInfo|카드 결제 수단 및 관련 정보, 결제 금액 등|
|FreeBaggage|여정별 무료 수하물 서비스 정보|
|EasyPayInfo|이지페이 결제에 필요한 인증값 (대한항공 등에서 사용됨)|

#### data.entities.request.ALPostFareRuleRequest
##### 역할
- 결제 이후 적용되는 운임 규칙을 조회할 떄 사용
- orderKey와 detailOrderId를 서버에 전달하여 해당 예약 건에 대한 후속 운임 규칙을 확인

#### data.enitities.request.ALPreFareRuleRequest
##### 역할
- 항공권 예약 이전 단계에서 운임 규칙을 조회할 때 사용하는 요청 모델
- 여정 키, 운임 키, PAIR 키, 항공사 코드, 출발/도착 공항, 출발일자 등 여정의 식별 정보를 기반으로 서버에 운임 규칙을 요청함
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
- 서비스 구분 코드(advrSvcDvsCd)와 노출 위치 코드(advrLocDvsCd)를 서버에 전달하여 홈 화면 등 특정 위치에 띄울 롤링 배너 정보를 서버에서 가져올 때 사용

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
- 입력된 카드 정보를 기준으로 카드에 대한 이름, 승인 기관 코드 명, 체크/법인 여부, 발급사 등을 서버로부터 받아오는 데 사용

#### data.entities.response.ALFareDiscountRuleResponse
##### 역할
- 신분 할인 운임 정보 조회
-  가는편(goIdFareRules)과 오는편(backIdFareRules) 각각에 대해 성인(adt)/소아(chd)/유아(inf)(paxType) 별 할인 정보(FareDiscountRule)를 포함함
- 할인 명칭, 신분 할인 코드, 운임/세금 및 할인율 등 정보를 포함
- 특정 여정에 대해 탑승객 신분에 따라 적용 가능한 운임 할인 규칙들을 서버에서 받아오는 데 사용

#### data.entities.response.ALFareRuleResponse
##### 역할
- 운임 규정 정보 응답
- 항공권 예매 시 적용되는 다양한 운임 규정 정보를 담는 모델
- 규정명, 설명, 정렬 순서, 유형 정보를 포함

#### data.entities.response.ALPaymentMethodResponse
##### 역할
- 결제수단 정보 조회
- 앱에서 결제수단 화면에 출력할 문구, 노출 여부 등을 서버에서 받아오는 역할
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
- 취소 요청 결과와 여정 상태 등을 클라이언트에 전달
- 예약 취소가 성공했는지, 추가 취소 접수가 필요한지, 여정의 상태가 어떤지 등을 확인
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
- 예약/주문 변호, 예약자/탑승객, 항공사 코드/명, 결제 정보, 항공편 정보 등 모든 예약 결과의 종합 정보 제공

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
|pairKey|여정 조합 키 (왕복 등 구분용)|
|cabinClass, totalFlightTime|좌석 등급 및 총 비행시간|
|freessrName, freessrValue|무료 수화물 정보|
|aprvDtm, aprvInfo|결제 일시 및 정보|
|cancDtm, cancFee, cancTotalAmt|취소 일시, 수수료, 총 환불 금액|
|shareAirlineName|공동 운항 항공사 명|
|PassengerInfo|탑승객 인덱스, 타입, 성별, 생년월일, 요금 정보, 환불 정보 포함|
|FareInfo|해당 여정에 대한 운임, 세금, 유류세, 수수료 등 상세 정보|
|Refund|환불 금액 정보|
|Name|탑승객 이름 구조(first, middle, last)|

#### data.entities.response.ALReservationCompleteListResponse
##### 역할
- 항공 예약 시스템에서 사용자의 예약 완료 목록 조회 시 사용되는 응답
- 예약이 완료된 항공권들의 여정 목록 정보를 조회

#### data.entities.response.ALReservationResponse
##### 역할
- 항공 예약 완료 후, 주문번호 및 여정, 탑승객, 결제금액, 할인 정보 등을 포함한 전체 예약 정보를 서버로부터 응답 받음
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
- 대한항공 등 특정 항공사의 간편결제 연동 시 사용되는 여정별 결제 정보 관리

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
- 고정 핀 여부, 별명, 생성시간 등 메타 정보 포함
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
|PaymentCard|개별 카드 정보 (카드번호, 보안데이터, 유효기간, 카드명, 카드사 등)|

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
- 후속 처리(발권, 취소 등)를 위한 참조 데이터

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
- 탑승객, 운임, 일정, 결제 등 예약과 관련된 모든 세부 정보를 구조화

##### 변수 및 함수
|변수|범위|내용|특징|
|:-----:|:---:|:---:|:---:|
|journeySequenceNo|val|여정 일련번호|nullable String|
|journeyStatusCode|val|여정 상태 코드|HL/OK/XX/QQ|
|pnrNumber|val|예약 번호|nullable String|
|orderKey|val|주문 번호|nullable String|
|passengerInfos|val|탑승객 정보 목록|List<PassengerInfo>, 기본값 빈 리스트|
|airline ~ shareAirlineName|val|항공편 정보|항공사, 공항, 일시, 좌석 등|

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
- 탑승객 수 및 좌석 등급 등 핵심 정보 요약 표시

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
|Journey|여정 상세 정보 (항공사, 공항, 일시, 세그먼트, PNR 등)|
|Segment|항공편 구간 정보 (예약등급, 좌석등급, 운행사, 비행시간 등)|
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
- PNR, 여정 키 등 항공사 연동을 위한 참조 데이터 관리

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
- 서비스 이용약관 및 개인정보처리방침 등의 약관 정보를 관리

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
|Travel|개별 항공편 정보 (항공사, 공항, 일시, 운임, 좌석, 프로모션 등)|
|FreeBaggage|무료 수하물 서비스 정보 (서비스명, 코드, 값, 대상, 단위)|
|PaxTypeFares|탑승객 타입별 운임 정보 (기본요금, 세금, 할인 등)|

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
- 프로모션, 배너, 카드, 약관 등 다양한 공통 데이터를 통합 관리

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
- 예약 목록, 상세 정보, 동기화, 이용 내역 등을 통합 관리

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
- 브라우저 열기, 클립보드 복사, 키보드 숨기기, 앱 설치 확인 및 마켓 이동, 배너 클릭 시 동작 처리 등 구현

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
    - 객체 내 상세 속성(airportArea, airportCode 등)을 활용


#### ext.StringExt
##### 역할
- String 및 Int 타입에 유용한 확장 함수들을 정의
- 항공사, 공항, 국가 등 코드를 이름으로 매핑
- 이메일, 이름, 생년월일 등 유효성 검사
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
- View, EditText, ViewPager2, RecyclerView 등 UI 컴포넌트에 대한 공통 확장 함수 모음
- 클릭 이벤트 처리, 포커스에 따른 UI 상태 변경, 페이지 애니메이션, 중앙 정렬 등 UI 로직 간소화


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
- 최근 검색 이력 자동 저장 기능 제공 (편도/다구간 등)
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
#### network.service.ALCommonService
##### 역할
- 공통 정보(광고, 배너, 카드, 결제 수단, 약관)를 조회하는 Retrofit 인터페이스
- 모든 API는 POST 방식, RxJava의 Single<Response<...>> 반환
- 공통 응답 구조인 ALBaseResponse<T>를 통해 일관된 데이터 수신

#### 메서드 목록

| 메서드명 | API 엔드포인트 | Request 타입 | Response 데이터 타입 | 설명 |
|:--|:--|:--|:--|:--|
| inqrAdvertStupInfo | api/mui/v2/inqrAdvrPupStupInf | ALPromotionBannerRequest | ALPromotionBannerResponse | 광고 배너 설정 정보 조회 |
| inqrAdvrPupStupInf | api/mui/v2/inqrAdvrPupStupInf | ALPromotionPopupRequest | ALPromotionPopupResponse | 광고 팝업 설정 정보 조회 |
| inqrMoappBnrList | api/mui/v2/inqrAdvrPupStupInf | ALRollingBannerRequest | ALRollingBannerResponse | 롤링 배너 정보 조회 |
| inqrPymMnsListAtPym | api/pym/v2/inqrPymMnsListAtPym | ALPaymentMethodRequest | ALPaymentMethodResponse | 결제 시 사용 가능한 결제 수단 목록 조회 |
| inqrPrpmAutPymStplAgrmYN | api/mbrs/v2/inqrStplList | ALTermsListRequest | ALTermsListResponse | 이용약관 등 약관 목록 조회 |
| inqrCardBinInfo | api/pym/v2/inqrCardBinInfo | ALCardInfoRequest | ALCardInfoResponse | 카드 BIN 정보 조회 |

#### 특징
- 동일한 API 경로를 사용하되 Request 객체의 타입으로 기능을 구분
- 서버 통신 실패/성공 여부를 포함한 ALBaseResponse<T> 래핑 구조
- 모든 응답은 Retrofit2의 Response<> 래핑 + RxJava의 Single로 감싸져 있음
- ViewModel 또는 Repository 단에서 처리 용이
