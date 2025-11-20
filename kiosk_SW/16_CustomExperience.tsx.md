GitHub Copilot

아래는 첨부하신 CustomExperience.tsx 파일을 블록/기능 단위로 쉽게 풀이한 설명입니다. 코드 흐름, 각 상태·핸들러의 역할, 주의점(잠재 이슈)와 개선 권장사항을 함께 정리했습니다.

파일 상단 — import
- React 훅과 라우터 훅: `useState`, `useEffect`, `useNavigate`  
  - \
  - 페이지 간 네비게이션과 상태·사이드 이펙트 관리에 사용.


- 여러 UI/단계 컴포넌트 임포트:
  - `StepContainer`, `GenderAgeSelector`, `FashionStyleGallery`, `ColorWheelPicker`, `SeasonSelector`, `NavigationButtons`, `FanCardGallery`, `LoadingScreen`
  - 각 단계별 UI를 분리한 컴포넌트들.
- 테마 훅: `useTheme()` — 현재 테마(색상 팔레트 등)를 가져옴.
- 이미지 임포트(여러 상황·향 유형 이미지) — 각 선택지에 이미지를 보여주기 위해 사용.

타입 정의
- `type SelectionData = { ... }`
  - 사용자가 선택하는 모든 항목을 구조화한 타입. 각 단계의 선택 상태를 한 객체로 관리하기 위해 사용:
    - `gender`, `age`, `fashionStyle`, `color`, `season`, `scentType`, `situation`, `concentration` 등 (배열 또는 문자열 형태).

상수: STEPS
- `STEPS` 배열은 각 UI 단계 순서와 `id`, `title`, `tab` 메타데이터를 정의.
  - 예: 첫 단계 `gender` → '먼저 기본 프로필을...' 등.
  - `id` 값은 상태(SelectionData) 키와 대응되어 현재 단계의 선택값을 가져오도록 설계됨.

상수: SITUATION_ITEMS
- 상황(사용 시나리오) 목록을 객체 배열로 정의. 각 항목: `id`, `title`, `description`, `image`.
- `FanCardGallery`에서 중간 아이템이 중심으로 올 때 표시할 데이터.

SituationSelector 컴포넌트
- 목적: `situation` 단계에서 중앙에 보이는 상황 선택 갤러리를 보여주고 선택값을 상위에 전달.
- Props:
  - `selections`: 현재 전체 선택 상태(`SelectionData`)
  - `setSelections`: 상태 갱신 함수(상위 컴포넌트의 setState)
- 내부 상태:
  - `currentIndex`: FanCardGallery에서 중앙에 위치한 아이템 인덱스(초기값은 selections.situation에 기반하거나 중앙 인덱스).
- 핸들러:
  - `handleCenterItemChange(index)`:
    - `currentIndex` 업데이트, `setSelections`로 `situation`을 `[SITUATION_ITEMS[index].id]`로 설정.
- 초기화 useEffect:
  - `selections.situation`이 비어 있고 `currentIndex === 0`이면 중앙 아이템(중간 인덱스)를 기본 선택으로 설정.
  - 의도: 아무것도 선택되지 않았을 경우 초깃값 자동 부여.
- 렌더링:
  - `FanCardGallery`에 `items`와 `onCenterItemChange` 전달.
  - 아래에 현재 선택된 상황의 `title`을 보여주는 카드(테마 색상 기반 dot 포함).

CustomExperience 컴포넌트 (메인)
- 목적: 사용자가 단계별로 입력(프로필 → 스타일 → 색상 → 계절 → 향 타입 → 상황)하면 마지막에 결과(Top3)로 이동시키는 흐름을 처리.
- 훅/상태:
  - `navigate` (react-router) — 결과 페이지로 이동할 때 사용.
  - `currentStep` — 현재 단계 인덱스(0부터).
  - `isLoading` — 최종 처리(모의 AI 처리) 중 로딩 스크린 표시 여부.
  - `selections` — `SelectionData` 타입의 상태, 초기값은 빈 배열/빈문자열로 초기화.
- 주석 처리된 `handleSelect` (미사용): 일반적 다중 선택 토글 로직 예시(현재는 각 단계에서 개별 핸들러를 사용).
- `handleNext`:
  - 현재 단계가 마지막이 아니면 `currentStep` 증가.
  - 마지막 단계면:
    - `isLoading = true`
    - 콘솔에 selections 출력
    - `setTimeout`으로 5초 대기(모의 AI 처리) 후 `navigate('/top3-results', { state: { selections } })`
- `handleBack`:
  - `currentStep > 0` 이면 이전 단계로, 아니면 루트('/')로 이동.
- `currentStepKey` / `currentSelections` 계산:
  - 현재 단계의 `id`를 `SelectionData` 키로 간주해 현 단계의 선택값을 참조.
- `canProceed` 결정 로직:
  - 첫 단계(`gender`)는 성별, 연령, MBTI(4글자, '_' 포함 금지) 조건을 모두 만족해야 진행 가능.
  - 그 외 단계는 `currentSelections`가 배열이면 길이 > 0이면 진행 가능.
- 로딩 상태 처리:
  - `isLoading`이면 `<LoadingScreen />`을 바로 렌더링(다음 화면으로 navigate 될 때까지).

렌더링(주요 구조)
- 최상위: `<StepContainer>` 컴포넌트로 단계 타이틀, 현재 단계, 전체 단계 수, 내비게이션(뒤/다음 버튼) 등을 전달.
  - `navigationButtons` prop에 `<NavigationButtons>` 전달(핸들러와 `canProceed`, `isLastStep` 전달).
- 각 단계별 조건부 렌더링(`currentStep === X`):
  1. step 0 — `<GenderAgeSelector>`: 성별/연령/MBTI 입력. props로 선택값과 변경 핸들러(onGenderSelect/onAgeSelect/onMBTISelect)를 전달.
  2. step 1 — `<FashionStyleGallery>`: 패션 스타일 선택. `onStyleChange`로 단일 선택 반영.
  3. step 2 — `<ColorWheelPicker>`: 색상 선택(프리셋 옵션 목록 전달). `onChange`는 선택 항목의 id를 `selections.color`에 저장.
  4. step 3 — `<SeasonSelector>`: 계절 선택. `onSeasonSelect`으로 단일 선택.
  5. step 4 — `<ColorWheelPicker>`(다른 옵션 집합): 향조 선택(이미지/설명 포함). `onChange`로 `scentType` 설정.
  6. step 5 — `<SituationSelector>`: 위에서 정의한 상황 선택 컴포넌트 사용.

주요 특징 및 의도
- 단계별로 선택을 하나의 `selections` 객체로 집계 → 마지막에 그 상태를 결과 페이지로 전달.
- UX: 각 단계마다 `StepContainer`와 `NavigationButtons`로 일관된 네비게이션 레이아웃을 유지.
- 이미지와 옵션은 로컬 assets로 불러와서 UI에 풍부한 비주얼을 제공.

주의점(잠재 이슈) 및 개선 권장
1. setTimeout 정리
   - `handleNext`에서 최종 단계의 `setTimeout`을 사용. 컴포넌트가 언마운트되면 타임아웃이 남아 있을 수 있음(메모리/네비게이션 충돌 가능).
   - 개선: `useRef`로 타이머 ID 보관하고 `useEffect` cleanup에서 `clearTimeout` 호출 권장.
2. FanCardGallery / 외부 라이브러리 의존
   - 이전 오류(`Failed to resolve import "swiper/react"`, `"ogl"`)가 있었음. 해당 컴포넌트들이 의존성을 요구하므로 `npm install swiper ogl` 등 설치 확인 필요.
   - 설치 없이 컴포넌트를 참조하면 dev 서버에서 에러 발생.
3. 접근성(ARIA / 키보드)
   - `Link` / 각각 선택 요소에 대한 `aria-label`, 키보드 포커스/탭 순서 검토 필요.
4. 상태 관리(성능)
   - `selections` 객체를 업데이트할 때마다 전체 컴포넌트가 리렌더될 수 있음. 필요 시 단계별 컴포넌트는 `memo` 처리하거나 상태를 단계 컴포넌트 내부로 국소화해 불필요한 렌더링 방지.
5. 입력 유효성
   - MBTI 검증: 현재 `selections.mbti.length === 4 && !selections.mbti.includes('_')` 로 검사. 이 검증 로직이 UI에서 올바르게 enforced 되는지(유저 입력 방법) 확인 필요.
6. UX: 로딩 처리
   - 모의 처리(5초) 동안 네비게이션 방지, 취소 옵션 제공 검토. `isLoading`이 true인 동안 뒤로가기 버튼 동작 제한을 고려.
7. Assets / i18n
   - 로컬 한글 파일명(예: `데이트.jpg`) 사용 시 빌드/배포 환경에서 인코딩 문제 발생 가능성 — 빌드 결과에서 파일 경로 확인 권장.
8. TypeScript 안전성
   - `currentSelections`가 `string[] | string` 등 다양한 타입일 수 있으니 타입 좁히기(guard)로 안정성 보장.

요약
- 이 파일은 단계형(멀티스텝) 사용자 입력 플로우를 관리하는 페이지입니다. 각 단계 컴포넌트는 선택값을 상위의 `selections`에 저장하고, 마지막에는 결과 페이지로 상태를 전달합니다. 주요 관심사는 외부 라이브러리(갤러리/3D 등) 의존성, 타이머(cleanup), 접근성, 성능 최적화입니다.

원하면 바로 다음 단계로:
- `FanCardGallery` / `ColorWheelPicker` 등의 내부 구현을 가져오면 그 컴포넌트가 성능·의존성 문제를 일으킬 가능성을 라인 단위로 분석해 드립니다. 어떤 컴포넌트를 먼저 볼까요?