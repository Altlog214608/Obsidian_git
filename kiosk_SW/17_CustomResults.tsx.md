- 파일: CustomResults.tsx — 한줄 요약  
  결과(추천) 화면을 렌더링하고, 라디얼 조절기(SimpleRadialAdjuster)로 향 파라미터를 조정하며 피드백/구매 모달을 관리하는 페이지입니다.

1) 최상단 import / 타입
- React 훅(useState), 라우터 훅(useLocation, useNavigate), 아이콘, 섹션 컴포넌트(SimpleRadialAdjuster, FeedbackModal, PurchaseModal, FeedbackButtons)와 타입(ScentParameter), theme 훅을 가져옵니다.
- SelectionData 인터페이스: location.state로 전달되는 사용자 선택 데이터 구조(배열들) 정의.

2) 위치/초기 데이터 로딩
- const location = useLocation(), navigate = useNavigate()  
  - 이전 페이지에서 navigate('/custom-results', { state: { selections, initialParameters } })로 전달된 데이터를 읽어옵니다.
- selections = (location.state?.selections as SelectionData) || null  
  - 전달된 selections가 없으면 null로 처리.
- initialParameters = (location.state?.initialParameters as ScentParameter[] | undefined) || undefined  
  - 초기 향 파라미터(옵션)을 복사해서 내부 상태 초기화에 사용.

3) 내부 상태들
- showFeedback, showFeedbackModal, showPurchaseModal: UI 모달/버튼 표시 플래그.
- adjustedParameters, currentParameters: ScentParameter[] 상태. initialParameters가 있으면 복사(...initialParameters)로 초기화(참조가 아닌 복사).

4) early redirect (selections 검사)
- if (!selections) { navigate('/'); return <div>Loading...</div> }  
  - location.state가 없을 경우(직접 진입 등) 홈으로 이동. (참고: render 중 직접 navigate 호출; 동작하지만 useEffect로 처리하면 더 명시적)

5) 핸들러
- handleAdjust(newParameters): adjustedParameters 및 currentParameters 업데이트 — "조정된 값 저장(확정)" 용도.
- handleParametersChange(parameters): 현재 파라미터 실시간 업데이트(조절 중) — SimpleRadialAdjuster의 onValuesChange에 바인딩.

6) JSX 구조(렌더링)
- 최상위 컨테이너: 그라데이션 배경 + blur된 흰 배경 카드(글래스모피즘 스타일).
- 헤더: 가운데 제목("추천 결과")와 오른쪽 X 버튼(aria-label="닫기") — 클릭 시 navigate('/')로 홈으로 복귀.
- 메인 컨텐츠:
  - 중앙에 SimpleRadialAdjuster 컴포넌트 배치.
    - props 요약: hidePresets, hideIntensitySlider, initialParameters, onTryItNowClick (Try It Now 클릭 시 showFeedback true), onValuesChange (실시간 파라미터 업데이트).
  - FeedbackButtons: Try It Now 이후에 나타나는 버튼들(조정 / 만족 → 구매 모달).
- 하단 네비게이션:
  - "다시 해보기" 버튼: navigate('/custom')로 커스텀 흐름으로 복귀.
  - "샘플 구매하기" 버튼: PurchaseModal 토글(추후 결제 로직 실행).
- 모달들:
  - FeedbackModal: currentParameters가 있을 때만 렌더, adjust 콜백(handleAdjust).
  - PurchaseModal: 항상 렌더(토글로 숨김/표시). 구매 완료 후 모달 닫음(콘솔 TODO).

7) 동작 흐름 요약
- 페이지 진입 시 selections가 없으면 홈으로 리다이렉트.
- SimpleRadialAdjuster로 값을 만지면 onValuesChange → currentParameters 업데이트.
- "Try It Now" 클릭 시 FeedbackButtons가 나타남.
- FeedbackButtons에서 "Adjust" → FeedbackModal 열림 → Adjust confirm → adjustedParameters 갱신.
- 만족 → PurchaseModal 열림 → 결제 로직(TODO) 호출 가능.

8) 주의점·권장 개선 (짧게)
- navigate('/')를 렌더 경로에서 직접 호출하는 대신 useEffect로 처리하면 안전합니다(렌더 시 사이드 이펙트 최소화).  
- setTimeout/비동기 타이머가 이 파일엔 없지만, 모달/조정 흐름에서 타이머가 있다면 cleanup 필요.  
- initialParameters 복사는 [...initialParameters]로 안전하게 하고 있어 좋음.  
- 접근성: 버튼(특히 "샘플 구매하기")에 aria-label 추가·포커스 관리(모달에 focus trap) 권장.  
- 성능: SimpleRadialAdjuster의 onValuesChange 빈번 호출 시 상위에서 불필요한 리렌더를 유발할 수 있으니 debouce 또는 memo화 고려.

9) 짧은 개선 코드(렌더 중 navigate → useEffect로 이동) 예
````typescript
// ...existing code...
const selections = (location.state?.selections as SelectionData) || null

useEffect(() => {
  if (!selections) navigate('/')
}, [selections, navigate])

if (!selections) return <div>Loading...</div>
// ...existing code...
````
끝. 추가로 특정 컴포넌트(SimpleRadialAdjuster, FeedbackModal 등) 내부를 같이 보면 더 구체적 개선안을 줄 수 있습니다.