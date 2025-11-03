📘 **React Design Patterns | Refine 요약**

이 문서는 **React 애플리케이션 개발 시 코드 구조의 일관성, 재사용성, 유지보수성**을 높이기 위한 주요 디자인 패턴을 정리한 Refine 공식 블로그 글입니다. (업데이트: 2025.07.23, React Server Components 추가)

---

### 🧭 **전체 목차**

1. Container & Presentation Pattern
    
2. Component Composition with Hooks
    
3. State Management with Reducers
    
4. Data Management with Providers
    
5. Component Enhancement with HOCs
    
6. Compound Components
    
7. Prop Combination
    
8. Controlled Inputs
    
9. Error Boundaries Pattern
    
10. ForwardRefs (Custom Components 관리)
    
11. Lazy Loading Components
    
12. Memoization Patterns (React.memo / useMemo / useCallback)
    
13. Data Fetching with React Server Components (RSC)
    
14. Conclusion
    

---

### 🧩 **핵심 요약**

#### 1️⃣ Container & Presentation Pattern

- **비즈니스 로직과 UI 렌더링 분리**  
    → `Container`는 데이터 fetch / 로직 계산 담당  
    → `Presentation`은 UI 표시 담당
    
- 장점: 모듈화, 테스트 용이성, 유지보수성 향상
    

#### 2️⃣ Component Composition with Hooks

- **Custom Hook**으로 상태 및 로직 분리
    
- 컴포넌트 재사용성과 테스트 용이성 향상
    
    ```tsx
    const [data, loading, error] = useFetchStarWarsCharacters();
    ```
    

#### 3️⃣ State Management with Reducers

- `useReducer`로 상태를 그룹화하고 action 기반으로 변경
    
- 복잡한 상태 관리 간소화 (ex: 로그인/로그아웃)
    

#### 4️⃣ Data Management with Providers

- **Context API** 기반 데이터 관리
    
- Prop drilling 방지
    
- `ThemeProvider`, `ThemeContext` 등으로 전역 데이터 전달
    

#### 5️⃣ Higher-Order Components (HOC)

- 컴포넌트에 **추가 기능 주입**
    
- 입력: 기존 컴포넌트 → 출력: 기능이 강화된 새 컴포넌트
    
    ```tsx
    const Enhanced = withUserData(Component);
    ```
    

#### 6️⃣ Compound Components

- **부모-자식 컴포넌트 간 구조적 상호작용**
    
- 재사용성과 유연성 향상
    
    ```tsx
    <Toggle>
      <Toggle.On>On</Toggle.On>
      <Toggle.Off>Off</Toggle.Off>
      <Toggle.Button />
    </Toggle>
    ```
    

#### 7 Prop Combination

- 관련 prop들을 하나의 객체로 묶어 전달  
    → 코드 간결화, 관리 용이
    

#### 8 Controlled Inputs

- **상태 기반 input 관리**
    
- `value`와 `onChange`로 상태 제어 (DOM 직접 접근 X)
    

#### 9 Error Boundaries Pattern

- 컴포넌트 오류 발생 시 전체 앱이 깨지지 않도록 보호
    
- `componentDidCatch`, `getDerivedStateFromError` 사용
    
- fallback UI 제공
    

#### 10 ForwardRefs

- **ref를 자식 컴포넌트로 전달**하여 DOM 접근 허용
    
- 외부 라이브러리나 커스텀 input과 상호작용 시 유용
    

#### 11 Lazy Loading Components

- **필요 시점에만 컴포넌트 로드** (`React.lazy`, `Suspense`)
    
- 초기 로딩 속도 개선 및 성능 최적화
    

#### 12 Memoization Patterns

- `React.memo`, `useMemo`, `useCallback`으로 불필요한 재렌더 방지
    
- 연산 비용 큰 컴포넌트에 최적
    

#### 13 Data Fetching with React Server Components (RSC)

- **서버에서 직접 데이터 가져와 렌더링**
    
- API 노출 없이 DB 접근 가능
    
- 클라이언트 전송 JS 감소 → 초기 로딩 성능 향상
    
    ```tsx
    export default async function Page() {
      const data = await fetch(...);
      return <List data={data} />;
    }
    ```
    

---

### 🧠 **결론**

이 문서는 React 애플리케이션을 **더 확장성 있고 유지보수하기 쉽게 만드는 핵심 패턴 13가지를** 정리합니다.  
적절히 조합해 사용하면 코드 품질, 협업 효율, 성능을 모두 개선할 수 있습니다.

---

원하신다면 이 내용을 기반으로

- 📄 **A4 한 페이지 요약문 (한글/영문 병기)**
    
- 🧱 **각 패턴별 코드 예시 정리표**
    
- 🧭 **프로젝트 적용 우선순위 가이드**  
    중 원하는 형태로 정리해드릴 수 있습니다.  
    어떤 방식으로 정리해드릴까요?