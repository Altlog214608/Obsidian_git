- 개요(한 문장)
  - CustomExperience 페이지는 여러 단계(프로필 → 스타일 → 색상 → 계절 → 향조 → 상황)를 한 군데서 순차적으로 제어하고, 각 단계 컴포넌트를 임포트하여 selections 상태를 갱신한 뒤 최종 선택을 결과 페이지로 전달하는 "멀티스텝 플로우 오케스트레이터"입니다.

- 데이터 흐름 (핵심)
  - 중앙 상태: CustomExperience 내부의 selections: SelectionData (한 곳에서 모든 단계의 선택값을 보유).
  - 렌더링: currentStep에 따라 해당 단계 컴포넌트를 조건부 렌더.
  - 전달(props):
    - 하위 컴포넌트들은 현재 선택값(selectedXXX)과 변경 콜백(onXXX)을 prop으로 받아서 setSelections를 통해 selections를 갱신(상향식 상태관리, "lifting state").
  - navigation: handleNext/handleBack로 currentStep 변경. 마지막 단계에서 selections를 navigate('/top3-results', { state: { selections } })로 전달.

- 각 파일(역할 요약)
  - StepContainer: 단계 레이아웃(타이틀, 진행도, 네비게이션 자리 제공).
  - GenderAgeSelector: 성별/연령/MBTI 입력 처리 → setSelections로 반영.
  - FashionStyleGallery: 패션 스타일 단일 선택 반영.
  - ColorWheelPicker: 색상/향조(옵션 목록 제공) 선택 컴포넌트(선택 시 onChange 호출).
  - SeasonSelector: 계절 선택 반영.
  - SituationSelector: SITUATION_ITEMS를 FanCardGallery로 보여주고 중앙 아이템 변경 시 selections.situation 업데이트.
  - FanCardGallery: (외부 컴포넌트) 중앙 카드 인덱스 변경 콜백 제공 — 여기에 swiper/ogl와 같은 외부 의존이 있을 수 있음.
  - LoadingScreen: 최종 처리(모의 AI) 중 표시.

- 사용자 상호작용 흐름 요약
  1. 사용자 입력 → 하위 컴포넌트가 onX 호출 → setSelections 합침.
  2. 버튼(다음) → canProceed 검사(pass 시 next).
  3. 마지막 단계 → isLoading true → LoadingScreen 보이고 setTimeout 후 결과 페이지로 navigate(state 포함).

- 주의/개선 포인트(우선순위)
  1. setTimeout 정리 필요: 컴포넌트 언마운트/빠른 내비게이션 시 타이머가 남음 → memory leak / 예기치 동작 가능.
  2. 외부 의존성: FanCardGallery가 swiper/ogl 같은 패키지에 의존하면 해당 패키지 설치 필요(개발서버 에러 원인).
  3. 타입 안전성: selections의 각 필드 타입(배열/문자열)을 잘 좁혀서 하위 컴포넌트에 정확히 전달.
  4. 접근성: 입력 컨트롤/링크에 aria-label/키보드 네비게이션 검토.
  5. 성능: selections가 커지면 모든 하위 컴포넌트가 리렌더될 수 있으므로 단계 컴포넌트를 React.memo로 감싸거나 상태 국소화 고려.

- 테스트 체크리스트 (2–5분)
  1. dev 서버 실행: npm run dev
  2. 페이지 경로: 앱에서 "맞춤 추천" 열기 → 각 단계에서 값 선택 가능 확인
  3. 마지막 단계 → 다음 클릭 → LoadingScreen 표시 후 top3-results로 이동되는지(개발자 도구 콘솔에서 selections 확인)
  4. FanCardGallery 관련 에러(log)에 swiper/ogl 누락 메시지 있으면 npm install swiper ogl

- 빠른 권장 패치: setTimeout cleanup
  - CustomExperience에서 마지막 단계의 setTimeout을 useRef로 관리하고 언마운트 시 clearTimeout 하도록 변경 권장. (아래 패치 예시)

````typescript
// ...existing code...
import { useRef } from 'react'
// ...existing code...

function CustomExperience(): React.JSX.Element {
  // ...existing code...
  const timeoutRef = useRef<number | null>(null)

  const handleNext = (): void => {
    if (currentStep < STEPS.length - 1) {
      setCurrentStep((prev) => prev + 1)
    } else {
      setIsLoading(true)
      console.log('Final selections:', selections)

      // Simulate AI processing time (5 seconds)
      if (timeoutRef.current) {
        window.clearTimeout(timeoutRef.current)
      }
      timeoutRef.current = window.setTimeout(() => {
        timeoutRef.current = null
        navigate('/top3-results', { state: { selections } })
      }, 5000)
    }
  }

  useEffect(() => {
    return () => {
      if (timeoutRef.current) {
        window.clearTimeout(timeoutRef.current)
        timeoutRef.current = null
      }
    }
  }, [])
  // ...existing code...
}
// ...existing code...
````

원하면
- FanCardGallery / ColorWheelPicker 내부(의존 라이브러리·성능) 코드도 분석해 드립니다. 어느 컴포넌트부터 볼까요?

