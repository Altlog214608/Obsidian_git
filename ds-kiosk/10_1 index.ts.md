GitHub Copilot

짧게 요약 — 핵심 개념과 실무 팁만.

1) electronAPI vs api
- electronAPI  
  - 이 프로젝트에서 import 한 `electronAPI`는 외부 패키지(@electron-toolkit/preload)가 미리 제공하는 "편의 브리지"입니다. 일반적으로 안전하게 노출해도 되는 Electron 기능(예: 제한된 ipc 래퍼, shell/openExternal 등)을 모아 둔 것이라 생각하면 됩니다. 즉, 범용적인 Electron 기능 세트(툴킷)입니다.
- api  
  - `api`는 프로젝트가 직접 정의해서 렌더러에 노출하려는 "앱 전용" 함수/메서드 집합입니다. 예: 설정 읽기, 파일 열기, 특정 네이티브 동작 등. 현재는 빈 객체라 실제 기능은 없습니다.

2) process.contextIsolated는 무엇인가? 왜 쓰나?
- 의미  
  - `process.contextIsolated`는 Electron이 브라우저(렌더러)에서 Context Isolation을 사용 중인지(true/false) 알려주는 플래그입니다.
  - Context Isolation = preload 스크립트가 renderer의 글로벌 스코프와 분리된 별도의 JS 컨텍스트에서 실행되는 설정.
- 왜 중요한가(보안 이유)  
  - 분리되어 있으면 렌더러(웹 코드)에서 Node API나 preload에서 만든 내부 객체를 직접 조작할 수 없습니다. XSS 등으로 임의 스크립트가 실행되어도 시스템 API에 접근하지 못하게 막습니다.
  - 안전한 방식은 preload에서 필요한 API만 `contextBridge.exposeInMainWorld('name', api)`로 명시적으로 노출하는 것입니다. (화이트리스트 방식)
- false(분리 안함)의 문제  
  - preload에서 `window.*`로 직접 노출하면 렌더러가 모든 노출된 객체를 임의로 바꿀 수 있고, 외부 스크립트가 Node 접근을 악용할 수 있어 위험합니다.

3) 실무 권장 패턴 (간단 예시)
- preload: 명시적 메서드 노출 (ipcRenderer.invoke 사용 권장)
````typescript
import { contextBridge, ipcRenderer } from 'electron'

const api = {
  getConfig: () => ipcRenderer.invoke('get-config'),
  openFile: (opts: { multi?: boolean }) => ipcRenderer.invoke('open-file', opts)
}

contextBridge.exposeInMainWorld('api', api)
````
- renderer 사용:
````ts
// window.api.getConfig().then(cfg => ...)
````

4) 보안/유지보수 팁 (짧게)
- 항상 BrowserWindow 생성 시: contextIsolation: true, nodeIntegration: false.  
- preload에서 절대 Node 모듈(예: fs)을 renderer에 직접 노출하지 말기. 필요한 기능만 함수로 래핑해서 노출.  
- IPC 채널은 이름을 고정하고 화이트리스트로 관리. main 쪽 핸들러에서도 입력 검증 수행.  
- 빈 `api = {}`는 제거하거나 실제 메서드 추가 — 빈 노출은 혼동 유발.  
- 타입 선언(.d.ts)으로 window.api/ window.electron 인터페이스를 정의해 두면 renderer에서 안전하게 사용 가능.

원하면
- 지금 `api`에 추가할 추천 메서드 목록(예: getAppVersion, readConfig, openFileDialog)과 그에 맞는 preload + main handler + d.ts 샘플 코드를 바로 생성해 드릴게요. 어떤 기능부터 추가할까요?