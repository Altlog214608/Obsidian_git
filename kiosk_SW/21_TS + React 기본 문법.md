좋아, 이 코드 한 파일에 **TS + React 기본 문법 거의 다 들어있어서**  
이거만 제대로 이해해도 타입스크립트-리액트 감 잡는 데 꽤 도움 될 거야.  
질문한 **`interface`랑 `export default`** 포함해서 중요한 것들 순서대로 짚어볼게.

---

## 1. `type` vs `interface` – 이 파일에서 각각 뭐 하는 애들인가

### 🔹 `type HorizontalSelectorOption = { ... }`

```ts
export type HorizontalSelectorOption = {
  id: string
  label: string
  value: string
  icon?: React.ReactNode
  image?: string
  description?: string
}
```

- `type`은 말 그대로 **타입 별명**이야.  
    → “이런 모양의 객체를 HorizontalSelectorOption 이라고 부를게” 라는 선언.
    

각 필드 의미:

- `id: string` → 무조건 문자열
    
- `label: string` → 무조건 문자열
    
- `value: string` → 무조건 문자열
    
- `icon?: React.ReactNode`  
    → `?`가 붙어서 **옵션(optional)** 필드. 있어도 되고 없어도 됨.  
    → `React.ReactNode`는 JSX 안에 들어갈 수 있는 거 전부 (텍스트, 엘리먼트, Fragment 등)
    
- `image?: string` → `string` 또는 `undefined`
    
- `description?: string` → `string` 또는 `undefined`
    

즉, 이 컴포넌트에서 쓰는 **옵션 하나의 형태**를 타입으로 딱 정의해둔 거.

---

### 🔹 `interface HorizontalSelectorProps { ... }`

```ts
interface HorizontalSelectorProps {
  options: HorizontalSelectorOption[]
  onChange?: (option: HorizontalSelectorOption) => void
  selectedId?: string
  title?: string
  showLabel?: boolean
  itemSize?: 'sm' | 'md' | 'lg'
}
```

- `interface`는 **객체 타입 정의**할 때 사용하는 또 다른 문법이야.
    
- 여기서는 **컴포넌트의 props 타입**을 정의 중.
    

각 필드 설명:

- `options: HorizontalSelectorOption[]`  
    → 옵션 객체 배열. **필수**.
    
- `onChange?: (option: HorizontalSelectorOption) => void`  
    → 함수 타입.  
    → 매개변수로 `HorizontalSelectorOption` 하나 받고, 리턴값은 `void`.  
    → `?` 있어서 props로 안 넘겨도 됨.
    
- `selectedId?: string` → 선택된 아이템 id (controlled 모드일 때 사용)
    
- `title?: string` → 상단 타이틀
    
- `showLabel?: boolean` → 아래쪽 라벨 보여줄지 여부
    
- `itemSize?: 'sm' | 'md' | 'lg'`  
    → **문자열 리터럴 유니온 타입**  
    → `'sm'`, `'md'`, `'lg'` 중 하나만 가능 (다른 문자열 쓰면 타입 에러)
    

> 💡 `type`이랑 `interface` 둘 다 “모양 정의” 하는데,
> 
> - `interface`는 주로 **객체/props**
>     
> - `type`은 객체뿐 아니라 `union` 같은 복잡한 타입 정의에도 많이 써.
>     

---

## 2. 컴포넌트 선언부 – TS + React 시그니처

```ts
export default function HorizontalSelector({
  options,
  onChange,
  selectedId: controlledSelectedId,
  title,
  showLabel = true,
  itemSize = 'md'
}: HorizontalSelectorProps): React.JSX.Element {
```

여기 중요 포인트 여러 개 있음.

### 2-1. `export default`

- 이 파일에서 **“기본으로 내보내는(export) 것”**이 이 함수라는 뜻.
    
- 다른 파일에서 import 할 때:
    
    ```ts
    import HorizontalSelector from './HorizontalSelector'
    ```
    
    처럼 **이름 마음대로** 가져올 수 있음 (물론 관례상 같은 이름 쓰지만).
    
- 만약 `export`만 있고 `default`가 없으면:
    
    ```ts
    export function HorizontalSelector() { ... }
    ```
    
    이 경우는:
    
    ```ts
    import { HorizontalSelector } from './HorizontalSelector'
    ```
    
    같이 **중괄호로 이름 그대로** 가져와야 함.
    

### 2-2. props 구조 분해 + 기본값

```ts
{
  options,
  onChange,
  selectedId: controlledSelectedId,
  title,
  showLabel = true,
  itemSize = 'md'
}: HorizontalSelectorProps
```

- `HorizontalSelectorProps` 타입을 가진 객체를 인자로 받는데,
    
- 그 자리에서 바로 **구조 분해**하고 있음.
    
- `selectedId: controlledSelectedId`  
    → props 이름은 `selectedId`인데, 함수 안에서는 `controlledSelectedId`라는 변수명으로 쓰겠다는 뜻.
    
- `showLabel = true`  
    → props에 `showLabel`이 안 들어오면 기본값 `true`.
    
- `itemSize = 'md'`  
    → 기본값 `'md'`.
    

### 2-3. `: React.JSX.Element`

- 이 함수가 **React JSX Element를 리턴한다**는 타입 선언.
    
- 보통은 `React.FC<HorizontalSelectorProps>` 이런 식으로도 많이 쓰지만,  
    요즘은 `FC` 없이 이렇게 함수 시그니처에 바로 리턴 타입 붙이는 스타일도 많이 씀.
    

---

## 3. `useState<string | undefined>` – 제네릭 타입

```ts
const [internalSelectedId, setInternalSelectedId] = useState<string | undefined>(undefined)
```

- `useState<...>` 부분이 **제네릭(generic)**.
    
- 이 상태값의 타입이 `string | undefined` 라는 뜻.
    
- 초기값도 `undefined`.
    
- 이후에 `setInternalSelectedId('spring')` 이런 식으로 `string` 넣으면 됨.
    

---

## 4. `??` (nullish coalescing) – 기본값 선택 연산자

```ts
const selectedId = controlledSelectedId ?? internalSelectedId
```

- `??` 는 **nullish coalescing operator**.
    
- `왼쪽이 null 또는 undefined 이면 오른쪽 사용` 이란 뜻.
    

즉:

- `controlledSelectedId`가 **있으면 그걸 사용** (부모가 제어하는 "controlled" 모드)
    
- 없으면 **내부 state(`internalSelectedId`) 사용** (uncontrolled 모드)
    

---

## 5. 선택된 옵션 찾기

```ts
const selected = selectedId ? options.find((opt) => opt.id === selectedId) : undefined
```

- `selectedId`가 있으면 `options` 배열에서 해당 id를 가진 옵션을 찾고,
    
- 없으면 `undefined`.
    

타입스크립트 덕분에 `selected`가 `HorizontalSelectorOption | undefined` 타입이 됨.  
그래서 아래에서 `selected ? ... : ...` 이런 식으로 조건 렌더링 하는 거지.

---

## 6. union type + 맵핑 객체

```ts
const sizeMap = {
  sm: { container: 'h-20 w-20', inner: 'h-16 w-16' },
  md: { container: 'h-24 w-24', inner: 'h-20 w-20' },
  lg: { container: 'h-32 w-32', inner: 'h-28 w-28' }
}

const sizes = sizeMap[itemSize]
```

- `itemSize` 타입이 `'sm' | 'md' | 'lg'`라서,
    
- `sizeMap[itemSize]`는 항상 저 셋 중 하나.
    
- 이 덕에 `sizes.container`, `sizes.inner` 사용할 때 타입 안정성 좋음.
    

(더 빡세게 하려면 `as const` 붙여서 완전 리터럴 타입으로 잠가버리기도 함.)

---

## 7. 옵셔널 체이닝 `onChange?.(option)`

```ts
const handleSelect = (option: HorizontalSelectorOption): void => {
  setInternalSelectedId(option.id)
  onChange?.(option)
}
```

- `onChange`가 있을 수도 있고 없을 수도 있는 상황 (`onChange?: ...`).
    
- `onChange?.(option)`  
    → `onChange`가 `undefined`면 아무것도 안 하고,  
    → 함수면 `onChange(option)` 호출.
    
- `if (onChange) onChange(option)` 이거랑 같은 의미인데,  
    한 줄로 줄인 문법.
    

---

## 8. JSX + Tailwind + TS의 조합

```tsx
<div
  className={[
    `relative flex ${sizes.container} items-center justify-center rounded-full transition-all duration-500`,
    'shadow-[0_0.1px_1px_rgba(0,0,0,0.1)]',
    isActive
      ? 'scale-110 shadow-[0_0.1px_1px_rgba(0,0,0,0.1)] ring-2 ring-offset-2 ring-white/50'
      : 'hover:scale-105 hover:shadow-[0_0.1px_1px_rgba(0,0,0,0.1)]'
  ].join(' ')}
>
```

여기서 중요한 점:

- `className`에 **문자열 배열 + `.join(' ')`** → tailwind 클래스를 조건부로 깔끔하게 합치는 패턴.
    
- `isActive`에 따라 다른 클래스 적용.
    
- `style={{ ... }}` 안에서는 **JS 객체**라서 camelCase 사용 (`backgroundColor`, `boxShadow` 등).
    

---

## 9. 조건부 렌더링 패턴들

```tsx
{selected ? (
  ...
) : (
  ...
)}
```

```tsx
{selected.description && (
  <p>...</p>
)}
```

- 첫 번째는 “있으면 A, 없으면 B”
    
- 두 번째는 “있으면 이 태그 자체를 렌더링, 없으면 아무것도 안 보여줌”
    

TS 덕분에 `selected`가 `undefined`일 수 있는 걸 알고 있어서  
조건문 안에서만 safely 접근할 수 있게 해줌.

---

## 요약 – 이 파일에서 배울 수 있는 TS/React 포인트

1. `type` / `interface`로 **데이터 구조 & props 타입 정의**
    
2. 옵셔널 필드 `?` / union 타입 (`'sm' | 'md' | 'lg'`)
    
3. 함수형 컴포넌트 시그니처 + props 타입 지정  
    `(...: HorizontalSelectorProps): React.JSX.Element`
    
4. `useState<타입>()` 제네릭으로 상태 타입 지정
    
5. `??` (nullish coalescing)로 controlled/uncontrolled 분기
    
6. 옵셔널 체이닝 `onChange?.(...)`
    
7. `export default`로 기본 내보내기 & import 방식 차이
    
8. JSX에서 조건부 렌더링, 스타일, className 조합 패턴
    

---

이 코드 그대로 기준으로

- “완전 기초 타입스크립트 문법 정리”
    
- “리액트 + TS 패턴 정리”  
    둘 다 따로 정리해줄 수도 있어.
    

지금 이 파일 보다가 **“이 부분은 에러 나는데?”**  
혹은 **“이 구문은 아직 잘 이해 안 됨”** 하는 줄 있으면  
그 줄만 딱 찍어서 보내줘, 그 라인 기준으로 더 깊게 파서 설명해줄게.