

---

### 📘 React Design Patterns | Refine — 번역 순서

1️⃣ **Introduction (소개)**  
2️⃣ **Container and Presentation Pattern (컨테이너 & 프레젠테이션 패턴)**  
3️⃣ **Component Composition with Hooks (Hooks 조합 패턴)**  
4️⃣ **State Management with Reducers (Reducer 기반 상태관리 패턴)**  
5️⃣ **Data Management with Providers (Provider 패턴)**  
6️⃣ **Component Enhancement with HOCs (고차 컴포넌트)**  
7️⃣ **Compound Components (컴파운드 컴포넌트)**  
8️⃣ **Prop Combination (Props 결합 패턴)**  
9️⃣ **Controlled Inputs (제어된 입력 패턴)**  
🔟 **Error Boundaries Pattern (에러 경계 패턴)**  
11️⃣ **ForwardRefs (Ref 전달 패턴)**  
12️⃣ **Lazy Loading Components (지연 로딩 패턴)**  
13️⃣ **Memoization Patterns (메모이제이션 패턴)**  
14️⃣ **React Server Components (RSC 기반 데이터 패칭)**  
15️⃣ **Conclusion (결론)**


---

## 🟦 1. Introduction (소개)

**원문 요약:**  
React 개발자는 검증된 디자인 패턴을 활용함으로써 문제를 더 빠르고 효율적으로 해결할 수 있습니다. 디자인 패턴은 결합도를 낮춘 일관성 있는 모듈 구성을 가능하게 하며, 결과적으로 유지보수성과 확장성이 높은 애플리케이션을 만드는 데 도움을 줍니다.

---

### 🇰🇷 **한글 번역**

**소개**

React 디자인 패턴은 이미 검증된 방식으로 반복되는 문제를 해결할 수 있도록 도와주는 설계 기법입니다.  
이러한 패턴을 활용하면 React 개발자들은 **시간과 노력을 절약**하면서도 **유지보수가 용이하고, 확장 가능하며, 효율적인 애플리케이션**을 만들 수 있습니다.

본 글에서는 다양한 React 디자인 패턴을 살펴보고, 이러한 패턴들이 React 애플리케이션 개발을 어떻게 개선할 수 있는지 구체적으로 다룹니다.

---

**이 글에서 다룰 주요 항목**

- Container & Presentation 패턴
    
- Hooks를 활용한 컴포넌트 조합
    
- Reducer 기반의 상태 관리
    
- Provider 패턴을 통한 데이터 관리
    
- 고차 컴포넌트(HOC)를 이용한 기능 확장
    
- Compound Components
    
- Prop Combination
    
- Lazy Loading
    
- Controlled Inputs
    
- Error Boundaries
    
- ForwardRefs
    
- React Server Components (RSC)
    

## 🟦 2. Container and Presentation Pattern (컨테이너 & 프레젠테이션 패턴)

이 패턴은 **비즈니스 로직과 UI 표현 로직을 분리**하는 것을 목표로 합니다.  
이를 통해 코드의 **모듈화, 테스트 용이성, 관심사의 분리(Separation of Concerns)** 원칙을 충실히 따를 수 있습니다.

React 애플리케이션에서는 종종 백엔드에서 데이터를 가져오거나 특정 로직을 계산해야 할 때가 있습니다.  
이때 컨테이너-프레젠테이션 패턴은 다음 두 가지 컴포넌트로 나누는 방식으로 빛을 발합니다:

- **Container Component**: 데이터 로딩, 연산, 상태 관리 등 **비즈니스 로직** 담당
    
- **Presentation Component**: 전달받은 데이터를 **UI에 렌더링**하는 역할 담당
    

```tsx
// StarWars 캐릭터 데이터를 불러오는 Container
import React, { useEffect, useState } from "react";
import CharacterList from "./CharacterList";

const StarWarsCharactersContainer: React.FC = () => {
  const [characters, setCharacters] = useState<Character[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(false);

  const getCharacters = async () => {
    setIsLoading(true);
    try {
      const res = await fetch("https://akabab.github.io/starwars-api/api/all.json");
      const data = await res.json();
      if (data) setCharacters(data);
    } catch {
      setError(true);
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => { getCharacters(); }, []);

  return <CharacterList loading={isLoading} error={error} characters={characters} />;
};

export default StarWarsCharactersContainer;
```

이처럼 비즈니스 로직은 컨테이너에, UI 표현은 프레젠테이션 컴포넌트로 분리하여 관리합니다.

---

## 🟦 3. Component Composition with Hooks (Hooks 조합 패턴)

Hooks는 React 16.8에서 도입된 기능으로, 함수형 컴포넌트에서도 상태와 생명주기 로직을 사용할 수 있게 해줍니다.  
Hooks를 활용하면 **공통 로직을 커스텀 Hook으로 분리**하여 재사용할 수 있습니다.

예를 들어 StarWars 데이터를 불러오는 로직을 `useFetchStarWarsCharacters`라는 Hook으로 분리하면 다음과 같습니다.

```tsx
// 데이터 패칭 로직만 분리한 Custom Hook
export const useFetchStarWarsCharacters = () => {
  const [characters, setCharacters] = useState<Character[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(false);
  const controller = new AbortController();

  const getCharacters = async () => {
    setIsLoading(true);
    try {
      const response = await fetch("https://akabab.github.io/starwars-api/api/all.json", {
        method: "GET",
        headers: { "Content-Type": "application/json" },
        signal: controller.signal,
      });
      const data = await response.json();
      if (data) setCharacters(data);
    } catch {
      setError(true);
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    getCharacters();
    return () => controller.abort();
  }, []);

  return [characters, isLoading, error];
};
```

이후 컨테이너 컴포넌트에서 간단히 불러와 사용할 수 있습니다.

```tsx
const StarWarsCharactersContainer: React.FC = () => {
  const [characters, isLoading, error] = useFetchStarWarsCharacters();
  return <CharacterList loading={isLoading} error={error} characters={characters} />;
};
```

이렇게 하면 **상태 관리 로직과 UI 로직을 완전히 분리**할 수 있고, Hook 단위 테스트도 가능합니다.

---

## 🟦 4. State Management with Reducers (Reducer 기반 상태관리 패턴)

여러 개의 상태를 각각 관리하기 어려울 때, `useReducer`를 사용해 상태를 액션 중심으로 관리할 수 있습니다.  
Reducer 패턴은 **액션(action)**을 통해 상태를 예측 가능하게 변경하는 방식입니다.

```tsx
import React, { useReducer } from "react";

const initState = { loggedIn: false, user: null, token: null };

function authReducer(state, action) {
  switch (action.type) {
    case "login":
      return { loggedIn: true, user: action.payload.user, token: action.payload.token };
    case "logout":
      return initState;
    default:
      return state;
  }
}

const AuthComponent = () => {
  const [state, dispatch] = useReducer(authReducer, initState);

  return (
    <div>
      {state.loggedIn ? (
        <>
          <p>Welcome {state.user.name}</p>
          <button onClick={() => dispatch({ type: "logout" })}>Logout</button>
        </>
      ) : (
        <button onClick={() => dispatch({ type: "login", payload: { user: { name: "John" }, token: "abc" } })}>Login</button>
      )}
    </div>
  );
};
```

---

## 🟦 5. Data Management with Providers (Provider 패턴)

**Context API**를 활용하여 데이터 전달 문제(prop drilling)를 해결하는 패턴입니다.

```tsx
export const ThemeContext = React.createContext(null);

export function ThemeProvider({ children }) {
  const [theme, setTheme] = React.useState("light");

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

이제 `useContext`로 어떤 자식 컴포넌트에서도 쉽게 접근할 수 있습니다.

```tsx
const TopNav = () => {
  const { theme, setTheme } = useContext(ThemeContext);
  return <div style={{ background: theme === "light" ? "#fff" : "#000" }}>Navigation</div>;
};
```

---

## 🟦 6. Component Enhancement with HOCs (고차 컴포넌트)

**HOC(Higher-Order Component)**는 컴포넌트를 입력으로 받아, 기능이 추가된 새 컴포넌트를 반환하는 패턴입니다.

```tsx
const higherOrderComponent = (Component) => {
  return class HOC extends React.Component {
    state = { name: "John Doe" };
    render() {
      return <Component name={this.state.name} {...this.props} />;
    }
  };
};

const AvatarComponent = (props) => (
  <div>
    <div>{props.name}</div>
    <p>I am a {props.description}</p>
  </div>
);

const EnhancedAvatar = higherOrderComponent(AvatarComponent);
```

이렇게 하면 동일한 로직을 여러 컴포넌트에 손쉽게 적용할 수 있습니다.

---

## 🟦 7. Compound Components (컴파운드 컴포넌트)

부모 컴포넌트와 자식 컴포넌트가 **Context나 Props를 통해 상호작용**하는 패턴입니다.  
예를 들어, `Toggle` 컴포넌트와 그 하위 요소들을 구성할 수 있습니다.

```tsx
const ToggleContext = createContext();

function Toggle({ children }) {
  const [on, setOn] = useState(false);
  const toggle = () => setOn(!on);

  return <ToggleContext.Provider value={{ on, toggle }}>{children}</ToggleContext.Provider>;
}

Toggle.On = ({ children }) => useContext(ToggleContext).on ? children : null;
Toggle.Off = ({ children }) => useContext(ToggleContext).on ? null : children;
Toggle.Button = (props) => {
  const { toggle } = useContext(ToggleContext);
  return <button onClick={toggle} {...props} />;
};
```

이 구조 덕분에, UI를 유연하게 재조합할 수 있습니다.

---

## 🟦 8. Prop Combination (Props 결합 패턴)

여러 관련 속성을 하나의 객체로 묶어 전달하는 방식입니다.

```tsx
function P({ color, size, children, ...rest }) {
  return <p style={{ color, fontSize: size }} {...rest}>{children}</p>;
}

const paragraphProps = { color: "red", size: "20px", lineHeight: "22px" };
<P {...paragraphProps}>텍스트 예시</P>;
```

이렇게 하면 코드가 단순해지고 관리가 쉬워집니다.

---

## 🟦 9. Controlled Inputs (제어된 입력)

입력값을 React 상태로 제어하는 패턴입니다.  
`value`와 `onChange`를 함께 사용하여 React가 입력값을 완전히 통제합니다.

```tsx
function ControlledInput() {
  const [inputValue, setInputValue] = useState("");
  return <input type="text" value={inputValue} onChange={(e) => setInputValue(e.target.value)} />;
}
```

이 방식은 DOM 직접 접근보다 **예측 가능하고 안정적인 입력 관리**를 가능하게 합니다.

---

## 🟦 10. Error Boundaries (에러 경계 패턴)

React 컴포넌트가 오류를 일으켰을 때 **전체 UI가 무너지는 것을 방지**하는 패턴입니다.  
`componentDidCatch`와 `getDerivedStateFromError` 메서드를 가진 클래스 컴포넌트로 구현합니다.

```tsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(error, info) { console.error(error, info); }

  render() {
    if (this.state.hasError) return <h1>문제가 발생했습니다.</h1>;
    return this.props.children;
  }
}
```

---

## 🟦 11. ForwardRefs (Ref 전달 패턴)

`React.forwardRef`를 사용하여 부모 컴포넌트에서 자식의 DOM 노드에 접근할 수 있게 합니다.

```tsx
const CustomInput = React.forwardRef((props, ref) => <input {...props} ref={ref} />);
const Parent = () => {
  const inputRef = useRef();
  useEffect(() => inputRef.current.focus(), []);
  return <CustomInput ref={inputRef} />;
};
```

---

## 🟦 12. Lazy Loading Components (지연 로딩)

큰 컴포넌트나 라우트를 **필요할 때만 불러와 렌더링**합니다.  
`React.lazy()`와 `Suspense`를 함께 사용합니다.

```tsx
const MyComponent = React.lazy(() => import("./MyComponent"));
function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <MyComponent />
    </Suspense>
  );
}
```

이는 초기 로딩 속도를 단축하고 사용자 경험을 향상시킵니다.

---

## 🟦 13. Memoization Patterns (메모이제이션)

불필요한 렌더링을 방지하기 위한 최적화 기법입니다.

- **React.memo** — props가 변경될 때만 리렌더링
    
- **useMemo** — 값 계산 결과를 캐싱
    
- **useCallback** — 함수 인스턴스를 캐싱
    

```tsx
const MyComponent = React.memo(({ count }) => <div>{count}</div>);
const calc = (num) => num * 2;
const value = useMemo(() => calc(number), [number]);
const handleClick = useCallback(() => console.log("Clicked"), []);
```

이 패턴은 데이터 집약적 UI나 복잡한 상호작용에 매우 유용합니다.

---

여기까지가 **1~13번 패턴 전체 번역**입니다 ✅  
다음 단계로 **14️⃣ React Server Components (RSC) + 15️⃣ Conclusion (결론)**까지 번역해드릴까요?