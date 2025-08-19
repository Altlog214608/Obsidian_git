FrSmart(파이썬 앱)와 React 모바일 웹 간 데이터 주고받기 방안과 React 앱 설계 및 구현 예시를 아래에 설명합니다.

***

## 1. FrSmart와 모바일 웹 데이터 주고받기 방안

### A. 로컬 네트워크 API 서버 방식  
- FrSmart 내에 간단한 Python 웹 서버(Flask, FastAPI 등)를 내장하거나 별도 실행  
- FrSmart가 AI 모델 추론 후 결과를 API 응답(JSON)으로 제공  
- 모바일 웹은 API를 호출해 최신 데이터 받아 UI 동적 갱신  
- 네트워크 환경 필요(같은 Wi-Fi 내 연결 등)

### B. 파일 공유 방식  
- FrSmart가 JSON 형식 결과를 로컬 파일(예: 공유 폴더)에 저장  
- 모바일 웹은 로컬 웹뷰, 혹은 로컬 서버를 통해 이 파일을 읽어 화면 표시  
- 간단하나 실시간성 낮고 사용자 환경에 제약 있음

### C. 클라우드 API 서버 방식  
- FrSmart가 클라우드 서버(예: AWS, GCP, Azure) API 호출로 데이터 업로드/저장  
- 모바일 웹은 클라우드 API를 호출해 데이터 취득  
- 인프라 필요 및 관리 복잡도 ↑  
- 확장성과 실시간성 우수

***

## 2. React 앱 설계 및 구현 예시

### A. 컴포넌트 구조 예

- `App`: 전체 루트 컴포넌트  
- `UserInputForm`: 사용자 입력폼 (성별, 나이, 선호 스타일, 색상 등)  
- `ResultDisplay`: AI 추천 결과 표시 (향수명, 사진, 향 노트 비율, 애니메이션 등)  
- `ProgressBar`: 노트별 애니메이션용 진행바  
- `ImageDownloadButton`: 이미지 다운로드 기능  
- `SNSShareButton`: SNS 공유 기능  

***

### B. 데이터 플로우

1. `UserInputForm`에서 입력값을 받아 `App` 상태로 전달  
2. `App`은 입력값을 FrSmart API 서버(로컬/원격)로 POST 요청  
3. FrSmart API에서 AI 결과를 JSON으로 응답  
4. `App`은 JSON 데이터 상태 저장 후 `ResultDisplay`로 전달  
5. `ResultDisplay`는 받은 데이터 기반으로 동적 UI, 애니메이션(예: React Spring, Framer Motion) 처리  
6. 사용자 조작에 따라 이미지 다운로드 및 SNS 공유 가능  

***

### C. React 구현 예시 (핵심 코드 스니펫)

```jsx
// App.js
import React, { useState } from 'react';
import UserInputForm from './UserInputForm';
import ResultDisplay from './ResultDisplay';

function App() {
  const [resultData, setResultData] = useState(null);

  const fetchRecommendation = async (inputParams) => {
    const response = await fetch('http://frsmart.local/api/recommend', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(inputParams)
    });
    const data = await response.json();
    setResultData(data);
  };

  return (
    
      
      {resultData && }
    
  );
}

export default App;
```

```jsx
// UserInputForm.js
import React, { useState } from 'react';

function UserInputForm({ onSubmit }) {
  const [form, setForm] = useState({
    gender: 'female',
    age: 20,
    style: '',
    color: '',
    // 기타 파라미터
  });

  const handleChange = (e) => {
    setForm({...form, [e.target.name]: e.target.value});
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(form);
  };

  return (
    
      {/* 입력 UI 예: 성별, 나이, 스타일 */}
      
        여성
        남성
      
      {/* 추가 입력폼 */}
      추천받기
    
  );
}

export default UserInputForm;
```

```jsx
// ResultDisplay.js
import React from 'react';

function ResultDisplay({ data }) {
  return (
    
      {data.perfumeName}
      
      
        탑 노트: {data.topNote.percent}% {data.topNote.name}
        미들 노트: {data.middleNote.percent}% {data.middleNote.name}
        베이스 노트: {data.baseNote.percent}% {data.baseNote.name}
      
      {/* 애니메이션 및 스크롤 이벤트 연동 가능 */}
      {/* 이미지 다운로드, SNS 공유 버튼 추가 */}
    
  );
}

export default ResultDisplay;
```

***

### D. 프론트엔드에서 이미지 다운로드 및 SNS 공유  
- 이미지 다운로드: HTML5 `` 태그의 `download` 속성 이용  
- SNS 공유: 각 플랫폼 공유 URL 또는 JS SDK 활용 (예: Facebook, Twitter, Kakao Talk)

***

## 결론
- FrSmart가 AI 처리 및 데이터 관리 수행  
- React 모바일 웹은 API 호출로 결과 받아 동적 시각화 UI 제공  
- 네트워크 API 서버(로컬 또는 클라우드) 구성 필요  
- React는 서버가 꼭 필요하지 않으나, 실시간 데이터 교환에 API 서버가 필수

필요시 API 서버 구축 예시 및 React 배포 방법도 안내 가능합니다.
