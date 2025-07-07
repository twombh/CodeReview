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
























