GitHub Copilot

`src/gesture-radial` 폴더의 역할을 설명하기 위해 먼저 그 폴더 구조와 파일들을 확인해야 합니다.

현재 폴더 구조를 보면:
````
src/renderer/src/components/gesture-radial/
  ├── constants.ts
  ├── GlassBloomButton.tsx
  ├── hooks.ts
  ├── index.ts
  ├── SimpleRadialAdjuster.tsx
  ├── types.ts
  └── utils.ts
````

## 전체 역할

**gesture-radial은 "제스처 기반 라디얼(원형) 인터페이스" 시스템**입니다. 사용자의 손가락 움직임(제스처)으로 원형 컨트롤러를 조작하는 UI 모듈입니다.

## 각 파일의 역할

### 1. **types.ts** — 타입 정의
````typescript
// 라디얼 컨트롤의 상태와 데이터 구조 정의
interface RadialGestureState {
  angle: number          // 각도
  distance: number       // 중심으로부터의 거리
  isActive: boolean      // 활성화 상태
}
````

### 2. **constants.ts** — 고정값/설정
````typescript
// 라디얼 UI의 고정 설정값들
export const RADIAL_RADIUS = 150
export const GESTURE_THRESHOLD = 20
export const ANIMATION_DURATION = 300
````

### 3. **utils.ts** — 유틸리티 함수
````typescript
// 계산 함수들
export function calculateAngle(x: number, y: number): number {
  // 마우스/터치 좌표를 각도로 변환
}

export function calculateDistance(x: number, y: number): number {
  // 중심으로부터의 거리 계산
}
````

### 4. **hooks.ts** — React 훅
````typescript
// 제스처 감지 및 상태 관리 훅
export function useGestureRadial() {
  // 터치/마우스 이벤트 감지
  // 라디얼 상태 업데이트
  // 콜백 실행
}
````

### 5. **GlassBloomButton.tsx** — 유리 디자인 버튼
````typescript
// 반투명 글래스 효과의 라디얼 버튼 컴포넌트
function GlassBloomButton({ 
  angle,      // 원 위의 각도
  distance,   // 중심으로부터의 거리
  label       // 버튼 레이블
}) {
  // 반투명 배경 + 블러 효과
}
````

### 6. **SimpleRadialAdjuster.tsx** — 간단한 라디얼 조정기
````typescript
// 사용자가 손가락으로 드래그해서 값을 조정하는 UI
function SimpleRadialAdjuster({
  value,           // 현재 값
  onChange,        // 값 변경 콜백
  min, max         // 최소/최대 값
}) {
  // 제스처로 값 조정
}
````

### 7. **index.ts** — 모듈 진입점 및 export
````typescript
// 이 폴더가 제공하는 모든 컴포넌트/함수 export
export { GlassBloomButton }
export { SimpleRadialAdjuster }
export * from './types'
export * from './hooks'
````

## 실제 사용 예시

````typescript
// 다른 컴포넌트에서 사용
import { SimpleRadialAdjuster, useGestureRadial } from '@renderer/components/gesture-radial'

function PerfumeTonerPage() {
  const [tone, setTone] = useState(50)
  
  return (
    <SimpleRadialAdjuster
      value={tone}
      onChange={setTone}
      min={0}
      max={100}
      label="향수 톤 조정"
    />
  )
}
````

## 키오스크 맥락에서의 의미

이 모듈은 **터치스크린 키오스크에 최적화**되어 있습니다:
- 손가락으로 원을 따라 드래그해서 값 조정
- 터치 제스처 자연스러운 인터랙션
- 반투명 글래스 디자인으로 세련된 UI

예: 향수 농도, 톤 조정, 계절 선택 등을 원형 제스처로 직관적으로 조작

## 파일 내용 확인 원할 시

````bash
# gesture-radial 폴더 내 모든 파일 확인
code src\renderer\src\components\gesture-radial
````

혹은 특정 파일(예: `SimpleRadialAdjuster.tsx`)의 코드를 붙여주시면 라인별로 상세 설명하겠습니다.