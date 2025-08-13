# Code Review (ttaxi)
## ttaxi.app
### Code
#### ttaxt.app.agree.CallTaxiAgreeActivity
##### 역할
- 택시 호출 서비스 이용 전 약관 동의 화면을 담당하는 Activity
- 진입 시 호출 경로 옵션(taxiRouteOption) 또는 약관 재동의 필요 여부(hasTermsToGetReAgreement) 파라미터를 Intent로 전달받아 상태 초기화
- UI 초기화
  - DataBindingUtil로 activity_calltexiagree 레이아웃 바인딩
  - ProgressDialog 초기화(로딩 표시용)
  - 약관 목록 프래그먼트(TaxiTermListFragment) 표시
- 프래그먼트 관리
  - showTermListFragment(): 약관 재동의 여부를 번들에 담아 TaxiTermListFragment 생성 및 표시
  - setFragment(): 전달받은 BaseFragment를 joinMembership 컨테이너에 add 후 백스택에 추가
- CallTaxiAgreeActivityContract.View 구현
  - showProgress / hideProgress: ProgressDialog 표시 및 숨김
  - popBackStack: 뒤로가기 동작 처리(단순 finish)
  - closeActivity: 액티비티 종료
  - showNetworkError: 네트워크 오류 AlertDialog 표시
  - setTermsList / getTermsList: 약관 리스트 데이터 저장 및 반환
  - goToCallTaxiMain: 호출 경로 옵션 여부에 따라 TaxiIntroActivity로 이동
- 이벤트 처리
  - onKeyDown: 하드웨어 뒤로가기(KEYCODE_BACK) 입력 시 finish
  - onActivityResult: REQUEST_TAXI 결과가 RESULT_OK이면 finish

#### ttaxt.app.agree.CallTaxiAgreeActivityContract
##### 역할
- CallTaxiAgreeActivity의 MVP 구조에서 View와 Presenter 간 Contract 정의
- View 인터페이스
  - setFragment(BaseFragment fragment): 지정된 프래그먼트를 화면에 설정
  - showProgress(): 로딩 UI 표시
  - hideProgress(): 로딩 UI 숨김
  - popBackStack(): 백스택에서 프래그먼트 제거 또는 뒤로가기 처리
  - closeActivity(): 액티비티 종료
  - showNetworkError(): 네트워크 오류 안내 UI 표시
  - setTermsList(List<TermsBundle>): 약관 목록 데이터 설정
  - getTermsList(): 약관 목록 데이터 반환
  - goToCallTaxiMain(): 택시 호출 메인 화면으로 이동
- Presenter 인터페이스
  - onDestroy(): Presenter 자원 해제 및 종료 처리


#### ttaxt.app.agree.TaxiTermListContract
##### 역할
- TaxiTermListFragment 등의 약관 목록 화면에서 View와 Presenter 간 Contract 정의
- View 인터페이스
  - showTerms(List<ResponseTerms.TermsBundle> agreements): 약관 목록(서버 응답 모델) 표시
  - showTerms(AgreementResponse.Data.Agreement[] agreements): 약관 목록(배열 형태) 표시
  - showAgreementTerms(List<ResponseMemberTerms.TermsBundle> agreementTermsList): 회원 약관 목록 표시
  - onResponseFailure(Throwable t): 약관 요청 실패 시 처리
  - setButtonClickable(): 버튼 활성화 처리
  - getTermList(): 현재 선택된 약관 동의 목록 반환(ArrayList<MembershipWrapperTermAgree>)
- Presenter 인터페이스
  - onClickPrev(): 이전 화면 클릭 처리
  - onRequestAgreement(boolean hasNewTerms): 약관 동의 요청 실행(신규 약관 여부 포함)
  - onClickTermAgreed(boolean hasNewTerm): 약관 동의 버튼 클릭 처리(신규 약관 여부 포함)

#### ttaxt.app.agree.TaxiTermListFragment
##### 역할
- 택시 호출 서비스 이용 전 약관 목록을 표시하고 동의 여부를 처리하는 Fragment
- CallTaxiAgreeActivityContract.View를 구현한 Activity와 연결되어 동작
- 주요 UI 구성 요소
  - tvBottom: 다음 단계 버튼(필수 약관 모두 동의 시 활성화)
  - ivAllAgree: 전체 동의 토글 버튼
  - tvAllAgreeDes: 전체 동의 설명 텍스트
  - recyclerWrapper: 약관 항목 리스트를 표시하는 RecyclerView
- presenter(TaxiTermListPresenter)와 연결하여 약관 데이터 요청 및 동의 처리
- 전체 동의/해제 시
  - ivAllAgree의 선택 상태를 토글하고 tvBottom 활성화 상태 변경
  - adapter를 통해 모든 항목의 동의 상태를 일괄 변경(setAllTermItemAgree)
- 약관 표시 메서드
  - showTerms(List<ResponseTerms.TermsBundle>): 서버 응답의 약관 목록을 MembershipWrapperTermAgree 리스트로 변환 후 표시
  - showTerms(AgreementResponse.Data.Agreement[]): 배열 형태 약관 데이터를 리스트로 변환 후 표시
  - showAgreementTerms(List<ResponseMemberTerms.TermsBundle>): 회원 약관 목록을 리스트로 변환 후 표시하며 설명과 버튼명도 UI에 반영
- 약관 항목 정렬: 필수 약관이 상단에 오도록 Comparator로 정렬
- getTermList(): 현재 동의 상태를 포함한 약관 목록 반환
- setButtonClickable(): 다음 버튼 클릭 가능 상태로 설정
- onBackPressed(): 뒤로가기 시 액티비티 종료
- onAttach(): 호스트 Activity가 CallTaxiAgreeActivityContract.View를 구현했는지 확인
- onDestroyView(): corpTermList 초기화로 메모리 누수 방지

#### ttaxt.app.agree.TaxiTermListPresenter
##### 역할
- TaxiTermListContract.Presenter를 구현한 것으로 약관 동의 화면의 비즈니스 로직 담당
- View 참조
  - activityView: CallTaxiAgreeActivityContract.View의 구현
  - fragmentView: TaxiTermListContract.View의 구현
- 약관 데이터 요청
  - onRequestAgreement(hasNewTerms)
    - AppDataRepositories의 TermsAndPoliciesRepository를 통해 회원 약관 목록 요청
    - 성공 시 activityView.setTermsList()로 액티비티에 저장하고 fragmentView.showAgreementTerms()로 UI 반영
- 약관 동의 처리
  - onClickTermAgreed(hasNewTermsToReAgreement):
    - 재동의 필요 시 requestTaxiTermsUpdate() 호출
    - 아니면 requestTMoneyGOCI() 호출
- TMoneyGO CI 요청
  - requestTMoneyGOCI()
    - RxMember.RequestCIInfo("N")로 CI 정보 요청
    - 성공 시 CI 값 저장 후 TGOMemberService() 호출
    - 실패 시 네트워크 오류 표시 및 버튼 활성화
- 약관 동의 서버 반영
  - requestTaxiTermsUpdate()
    - activityView.getTermsList()에서 선택된 약관 리스트 수집
    - RequestTaxiTermsUpdate 객체 생성 후 RxMember.RequestTaxiTermsUpdate() 호출
    - 성공 시 TGoLoginManager.getLinkedServiceList() 호출 후 택시 호출 메인 화면 이동
    - 실패 시 네트워크 오류 표시 및 버튼 활성화
- TGOMemberService 연동
  - TGOMemberService()
    - OndaTaxiApi.TGOMemberService() 호출, 성공 시 Access Token 저장(saveToken) 후 requestTaxiTermsUpdate() 재호출
    - 실패 시 네트워크 오류 표시 및 버튼 활성화
- 기타
  - unsubscribe(): CompositeDisposable 초기화
  - saveToken(): TaxiPref에 Access Token 저장

#### ttaxt.app.historydetail.adapter.TaxiHistoryDetailAdapter
##### 역할
- 택시 결제 내역 상세 화면(TaxiHistoryDetailFragment)에서 결제 수단별 내역을 표시하는 RecyclerView 어댑터
- AbstractRecyclerListAdapter<RMGetPaymentReceipt.PayDivList, TaxiHistoryDetailViewHolder>를 상속받아 데이터 바인딩 처리
- 주요 동작
  - onCreateViewHolder
    - view_holder_history_detail 레이아웃을 inflate하여 TaxiHistoryDetailViewHolder 생성
  - onBindViewHolder
    - 현재 position의 PayDivList 데이터를 가져와 ViewHolder의 bind(item, show) 메서드로 바인딩
  - setShowAmount(show: Boolean)
    - 결제 금액 표시 여부(show 플래그) 설정
    - ViewHolder 바인딩 시 show 값에 따라 금액 표시 여부를 제어

#### ttaxt.app.historydetail.adapter.TaxiHistoryDetailViewHolder
##### 역할
- 택시 결제 내역 상세 화면에서 결제 내역 항목 하나를 바인딩하는 RecyclerView.ViewHolder
- ViewHolderHistoryDetailBinding과 TaxiHistoryDetailFragment를 사용해 데이터와 UI를 연결
- 주요 기능
  - bind(item, show)
    - lifecycleOwner와 fragment를 바인딩에 설정
    - 결제 일자(paymentDivDate): -를 .로 변경하여 표시
    - 결제 수단명(paymentDivTypeNm) 설정
    - 결제 금액(payDivAmount)을 3자리 콤마 형식 + 원으로 변환 후 표시
    - show 값에 따라 금액 표시 여부를 제어
  - convertAmountString(amount)
    - amount 값이 정수인지 확인 후 숫자로 변환
    - DecimalFormat("###,###")으로 3자리 단위 콤마 추가
    - 원 단위를 붙여 문자열 반환

#### ttaxt.app.historydetail.TaxiHistoryDetailActivity
##### 역할
- 택시 호출 내역 상세 화면을 표시하는 Activity
- 특정 호출 ID(callId)와 택시 타입, 하차 여부(isGetOff)를 인자로 받아 상세 내역 표시
- 정적 메서드
  - start(): Context에서 Activity 시작
  - startActivityForResult(): Activity에서 결과를 받으며 시작
- 화면 초기화(onCreate)
  - 인텐트에서 callId, taxiType, isGetOff 값 추출
  - Toolbar 설정
  - ProgressDialog 초기화
  - 하차 상태(CallState.COMPLETE) 저장
  - TaxiHistoryDetailFragment 생성 및 container_fragment에 추가
  - onBackPressedHandler를 fragment로 설정
- onResume
  - 상태바 스타일 설정
- 뒤로가기(onBackPressed)
  - onBackPressedHandler가 처리하면 종료하지 않음
  - Fragment 백스택이 있으면 popBackStack()
  - Task Root이면 isGetOff 여부에 따라 홈 이동 및 TaxiMainActivity 실행
  - 그 외는 super.onBackPressed()
- 메뉴 아이템 선택(onOptionsItemSelected)
  - 홈 버튼(android.R.id.home) 클릭 시 isGetOff 여부에 따라 홈 이동 및 TaxiMainActivity 실행 후 종료
- startHome()
  - CallState.COMPLETE 설정
  - 드라이버 평가 키 여부에 따라 마지막 호출 ID 초기화
  - 출발지, 도착지, 택시 타입, 호출 위치 등의 마지막 정보 초기화
  - RouteSearchDataStore의 현재 경로 null로 초기화

#### ttaxt.app.historydetail.TaxiHistoryDetailFragment
##### 역할
- 택시 호출 내역 상세 화면의 UI 및 사용자 상호작용 처리
- TaxiHistoryDetailViewModel과 DataBinding을 사용하여 데이터 표시 및 이벤트 처리
- 주요 기능
  - newInstance(callID, taxiType, isGetOff): 호출 ID, 택시 타입, 하차 여부를 인자로 Bundle에 담아 Fragment 생성
  - onCreateView
    - FragmentTaxiHistoryDetailBinding을 통해 레이아웃과 바인딩
    - ViewModel 초기화 및 어댑터(TaxiHistoryDetailAdapter) 설정
  - onViewCreated
    - LiveData 옵저빙 설정(observe())
    - 전달받은 인자 로드(loadArguments)
    - 화면 초기화(init) 및 데이터 요청(viewModel.requestHistory)
  - observe()
    - networkState: 네트워크 오류 시 showNetworkError 호출
    - toastMsg: 토스트 메시지 표시
    - showDriverEvaluation: 기사 평가 다이얼로그 표시
    - receivableData: 미납 요금 화면(TaxiUnpaidFeeActivity) 이동 후 액티비티 종료
    - showPayAmount: 금액 표시 여부 어댑터에 전달
  - 즐겨찾기 기능
    - favoritesButton(): 내 호출이면 삭제, 아니면 추가 다이얼로그 표시
    - favoriteAddDialog(): 즐겨찾기 추가 확인 다이얼로그
  - 공유 기능
    - shareButton(): 현재 화면의 영수증 이미지를 캡처하여 공유 인텐트 실행
  - 미납 결제
    - unPaidPaymentButton(): UnpaidChargeActivity 호출
  - 기사 평가
    - showDriverEvaluation(): TaxiDriverEvaluationDialogFragment 호출 및 평가 데이터 전송
  - 기사 호출
    - driverCall(): 기사에게 전화 걸기 확인 다이얼로그
  - 영수증 저장
    - showReceipt(): 저장 권한 체크 후 이미지 저장
    - saveReceipt(): View를 이미지로 저장 (Android Q 이상은 MediaStore 사용, 이하 버전은 FileOutputStream 사용)
    - getImageUri2(): binding.useinfolayout을 Bitmap으로 변환 후 저장, 저장 성공 시 URI 반환
    - getBitmapFromView(view): View를 Bitmap으로 변환
  - 권한 처리
    - checkWritePermission(): 저장소 쓰기 권한 여부 확인
    - requestWritePermission(requestCode): 저장소 쓰기 권한 요청
  - 네트워크 오류 다이얼로그
    - showNetworkError(): AlertDialogFragment로 네트워크 오류 안내
  - 기타
    - onBackPressed(): 항상 false 반환 (Activity에서 처리)
    - onActivityResult(): 미납 결제 성공 시 내역 재요청

#### ttaxt.app.historydetail.TaxiHistoryDetailViewModel
##### 역할
- 택시 호출 내역 상세 화면에서 View와 데이터 계층(API, Pref) 사이를 중개하는 ViewModel
- 결제 영수증 조회, 미수금 처리, 운전자 평가 요청, 즐겨찾기(내 호출) 추가·삭제 기능을 제공
- 네트워크 상태, 토스트 메시지, UI 표시 여부 등을 LiveData<Event>로 관리하여 화면 상태를 실시간으로 반영
- 서버 응답 데이터(RMGetPaymentReceipt)를 가공하여 차량명, 결제 금액, 호출 유형(본인호출과 타인호출) 등에 맞춘 UI 제어 로직 수행
- 호출 ID가 비숫자일 경우 API 호출에 맞게 보정 처리하며 택시 타입에 따른 전화 버튼 표시 여부를 결정

#### ttaxi.app.intro.TaxiIntroActivity
##### 역할
- 택시 서비스 진입 시 초기 화면을 담당하는 Activity
- 진입 유형(main과 route)에 따라 위치 확인, 온다 연동 체크, 결제수단 마이그레이션 후 다음 화면으로 이동
- ViewModel 이벤트를 관찰해 팝업 표시, 결과 반환, 화면 종료 및 액티비티 전환을 수행
- startMain, startRoute 등 다양한 정적 메서드로 main 또는 route 모드 진입 지원
- 인텐트 데이터 추출 후 ViewModel과 Pref에 저장
- 위치 서비스 활성화 및 현재 위치 획득 시 온다 서비스 연동 여부 확인
- 팝업을 통한 결제수단 이관 여부 확인
- main 모드일 경우 TaxiMainActivity, route 모드일 경우 TaxiRouteResultActivity로 이동

#### ttaxi.app.intro.TaxiIntroViewModel
##### 역할
- 택시 온다 진입 로직을 담당하는 ViewModel
- 서버 상태, 연동, 설정 조회 후 화면 전환 여부를 결정하고 Event(LiveData)로 Activity에 알림
- 위치 정보, 진입 모드(main, route), 옵션(대화없이, 과속없이, 내비따라) 값을 보관하고 후속 API 호출에 활용
- 온다 서버 점검, 미수금, 약관 재동의, 미연동 등 예외 상황을 분기 처리
- 결제수단 마이그레이션 팝업 노출 및 사용자 선택에 따른 이관 API 호출
- readConfig로 호출 옵션, 토스트, 배너, 쿠폰 등 환경 설정을 수신해 Pref에 저장
- 연동 여부 확인(requestCallInfoGate) 후 메인 이동 또는 약관 동의 화면으로 유도
- 중복 실행 방지를 위해 Event 래핑으로 단발성 UI 이벤트를 관리

#### ttaxi.app.main.adapter.OnFavoriteButtonItemClickInterface
##### 역할
- 즐겨찾기 버튼 클릭 이벤트를 외부로 전달하기 위한 인터페이스
- 어댑터에서 특정 아이템의 즐겨찾기 버튼이 눌렸을 때 해당 position을 콜백 메서드로 전달
- Activity나 Fragment가 이 인터페이스를 구현해 클릭된 아이템의 즐겨찾기 상태를 변경하거나 관련 동작을 수행하도록 함

#### ttaxi.app.main.adapter.TaxiFavoriteSelectAdapter
##### 역할
- 택시 즐겨찾기 선택 목록을 표시하는 RecyclerView 어댑터
- TaxiFavoriteListItem 데이터를 기반으로 항목 이름, 아이콘, 즐겨찾기 상태를 UI에 반영
- 즐겨찾기 버튼 클릭 시 OnFavoriteButtonItemClickInterface 콜백을 통해 클릭된 position을 외부로 전달
- addItem, addAll 메서드로 데이터 추가 및 갱신 기능 제공
- 항목 접근성을 위해 position과 상태에 따라 contentDescription을 설정

#### ttaxi.app.main.adapter.TaxiSelectPhoneNumberAdapter
##### 역할
- 택시 호출 시 사용할 전화번호 유형을 선택하는 RecyclerView 어댑터
- TaxiSelectPhoneNumberDialog.INPUT_TYPE 목록을 기반으로 항목을 표시하고 선택 상태를 시각적으로 반영
- 선택된 항목은 폰트, 배경색, 글자색, 체크 아이콘 등으로 강조 표시
- 특정 위치(3, 4번)에 해당하는 항목에는 승객 아이콘과 글자 크기 변경 적용
- 항목 클릭 시 OnHandleRouteResultListener 콜백을 통해 선택된 position을 외부로 전달

#### ttaxi.app.main.TaxiMainActivity
##### 역할
- 택시 서비스 메인 화면을 표시하는 Activity
- TaxiMainFragment를 생성하여 메인 UI를 구성하고, 경로 및 주소 관련 데이터(RouteSearchDataStore)를 초기화
- 프로모션 팝업 표시 여부를 인텐트로 전달받아 Fragment 생성 시 반영
- onActivityResult를 Fragment로 전달하여 하위 화면에서 돌아온 결과 처리 지원
- 뒤로가기 시 Firebase Analytics 이벤트(onda_main_back_click) 로그 기록
- 정적 start 메서드로 애니메이션 여부와 프로모션 팝업 표시 여부를 제어하며 화면 진입 가능

#### ttaxi.app.main.TaxiMainFragment
##### 역할
- 메인 지도 화면을 담당하는 BaseMapFragment
- 카카오맵 초기화, 현재 위치 표시 및 주변 택시 마커 주기적 갱신(SearchTaxiTimer)
- 출발과 도착 주소 관찰로 다음 버튼 활성화 및 경로 결과 화면으로 이동(TaxiRouteResultActivity)
- 메뉴 동작(내역, 쿠폰, 요청, 드로어 열기와 닫기) 처리 및 Firebase Analytics 로그 기록
- 대신불러주기(타인 호출) 전화번호 선택 BottomSheet, 직접 입력과 연락처 입력 처리 및 최근 이력 저장
- 집과 회사 즐겨찾기 처리 및 지도 목적지 선택 흐름 연동
- 프로모션과 쿠폰 팝업 노출, 토스트와 네트워크 오류 안내 다이얼로그 표시
- GPS 설정 변경 브로드캐스트 대응, 런타임 권한 요청, 생명주기별 타이머와 리시버 관리

#### ttaxi.app.main.TaxiMainViewModel
##### 역할
- 메인 지도 화면의 상태를 관리하는 ViewModel
- 집과 회사 즐겨찾기 보유 여부, 출발과 도착 주소, 다음 버튼 활성화 상태를 LiveData로 제공
- 토스트 메시지와 주변 택시 목록(taxiList), 멀티콜 이용중 버튼 노출과 카운트 상태를 LiveData로 제공
- 결제 영수증 조회(newGetPaymentReceipt)로 최근 결제 금액과 유형과 기사명 등을 Pref에 저장하고 평가 다이얼로그 트리거 지원
- 기사 평가 전송(tGODriverEvaluationReq) API 호출 및 완료 상태 저장
- 주변 택시 조회(requestSearchTaxi)로 지도에 표시할 택시 리스트 갱신
- 멀티콜 이용 현황(requestCallsInUse) 조회로 이용중 버튼 표시 및 이용중 건수 반영

#### ttaxi.app.multicall.TaxiMultiCallActivity
##### 역할
- 택시 멀티콜 기능을 제공하는 화면의 Activity
- ActivityMultiCallBinding을 사용해 레이아웃과 데이터 바인딩 설정
- 커스텀 액션바에서 뒤로가기 및 이용 내역(웹뷰) 버튼 동작 설정
- TaxiMultiCallFragment를 생성하여 컨테이너에 추가
- onBackPressed에서 프래그먼트 백스택 처리 후 루트 Activity 여부에 따라 종료 또는 기본 동작 수행
- 정적 start 메서드로 외부에서 TaxiMultiCallActivity 실행 지원

#### ttaxi.app.multicall.TaxiMultiCallAdapter
##### 역할
- 멀티콜 이용 내역을 표시하는 RecyclerView 어댑터
- RMCallsInUse.CallInUse 데이터를 TaxiMultiCallItemViewHolder에 바인딩
- 항목 클릭 시 OnMultiCallItemClickListener 콜백을 통해 클릭된 CallInUse 객체를 외부로 전달
- ViewHolder 생성 시 TaxiMultiCallItemViewHolder.create 호출로 뷰 초기화

#### ttaxi.app.multicall.TaxiMultiCallFragment
##### 역할
- 멀티콜 이용 내역 목록 화면을 표시하는 Fragment
- ViewModel을 통해 멀티콜 내역을 조회하고 RecyclerView(TaxiMultiCallAdapter)에 바인딩
- 취소된 호출 내역을 별도로 수집하여 전체 삭제 기능 제공(removeCallInUseAll)
- 개별 항목 클릭 시 상태에 따라 호출 복원 또는 삭제 후 복원 처리(clickTaxiItem)
- 호출 복원 시 호출 정보(callRepairString)를 파싱하여 RouteSearchDataStore와 TaxiPref에 출발지와 도착지, 결제·쿠폰·차량 정보 등을 저장
- 호출 복원 후 TaxiRouteResultActivity로 이동하여 이전 호출 상태를 이어감
- UI 상태(목록, 빈 화면, 삭제 버튼 표시) 제어 및 취소 리스트 초기화 로직 포함

#### ttaxi.app.multicall.TaxiMultiCallItemViewHolder
##### 역할
- 멀티콜 이용 내역의 개별 항목 UI를 표시하고 데이터와 바인딩하는 ViewHolder
- RMCallsInUse.CallInUse 객체의 택시 타입, 차량 번호, 출발·도착지, 결제 타입, 호출 상태, 불러주기 대상 정보를 뷰에 세팅
- 결제 타입이 현장일 경우 텍스트 색상을 강조(#fa6a46)로 변경
- 불러주기 대상이 본인인지 타인인지 구분하고 전화번호 형태일 경우 하이픈을 추가해 표시
- 항목 클릭 시 OnMultiCallItemClickListener 콜백을 통해 클릭된 데이터 전달

#### ttaxi.app.multicall.TaxiMultiCallViewModel
##### 역할
- 멀티콜 이용 내역 상태를 관리하는 ViewModel
- callsInUse API 호출로 멀티콜 목록을 조회하여 LiveData로 제공
- 응답에 포함된 프로모션 정보(promotionInfo)를 보관해 후속 화면에서 활용
- 실패 콜백에서는 별도 처리 없이 무시(필요 시 확장 가능)

#### ttaxi.app.myfavorite.adapter.TaxiMyCallAdapter
##### 역할
- 즐겨찾기 택시 경로(즐겨 찾는 호출 내역)를 RecyclerView로 표시하는 어댑터
- 시작과 종료 지점, 별칭, 택시 종류, 옵션명 등 경로 정보를 UI에 바인딩
- 고정(Pin) 여부, 삭제 버튼 등의 UI 상태를 제어
- ViewHolder 내부에서 항목 클릭, 고정 변경, 삭제, 설정 다이얼로그 호출 등의 이벤트를 NotifyFavRouteItemChanged 인터페이스로 전달

#### ttaxi.app.myfavorite.MyCallContract
##### 역할
- 내 호출(My Call) 화면의 MVP 구조에서 View와 Presenter 간 Contract 정의
- View: 화면(UI)에서 필요한 동작과 데이터를 표시하는 메서드 정의 (토스트 메시지, 프로그레스 표시와 숨김, 빈 화면과 네트워크 오류 표시, 호출 목록 전달, 다른 화면 이동, 토큰 확인)
- Presenter: UI 이벤트 처리 및 비즈니스 로직 수행 메서드 정의 (고정 상태 변경, 출발과 도착 지점 설정, 호출 삭제, 닉네임 변경, 정렬 및 목록 요청, 서버 데이터 조회, ONDA 등록 여부 확인)

#### ttaxi.app.myfavorite.MyCallFragment
##### 역할
- 내 호출 목록 화면을 담당하는 Fragment로 MVP의 View 구현
- Presenter와 연동해 생명주기(onCreate과 onResume)와 데이터 로딩 흐름 제어
- RecyclerView 초기화 및 TaxiMyCallAdapter 연결, 정렬바 표시 제어
- 빈 목록과 네트워크 오류와 로딩 등 상태 UI 전환
- 스와이프 애니메이션에 따른 고정과 삭제 액션 UI 처리 및 이전 항목 상태 복구
- 설정 시트(TaxiSettingBottomSheetDialog)로 고정, 닉네임 변경, 삭제 동작 연계
- 항목 클릭 시 출발·도착지 지정 후 라우팅 실행(TaxiIntroActivity 또는 TaxiRouteResultActivity)
- 액세스 토큰 체크 후 경고 다이얼로그 또는 목록 조회 트리거

#### ttaxi.app.myfavorite.MyCallPresenter
##### 역할
- MyCallContract.Presenter를 구현하여 내 호출(My Call) 기능의 비즈니스 로직을 처리
- 즐겨찾기 호출 목록 조회, 추가, 삭제, 수정, 정렬 등의 동작을 수행하고 결과를 View에 전달
- OndaTaxiApi를 이용해 서버와 통신하며 호출 데이터 갱신 및 삭제 요청 처리
- 출발지·도착지 정보를 RouteSearchDataStore에 저장하여 다음 화면(TaxiIntroActivity, TaxiRouteResultActivity 등)으로 라우팅
- 고정(Pin) 상태 변경 시 목록 재정렬 및 최대 고정 개수 제한 로직 포함
- 온다 서비스 가입 여부를 판단하고 토큰 체크를 통한 목록 조회 트리거
- Presenter 생명주기 메서드에서 로그를 기록해 상태 추적

#### ttaxi.app.myfavorite.TaxiMyCallActivity
##### 역할
- 내 호출(TaxiMyCall) 화면의 Activity로, MyCallFragment를 포함하여 호출 목록 UI를 구성
- 툴바 설정 및 뒤로가기 네비게이션 버튼 처리
- 오른쪽 상단 도움말 버튼 클릭 시 FavoriteRouteGuideDialogFragment 표시
- onKeyDown과 onBackPressed를 재정의하여 뒤로가기 시 TaxiMainActivity로 이동
- MyCallFragment를 초기화하며 OnBackPressedHandler를 통해 뒤로가기 동작을 위임 가능