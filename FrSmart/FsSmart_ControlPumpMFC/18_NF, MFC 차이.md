NF와 MFC 파일은 모두 MODBUS 프로토콜을 기반으로 한 펌프 및 솔레노이드 밸브를 제어하는 C 소프트웨어 모듈이지만, 적용 대상과 일부 세부 구조 및 기능에서 차이가 있습니다. 두 파일의 차이점을 아래와 같이 정리해 드립니다.

---

## 1. 적용 대상 및 목적

- **NF 파일**
    
    - 기존에 사용하던 치매 기기에 적용된 제어 모듈입니다.
        
    - 펌프, 밸브, 향료 분사 등의 제어를 위해 설계되어 UI와 연동해 명령을 전달하고 상태를 관리합니다.
        
    - UI에서 받은 발향, 세정, 발향&세정 등 명령과 관련된 구조체(`S_SCENT`)와 제어 기능을 포함합니다.
        
- **MFC 파일**
    
    - 새로 개발하는 향수추천 모델 등 최신 또는 다른 장비에 사용할 목적으로 작성된 제어 모듈입니다.
        
    - 펌프와 솔레노이드 보드를 분리하여 다중 보드, 다중 포트(예: 최대 13보드 × 6포트)의 제어가 가능한 구조를 가집니다.
        
    - 복잡한 보정 계수와 duty 계산, 펌프 및 밸브 포트별 상태 관리 기능이 강화되어 있습니다.
        

---

## 2. 구조체 및 상태 데이터

- **NF**
    
    - 주로 `S_SCENT` 구조체가 중심이며, 명령 ID, 향 번호, 전력 세기, 실행 시간 등 비교적 단순한 정보 관리에 초점.
        
- **MFC**
    
    - `S_MFC` 구조체는 총 솔레노이드 보드 수, 전체 다중 흐름 라인, 각 밸브의 duty/상태 배열, 보정계수, offset 등을 포함하는 보다 복합적이고 정교한 상태 관리.
        

---

## 3. 레지스터 주소 및 제어 방식

- **NF**
    
    - 보드 수, 제품 정보, 펌프 타입, 파워, 시간 등 기본 펌프 제어 레지스터 위주.
        
    - 명령 전송은 단일 및 다중 레지스터 쓰기를 통해 간단한 펌프 및 밸브 제어.
        
- **MFC**
    
    - 세밀한 솔레노이드 보드별 6포트 duty 설정을 위한 연속적 레지스터 주소 관리, 최대 13보드 확장성 제공.
        
    - 펌프 전력, 클린 전력, 주파수 등 다양한 파라미터 조절에 대한 홀딩 레지스터가 별도로 마련되어 있음.
        

---

## 4. 기능 및 복잡도

- **NF** 는 향료 제어용에 맞춰 비교적 간단한 동작과 구조로 설계되어, UI 연동이 직관적이고 특정 향료 및 명령에 집중.
    
- **MFC** 는 다중 보드/포트 구성에 대응하기 위해 복잡한 보정 수식, dynamic 메모리 할당, 다양한 상태 감시 기능을 갖추어 좀 더 범용적이고 확장성 높은 제어를 지원.
    

---

## 5. 통신 및 프로토콜 처리

양쪽 모두 MODBUS 통신 프로토콜을 기반으로 하지만,

- NF는 명령 ID 위주로 명확히 특성화된 데이터 필드를 사용하며,
    
- MFC는 더 많은 상태 값을 처리하기 위한 다양한 입출력 레지스터와 다중 주소계를 사용합니다.
    

---

## 요약

|구분|NF 파일|MFC 파일|
|---|---|---|
|대상 장비|기존 치매 기기용|신형 향수추천 모델용 다중 보드/솔레노이드 제어|
|주요 구조체|단순 명령/파라미터 중심(`S_SCENT`)|복잡한 보정 및 상태 관리 중심(`S_MFC`)|
|제어 대상|향료 번호, 펌프/밸브 제어|다중 보드, 포트별 duty 및 상태, 펌프 전력/주파수 등|
|레지스터 맵|상대적으로 단순|복수 보드·포트 대응, 세밀한 주소 지정|
|기능 수준|UI 중심, 단순 제어|확장성 높은 다중 채널 제어 및 보정 기능 포함|
|통신 방식|MODBUS 기반, 단일/다중 레지스터 쓰기|MODBUS 기반, 다중 채널 및 입출력 레지스터 다수 포함|

---
MFC 파일에서 향(밸브) 제어 명령이 NF 파일처럼 `s_serial.send` 배열에 향 번호, 명령 번호, 시간, 세기 등을 직접 순차적으로 채우는 방식과는 다릅니다. 그리고 헤더에 선언된 `ReadRegister_MFC`나 `WriteSingleReg_MFC` 같은 함수들은 주로 개별 레지스터 읽기/쓰기용으로 보통 향 분사 명령을 직접 내리는 함수는 아닙니다.

아래에 MFC에서 향 분사 명령이 어떤 함수에서 어떻게 만들어지고 보내지는지 정리해 드립니다.

---

## 1. NF와 MFC 구조체 및 명령 전송 차이점

- **NF 경우**
    
    - `s_scent` 구조체에 `cmdid`, `scentno`(향 번호), `power`(전력 세기), `runtime`(시간) 등이 포함됨
        
    - 향 분사 명령은 `WriteMultiRes_NF`에서 `s_serial.send[]` 배열에 명령 ID, 향 번호, 시간, 세기 등 순차적으로 바이트를 채워 넣고 `SendToCom`으로 송신
        
- **MFC 경우**
    
    - `s_mfc` 구조체는 다중 보드·포트 제어용으로 설계되어 있음
        
    - 개별 향번호나 명령 ID를 직접 담는 구조체 필드는 보이지 않고
        
    - 대신 **다중 솔레노이드 보드 단위로 duty값(출력 비율)을 계산해서 전송하는 형태**가 핵심임
        

---

## 2. MFC에선 향 분사 명령이 어디서?

## 핵심 함수: `SetDutyBoard_MFC(int sbdindex)`

- 이 함수는 **특정 솔레노이드 보드(sbdindex, 0부터 시작)의 6개 포트 duty 값을 계산해서 MODBUS `Write Multiple Registers` 명령 패킷을 구성**하여 `s_serial.send[]`에 채움
    
- 기본 패킷 구성 방식


    s_serial.send[0] = MFC_MID;       // 장치 ID (0x01)
	s_serial.send[1] = MBC_WMR;       // MODBUS 함수코드 (0x10, multi write)
	s_serial.send[2,3] = 시작 주소 (MFC_SPA + sbdindex*8)
	s_serial.send[4,5] = 레지스터 개수 (6)
	s_serial.send[6] = 바이트 수 (6*2=12)
	s_serial.send[7~] = 6개의 duty 값 (2바이트씩)
    
- 여기서 duty 값은 내부 보정 공식에 따라 각 밸브별 duty `%`를 계산해 넣음
    
- 이후 `SendToCom(&s_serial)` 호출 → 펌웨어로 전송
    

## 특징

- MFC에서는 **“향 분사 명령 = 각 솔레노이드 보드별 포트 duty 값 설정”** 로 간주됨
    
- 명령 번호나 향 번호 같은 식별자 개념은 `NF` 같은 별도 구조체에 없는 대신,
    
- 프로그램 로직에서 특정 duty 배열 인덱스를 향 번호/밸브 번호로 간주하는 식으로 처리
    

---

## 3. `ReadRegister_MFC`와 `WriteSingleReg_MFC`는?

- 이 함수들은 **MODBUS 레지스터 한 개 또는 여러 개 단위로 읽고 쓰는 일반 함수**입니다.
    
- 즉, `ReadRegister_MFC`는 상태 값을 읽는 데, `WriteSingleReg_MFC`는 명령 코드(`MFC_CMD`) 또는 펌프 세기 등 단일 레지스터 값 변경에 쓰입니다.
    
- 하지만 실제 향 분사와 같이 6채널의 duty를 한꺼번에 전달하는 다중 쓰기는 `SetDutyBoard_MFC`에서 수행함
    

---

## 4. 정리

|구분|NF (기존 치매기기)|MFC (신형 향수추천)|
|---|---|---|
|명령 데이터 담는 구조체|`s_scent` (명령ID, 향번호, 세기, 시간 등)|`s_mfc` (다중 보드/포트 설계, duty/상태/보정 계수)|
|명령 패킷 작성|`WriteMultiRes_NF`에서 cmdid, scentno 등 순서대로 바이트 채움|`SetDutyBoard_MFC`에서 특정 보드 6개 duty 배열을 MODBUS 다중 쓰기로 패킷 작성|
|향 분사 명령 함수|`WriteMultiRes_NF`|`SetDutyBoard_MFC`|
|개별 레지스터 쓰기|`WriteSingleRes_NF`|`WriteSingleReg_MFC` (간단 제어용)|
|명령/데이터 송신|`SendToCom` 호출|`SendToCom` 호출|

---

즉,

- **MFC에서는 향 번호/명령 번호가 직접 패킷에 포함되는 대신, 각 밸브를 duty 배열 인덱스로 관리하며, 여러 밸브 제어정보를 동시에 보드 단위로 MODBUS 다중레지스터 쓰기 명령을 보내는 방식입니다.**
    
- 향 분사 명령은 `SetDutyBoard_MFC()` 함수 안에서 계산, 생성하여 `SendToCom()`으로 송신됩니다.
    

---

필요하면 `SetDutyBoard_MFC()` 함수 내 duty 계산 및 패킷 생성 상세 코드를 설명해 드릴 수 있습니다.  
궁금하시면 알려주세요!

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/66634ce8-c1f4-4431-91bb-5a76cb668532/ControlPumpNF.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/66634ce8-c1f4-4431-91bb-5a76cb668532/ControlPumpNF.c)
2. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d58a4d75-b0b2-41d9-b35e-10d496144510/ControlPumpNF.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d58a4d75-b0b2-41d9-b35e-10d496144510/ControlPumpNF.h)
3. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/45ee907a-cb21-4f70-b16f-7b7680a3cf9d/ControlPumpMFC.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/45ee907a-cb21-4f70-b16f-7b7680a3cf9d/ControlPumpMFC.c)
4. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d548b631-55d6-491e-ab57-4419dc78cd9d/ControlPumpMFC.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d548b631-55d6-491e-ab57-4419dc78cd9d/ControlPumpMFC.h)