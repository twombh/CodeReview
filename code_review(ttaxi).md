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

