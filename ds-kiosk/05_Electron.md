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