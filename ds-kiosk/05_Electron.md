# Electron 애플리케이션 구조 설명

## 개요
Electron은 Chromium과 Node.js를 결합하여 데스크톱 애플리케이션을 만들 수 있게 해주는 프레임워크입니다. 이 프로젝트에서는 크게 3가지 주요 프로세스로 구성됩니다.

## 1. Main Process (메인 프로세스)
- 위치: index.ts
- 역할:
  - 애플리케이션의 생명주기 관리
  - 네이티브 OS 기능 접근
  - 브라우저 창(BrowserWindow) 생성 및 관리
  - IPC (Inter-Process Communication) 통신 처리

```typescript
import { app, BrowserWindow } from 'electron'
import path from 'path'

function createWindow() {
  const win = new BrowserWindow({
    width: 1024,
    height: 768,
    webPreferences: {
      nodeIntegration: false,
      contextIsolation: true,
      preload: path.join(__dirname, '../preload/index.js')
    }
  })
  
  // 개발 모드에서는 로컬 서버, 프로덕션에서는 빌드된 파일 로드
  if (process.env.NODE_ENV === 'development') {
    win.loadURL('http://localhost:3000')
  } else {
    win.loadFile(path.join(__dirname, '../renderer/index.html'))
  }
}
```

## 2. Preload Process (프리로드 프로세스)
- 위치: index.ts
- 역할:
  - Main과 Renderer 프로세스 사이의 보안 브리지
  - 렌더러에 노출할 API 정의
  - Node.js API의 제한된 접근 제공

```typescript
import { contextBridge, ipcRenderer } from 'electron'

contextBridge.exposeInMainWorld('electronAPI', {
  // 렌더러에서 사용할 수 있는 안전한 API 정의
  sendMessage: (message: string) => ipcRenderer.send('message', message),
  onResponse: (callback: Function) => ipcRenderer.on('response', callback)
})
```

## 3. Renderer Process (렌더러 프로세스)
- 위치: renderer
- 역할:
  - 웹 기술(React, CSS)로 UI 구현
  - 사용자 인터페이스 렌더링
  - 프리로드 스크립트를 통한 메인 프로세스와 통신

```typescript
import React from 'react'

declare global {
  interface Window {
    electronAPI: {
      sendMessage: (message: string) => void
      onResponse: (callback: Function) => void
    }
  }
}

function App() {
  const handleClick = () => {
    window.electronAPI.sendMessage('Hello from Renderer!')
  }
  
  return (
    <button onClick={handleClick}>
      Send Message to Main
    </button>
  )
}
```

## 프로세스 간 통신 (IPC)
- Main ↔ Renderer 간 통신은 IPC를 통해 이루어짐
- 보안을 위해 직접 통신은 불가능하며, preload 스크립트를 통해 브리지 구성

## 보안 특징
1. Context Isolation
   - 렌더러 프로세스와 메인 프로세스의 실행 컨텍스트를 분리
   - `contextIsolation: true` 설정으로 활성화

2. Node Integration
   - 기본적으로 비활성화 (`nodeIntegration: false`)
   - 렌더러에서 직접적인 Node.js API 접근 방지

3. Sandbox
   - 렌더러 프로세스는 샌드박스 환경에서 실행
   - 시스템 리소스에 대한 직접 접근 제한

## 빌드 및 패키징
- electron-builder 또는 electron-forge를 사용
- 운영체제별 실행 파일 생성 (.exe, .dmg, .deb 등)
- 자동 업데이트 지원 가능

이 프로젝트에서는 위의 구조를 기반으로 하여 디지털 향수 키오스크 UI를 구현하고 있습니다.


---

# 유튜브 강의

##### 일렉트론은 JS, HTML, CSS를 이용해서 데스크톱 애플리케이션을 만들 수 있는 프레임워크

#### 크로스플랫폼 지원 하나의 코드 -> 윈도우, 맥, 리눅스 구동 가능

#### 백엔드 Node.js (OS native 기능)
#### 프론트엔트 크로미엄 Chromium 크롬 브라우저의 기반이 된 오픈소스(Web 기반 기술)

#### electron 기반 Atom, vscode, slack, discode

Anguler , Vue, React 사용 가능

#### 장점

1. 웹기술을 이용해 Desktop Application 개발이 가능
2. 한개의 코드로 Cross platform에서 작동
3. NPM을 이용해 node package들을 사용할 수 있음

#### 일렉트론 퀵스타트

node.js 설치 필요

cmd - node -v, npm -v 확인

#### 프로젝트 폴더 생성

mkdir my-electron-app
cd my-electron-app

#### 프로젝트 생성

npm init

#### Electron package 추가

npm install --save-dev electron

development일때만 electron 사용

#### Packge.json에 script 추가

"start" : "electron ."

npm start 로 스크립트를 실행 가능

#### index.html 생성

root 폴더에 index.html을 만들고 다음과 같이 작성

```html
<!DOCTYPE html> 
<html> 
	<head> 
		<meta charset="UTF-8"> 
		<!-- https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP --> 
		<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'"> 
		<meta http-equiv="X-Content-Security-Policy" content="default-src 'self'; script-src 'self'"> 
		<title>Hello World!</title> 
	</head> 
	<body>
	 <h1>Hello World!</h1>
	  We are using Node.js <span id="node-version"></span>, 
	  Chromium <span id="chrome-version"></span>, 
	  and Electron <span id="electron-version"></span>. 
	  </body> 
</html>
```


#### index.js 생성

root 폴더에 index.js를 만들고 다음과 같이 작성

```javascript
const { app, BrowserWindow } = require('electron') // include the Node.js 'path' module at the top of your file 
const path = require('path')

 // modify your existing createWindow() function 
function createWindow () { 
	 const win = new BrowserWindow({ 
	 width: 800, 
	 height: 600, 
	 webPreferences: { preload: path.join(__dirname, 'preload.js') } 
	 }) 
	 win.loadFile('index.html') 
} 
app.whenReady().then(() => { 
	createWindow() 
}) 
app.on('window-all-closed', function () { 
	if (process.platform !== 'darwin') app.quit() 
})
```



preload.js 생성

- root 폴더에 preload.js를 생성 합니다.

```javascript
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    if (element) element.innerText = text
  }

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency])
  }
})
```

프로젝트 실행

```
npm start
```

실행 화면

![](https://blog.kakaocdn.net/dna/bvFBKn/btrfo4ob8Wj/AAAAAAAAAAAAAAAAAAAAADzzWl876WsoDp7Fqba_6A58ViFEJ0eTZk0_pNX-Tgow/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1764514799&allow_ip=&allow_referer=&signature=04N5x0GyD9XhNVa%2Bp0AO6bn6lPs%3D)

Electron 실행

패키징과 배포

- 패키징과 배포를 위한 가장 빠른 방법은 Electron Forge를 사용하는 것입니다.
- Electron Forge 패키지를 추가합니다.

```
npm install --save-dev @electron-forge/cli
npx electron-forge import
```

- Forge의 "make"를 이용하여 빌드합니다.

```
npm run make
```

- 실행 파일의 위치는 다음과 같습니다.

```
\out\my-electron-app-win32-x64\my-electron-app.exe
```


- 리액트 패키지를 추가

```
npm install --save react react-dom
```

- 리액트 소스 폴더 만들기

```
mkdir src/js
```

- index.html에 root id 생성

```
<div id="root"></div>
```

- src/js/index.js 파일 만들기

```
import React from 'react';
import ReactDom from 'react-dom';

ReactDom.render(<h1>Hello React App</h1>, document.getElementById('root'));
```

- 웹팩용 패키치 추가

```
npm install --save-dev @babel/core @babel/preset-env @babel/preset-react babel-loader css-loader style-loader sass-loader sass webpack webpack-cli
```

- webpack.common.js 추가 (root 폴더)

```javascript
const path = require('path');

module.exports = {
  mode: 'development',
  entry: './src/js/index.js',
  devtool: 'inline-source-map',
  target: 'electron-renderer',
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [[
              '@babel/preset-env', {
                targets: {
                  esmodules: true
                }
              }],
              '@babel/preset-react']
          }
        }
      },
      {
        test: [/\.s[ac]ss$/i, /\.css$/i],
        use: [
          // Creates `style` nodes from JS strings
          'style-loader',
          // Translates CSS into CommonJS
          'css-loader',
          // Compiles Sass to CSS
          'sass-loader',
        ],
      }
    ]
  },
  resolve: {
    extensions: ['.js'],
  },
  output: {
    filename: 'app.js',
    path: path.resolve(__dirname, 'build', 'js'),
  },
};
```

- package.json에 webpack 스크립트 추가

```
"watch": "webpack --config webpack.common.js --watch",
```

- 스크립트 실행  
    - 스크립트가 실행되면 build/js/app.js가 생성됨.

```
npm run watch
```

- index.html에 app.js 추가

```
<div id="root"></div>
<script src="./build/js/app.js"></script>
```

- 실행

```
npm start
```

![](https://blog.kakaocdn.net/dna/sR9eq/btrftnVy8xr/AAAAAAAAAAAAAAAAAAAAAOI8vwm-gYwG1jyXwKTJ-XxL3DMzwhuLhiixkczITBc2/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1764514799&allow_ip=&allow_referer=&signature=mkLMqQKnUdKDG%2BTnlYeHwqsesfM%3D)

React App 실행 화면

React Component 만들기

- src/js/App.js 추가

```
import React from 'react';

export default function App() {
    return(
        <h1>I am App Component</h1>
    )
}
```

- src/js/index.js 수정

```
import App from './App';

ReactDom.render(<App/>, document.getElementById('root'));
```

- 저장(Ctrl+s)
- 실행화면 창에서 Reload(Ctrl+r)

![](https://blog.kakaocdn.net/dna/0i1ar/btrfDgOmD5x/AAAAAAAAAAAAAAAAAAAAANRcS9nuS1RhHEmhkl0FbFAOxFQbxEAd_koqvQl8e9N3/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1764514799&allow_ip=&allow_referer=&signature=Y%2F%2B3NPxykvTf5M5OIGPRCYZzkQ0%3D)

React Component 실행



앱이 처음에 실행되지 않았던 주요 원인들과 해결 방법을 설명해드리겠습니다:

1. **Electron Forge 설정 문제**
   - 초기에는 Electron Forge가 제대로 설정되지 않았었습니다.
   - `npx electron-forge import` 명령어로 프로젝트를 Electron Forge 형식으로 변환하여 해결했습니다.
   - 이 과정에서 필요한 설정 파일들과 스크립트가 자동으로 추가되었습니다.

2. **React 18 관련 문제**
   - index.js에서 예전 방식의 React 렌더링 메소드를 사용하고 있었습니다:
   ```javascript
   // 이전 코드 (동작하지 않음)
   ReactDOM.render(<h2>Hello, Electron!</h2>, document.getElementById("root"));
   ```
   - React 18의 새로운 방식으로 수정했습니다:
   ```javascript
   // 새로운 코드 (정상 동작)
   import { createRoot } from "react-dom/client";
   const root = createRoot(container);
   root.render(<h2>Hello, Electron!</h2>);
   ```

3. **Electron의 보안 설정 문제**
   - index.js에서 BrowserWindow의 웹 보안 설정이 React 앱 실행을 막고 있었습니다.
   - `nodeIntegration`과 `contextIsolation` 설정을 추가하여 해결했습니다:
   ```javascript
   webPreferences: {
     nodeIntegration: true,
     contextIsolation: false,
     preload: path.join(__dirname, 'preload.js')
   }
   ```

4. **빌드 프로세스 순서**
   - webpack과 Electron을 동시에 실행해야 했습니다.
   - `npm run watch`로 webpack을 실행하여 app.js 파일을 생성
   - 그 다음 `npm start`로 Electron 앱을 실행

이러한 문제들은 주로 다음과 같은 상황에서 발생합니다:
- Electron과 React를 함께 사용할 때의 보안 설정
- React 18의 새로운 렌더링 API 적용
- 빌드 도구(webpack)와 실행 도구(electron-forge)의 올바른 설정과 실행 순서



Webpack은 모던 JavaScript 애플리케이션을 위한 모듈 번들러입니다. 좀 더 쉽게 설명하면:

1. **Webpack의 역할:**
   - 여러 파일들을 하나로 합쳐줍니다 (번들링)
   - JSX, TypeScript, SASS 등을 브라우저가 이해할 수 있는 JavaScript와 CSS로 변환
   - 코드를 최적화하고 압축
   - 이미지나 폰트 같은 자원들도 관리

2. **왜 필요한가?**
   - 우리 프로젝트에서는:
     ```javascript
     import React from 'react';  // React 라이브러리 불러오기
     import App from './App';    // App.js 파일 불러오기
     ```
   - 이런 import 문은 브라우저가 직접 이해할 수 없습니다
   - Webpack이 이 파일들을 모두 모아서 브라우저가 이해할 수 있는 하나의 파일(app.js)로 만들어줍니다

3. **`npm run watch`가 필요한 이유:**
   - `watch` 모드는 파일 변경을 감지하고 자동으로 다시 빌드합니다
   - 예를 들어:
     1. `App.js`를 수정하면
     2. webpack이 이를 감지하고
     3. 즉시 새로운 app.js를 생성
     4. Electron 앱이 이 새로운 파일을 로드

4. **빌드 과정 예시:**
   ```plaintext
   [소스 파일들]
   src/js/index.js ----→
   src/js/App.js   ----→  Webpack  ----→ build/js/app.js
   node_modules/   ----→
   ```

5. **만약 `npm run watch`를 실행하지 않으면:**
   - 파일을 수정해도 app.js가 업데이트되지 않음
   - Electron 앱은 여전히 이전 버전의 코드를 보여줌
   - 변경사항이 화면에 반영되지 않음

이것이 바로 우리가 개발할 때:
1. 첫 번째 터미널에서 `npm run watch` (webpack이 파일 변경을 감시)
2. 두 번째 터미널에서 `npm start` (Electron 앱 실행)

이렇게 두 개의 프로세스를 실행해야 하는 이유입니다.

webpack은 마치 통역사처럼 우리가 작성한 최신 JavaScript 코드를 브라우저가 이해할 수 있는 형태로 변환해주는 중요한 도구라고 생각하시면 됩니다.

네, 최신 Electron과 React를 함께 사용하는 좋은 학습 자료들을 찾아보겠습니다: 

Using "Fetch Web Page"

Fetched 2 resources


---


최신 Electron과 React를 함께 사용하는 학습 자료를 몇 가지 추천해드리겠습니다:

1. **공식 문서와 튜토리얼**
   - [Electron 공식 문서](https://www.electronjs.org/docs/latest/) - 가장 최신의 정보를 제공
   - [Electron Forge](https://www.electronforge.io/) - 최신 Electron 앱 개발을 위한 공식 도구

2. **추천 강의 및 학습 경로**:

   a. **Electron Forge로 시작하기** (권장)
   ```bash
   npm init electron-app@latest my-app
   ```
   이 방법은 최신 버전의 보일러플레이트를 자동으로 설정해줍니다.

   b. **온라인 강의 플랫폼**:
   - Udemy: "Electron for Desktop Apps: The Complete Developer's Guide" (2025 업데이트)
   - egghead.io의 "Build a Desktop Application with Electron"
   - YouTube: "Fireship" 채널의 Electron 튜토리얼

3. **최신 Electron + React 프로젝트 구조**:
   ```
   my-electron-app/
   ├── src/
   │   ├── main/
   │   │   └── index.ts          # 메인 프로세스
   │   ├── preload/
   │   │   └── index.ts          # 프리로드 스크립트
   │   └── renderer/
   │       ├── App.tsx           # React 컴포넌트
   │       └── index.tsx         # 렌더러 프로세스
   ├── index.html
   ├── package.json
   └── tsconfig.json
   ```

4. **모던 개발 스택**:
   - TypeScript 사용 (타입 안정성)
   - React 18+ (최신 기능 활용)
   - Vite (빠른 개발 환경)
   - Electron Forge (패키징과 배포)

5. **좋은 GitHub 예제들**:
   - [electron-vite-react](https://github.com/electron-vite/electron-vite-react)
   - [electron-react-boilerplate](https://github.com/electron-react-boilerplate/electron-react-boilerplate)

이러한 최신 자료들을 참고하시면, 옛날 버전의 문제점들(React 18 이전의 렌더링 방식, 보안 설정 문제 등)을 피하고 현대적인 방식으로 개발하실 수 있습니다.

특히, Electron Forge를 사용하면 프로젝트 설정, 빌드, 패키징 등이 자동화되어 있어 개발에만 집중하실 수 있습니다. 새로 시작하시는 경우 이 방법을 추천드립니다.