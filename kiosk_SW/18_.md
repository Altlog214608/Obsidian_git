GitHub Copilot

요약 한 문장  
- Top3Results.tsx는 사용자가 선택한 항목(selections)을 받아 "추천 향 파라미터"를 생성(generateRecommendation)하고 상위 3개 향을 화면에 보여주며, 상세 조절 페이지로 이동하거나 구매 모달을 여는 UI를 제공하는 결과 페이지입니다.

파일 구조별 설명 (블록 단위, 간단·명확)

1) import / 타입 선언
- useState, useMemo, useLocation, useNavigate 등 React/라우터 훅을 불러옵니다.  
- ScentParameter 타입, SCENT_PARAMETERS 상수, PurchaseModal, useTheme, perfumeImage 등 UI/데이터 의존성을 임포트합니다.
- SelectionData 인터페이스: 이 페이지가 기대하는 selections 객체의 형태(gender, age, mbti, fashionStyle, color, season, scentType, situation, concentration).

2) SCENT_NAMES / SCENT_NAMES_EN
- 향조 id → 한글/영문 이름 매핑 객체. 화면에 보여줄 라벨(한글, 영문 필기체)을 여기서 해결합니다.

3) generateRecommendation(selections)
- 목적: selections를 받아 ScentParameter[](각 향조 id, 기본 메타 포함)에 값(value)을 할당해 "추천 파라미터"를 생성.
- 동작:
  1. baseScentType(사용자 선택의 첫 항목) / season / situation을 결정(없으면 기본값).
  2. SCENT_PARAMETERS를 복사하여 모든 value를 50으로 초기화.
  3. scentTypeMap, seasonMap, situationMap: 각 선택값에 매핑되는 향조 id 리스트 정의(도메인 룰).
  4. primary(향조 타입) 항목들에 +40, season 항목들에 +30, situation 항목들에 +30을 더해 최종 value를 계산(최대 100 cap).
- 특징: 단순 가중치 기반 규칙 엔진 — 직관적이고 디버깅 쉬움. (더 복잡한 조합 논리는 확장 가능)

4) Top3Results 컴포넌트
- selections 결정:
  - props로 전달된 selections가 우선, 없으면 location.state에서 읽음, 없으면 null 처리.
- recommendation: useMemo로 generateRecommendation 실행(입력 selections 변경 시만 재계산).
- top3Scents: recommendation을 value 내림차순 정렬 후 상위 3개를 골라 SCENT_NAMES(한글)과 SCENT_NAMES_EN(영문)으로 라벨을 붙임.
- 즉시 리다이렉트(주의): selections 또는 recommendation이 없으면 navigate('/')를 호출하고 Loading을 반환. (렌더 도중 navigate 호출 — 권장 개선 아래 참조)
- handleAdjustClick:
  - SCENT_PARAMETERS 기준 순서에 맞추어 recommendation 값을 매칭한 sortedParameters를 만들고 navigate('/results', { state: { selections, initialParameters: sortedParameters } })로 상세조절 페이지로 이동. (SCENT_PARAMETERS 순서·angle 의존성을 유지해야 함)
- handlePurchaseClick:
  - showPurchaseModal 토글로 구매 모달 표시.

5) JSX 구조(주요 영역)
- 최상위 컨테이너: 화면 전체를 채우는 그라디언트 배경.
- Main Card:
  - 헤더: 타이틀("추천 결과")와 닫기(X) 버튼(클릭 시 홈으로 navigate).
  - 중앙 영역:
    - 설명 문구
    - 향수 이미지(perfumeImage)와 그 위에 paper-style label(작은 카드) — 그 카드 안에 상위 3개 향의 영문 필기체 텍스트(크기/가중치로 1/2/3위 강조).
  - 향 세부조절 버튼: handleAdjustClick 바인딩, hover 효과 및 theme.palette 기반 그라디언트 사용.
  - 네비게이션 버튼: "처음으로", "샘플 구매하기" (구매는 모달 오픈).
- PurchaseModal: showPurchaseModal 상태로 열림/닫힘, onPurchase에서 결제 로직 TODO.

6) 실무에서 주의할 점 / 개선 권장 (우선순위)
- 렌더 중 navigate 호출
  - 현재 selections 없을 때 render 시 바로 navigate('/')를 호출합니다. 렌더 내 사이드 이펙트이므로 useEffect로 옮기는 것이 안전합니다(React 경고/불필요한 re-render 방지).
- SCENT_PARAMETERS 의존성
  - handleAdjustClick에서 SCENT_PARAMETERS의 순서(또는 angle 속성)에 의존해 초기 파라미터 정렬을 만듭니다. SCENT_PARAMETERS 정의가 바뀌면 매칭 로직이 깨질 수 있으므로 주의.
- performance
  - generateRecommendation은 가볍지만, 복잡해지면 useMemo 의존성(selections) 관리와 캐싱 전략 필요.
- 접근성
  - 이미지 alt 있음(Perfume). 버튼들에 aria-label 보강(특히 "향 세부조절" 버튼은 시멘틱 텍스트로 충분하지만 모달 트랩·포커스 관리 필요).
- 스타일/테마
  - currentTheme.palette.*를 사용해 inline style에 `${palette}20`같은 방식으로 알파값을 붙임 — palette가 항상 #RRGGBB 포맷인지 확인 필요(포맷 예외 처리 권장).
- 결제 흐름
  - PurchaseModal onPurchase은 TODO — 실제 결제 로직/서버 연동·에러 핸들링·결제 성공/실패 UI 필요.

7) 짧은 개선 코드 (렌더 내 navigate → useEffect로 이동)
- 권장: selections 검사 후 즉시 navigate 호출하는 대신 useEffect로 처리.
- 변경 예시:

````typescript
// ...existing code...
const selections = propsSelections || (location.state?.selections as SelectionData) || null
const [showPurchaseModal, setShowPurchaseModal] = useState(false)

// 기존: if (!selections || !recommendation) { navigate('/'); return <div>Loading...</div> }
// 권장:
useEffect(() => {
  if (!selections) {
    navigate('/', { replace: true })
  }
}, [selections, navigate])

if (!selections || !recommendation) {
  return <div>Loading...</div>
}
// ...existing code...
````

간단 요약 / 권장 우선 작업
- 핵심: generateRecommendation 로직은 이해/확장하기 쉬운 규칙 기반.  
- 먼저 할 일: 렌더 중 navigate를 useEffect로 이동, SCENT_PARAMETERS 순서 일관성 확인, PurchaseModal 결제 흐름 설계(백엔드 연동) 및 접근성(모달 focus-trap) 보강.

더 필요하면
- generateRecommendation의 가중치/매핑을 바꾸는 방법(예: 가중치 조정 UI, 머신러닝 연동) 예시를 만들어 드리거나, Top3Results에서 보여주는 라벨·스타일을 컴포넌트화한 패치 코드를 바로 만들어 드립니다. 어느 쪽으로 도와드릴까요?