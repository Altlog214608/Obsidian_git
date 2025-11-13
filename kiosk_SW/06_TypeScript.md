# TypeScript 기본 개념과 사용법

## TypeScript란?
JavaScript의 슈퍼셋으로, 정적 타입을 추가한 프로그래밍 언어입니다. Microsoft가 개발했으며, 대규모 애플리케이션 개발에 적합합니다.

## 1. 기본 타입 시스템

### 기본 타입
````typescript
// 기본 타입 예시
let name: string = "John";
let age: number = 25;
let isStudent: boolean = true;
let numbers: number[] = [1, 2, 3];
let tuple: [string, number] = ["hello", 10];
````

### 인터페이스
````typescript
// 객체의 형태를 정의
interface User {
  name: string;
  age: number;
  email?: string; // 선택적 속성
  readonly id: number; // 읽기 전용 속성
}

const user: User = {
  name: "John",
  age: 25,
  id: 1
};
````

## 2. 고급 타입

### Union Types
````typescript
// 여러 타입 중 하나를 사용
type Status = "pending" | "approved" | "rejected";
let currentStatus: Status = "pending";
````

### Generic Types
````typescript
// 재사용 가능한 컴포넌트 생성
function getArray<T>(items: T[]): T[] {
    return new Array<T>().concat(items);
}

let numbers = getArray<number>([1, 2, 3]);
let strings = getArray<string>(["a", "b", "c"]);
````

## 3. React에서의 TypeScript

### 컴포넌트 예시
````typescript
interface ButtonProps {
  text: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary';
}

const Button: React.FC<ButtonProps> = ({ 
  text, 
  onClick, 
  variant = 'primary' 
}) => {
  return (
    <button 
      onClick={onClick}
      className={`btn-${variant}`}
    >
      {text}
    </button>
  );
};
````

### Hooks with TypeScript
````typescript
// useState with TypeScript
const [count, setCount] = useState<number>(0);

// Custom Hook
interface UseUserResult {
  user: User | null;
  loading: boolean;
  error: Error | null;
}

function useUser(id: string): UseUserResult {
  // ...implementation
}
````

## 4. 유용한 TypeScript 기능

### Type Assertions
````typescript
let someValue: unknown = "this is a string";
let strLength: number = (someValue as string).length;
````

### Mapped Types
````typescript
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};

type Partial<T> = {
    [P in keyof T]?: T[P];
};
````

## 5. TypeScript 설정

### tsconfig.json 기본 설정
````json
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}
````

## 6. 디버깅 및 개발 도구

### VS Code 설정
- TypeScript IntelliSense
- 자동 완성
- 실시간 타입 체크
- 리팩토링 도구

### 컴파일 명령어
```bash
# TypeScript 설치
npm install -g typescript

# 컴파일
tsc filename.ts

# 실시간 컴파일
tsc filename.ts --watch
```

## 7. 모범 사례

1. **타입 명시적 선언**
````typescript
// 좋은 예시
interface UserState {
  users: User[];
  loading: boolean;
  error: Error | null;
}

// 나쁜 예시
const state = {
  users: [],  // any[] 타입으로 추론됨
  loading: false
};
````

2. **타입 가드 사용**
````typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function processValue(value: unknown) {
  if (isString(value)) {
    console.log(value.toUpperCase()); // 타입 안전성 보장
  }
}
````

이러한 TypeScript의 기능들을 활용하면 더 안전하고 유지보수가 쉬운 코드를 작성할 수 있습니다.

