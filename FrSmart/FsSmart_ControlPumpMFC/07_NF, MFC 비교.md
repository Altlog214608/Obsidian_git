NF / MFC는 펌웨어 쪽에서 장치 제어 모듈의 성격을 구분하는 명칭이에요.  
올려주신 **ControlPumpNF**와 **ControlPumpMFC**를 비교해보면 이런 차이가 있습니다.

---

### 1. 용어 의미

- **NF**
    
    - “Normal Flow” 또는 “Non-Mass Flow Controller” 계열로, 단일 펌프 + 밸브를 순차적으로만 구동하는 구조에 맞춰진 코드입니다.
        
    - 보드에서 각 포트를 개별 ON/OFF 하면서 한 번에 한 향만 낼 때 쓰는 방식.
        
- **MFC**
    
    - “Mass Flow Controller” 계열로, 여러 개의 밸브를 동시에 열고 각각의 듀티(퍼센트) 값을 제어하는 구조입니다.
        
    - 유량 센서(FS scale)와 offset 보정값을 써서, 2개 이상 포트를 원하는 비율로 동시에 발향할 수 있습니다.
        

---

### 2. 구조 차이

|항목|NF 버전|MFC 버전|
|---|---|---|
|발향 방식|순차 발향 (한 포트씩)|동시 발향 가능 (여러 포트 duty 제어)|
|설정값|단일 펌프/밸브 시간·세기만 지정|fscale, offset, 각 포트 duty 테이블 포함|
|MODBUS 명령|단일 포트 주소에 Write|MFC 전용 주소 체계(`MFC_SPA + offset`)로 각 포트 duty 전송|
|펌웨어 데이터 구조|단순 배열/카운터|`S_MFC` 구조체로 보드·라인·duty·보정값 관리|

---

### 3. 질문 주신 “NF → MFC 교체 가능 여부”

- **프레임워크 수준**에서는 ConfigCOM, FunctionSerial, MODBUS 모듈은 그대로 쓰고  
    **ControlPumpNF.c/h** 대신 **ControlPumpMFC.c/h**로 교체하면,  
    프로그램이 MFC 주소 체계와 동시 발향 로직을 사용하게 됩니다.
    
- 단, 교체 시에는:
    
    1. **펌웨어가 MFC 명령 세트를 지원해야** 합니다. (NF 전용 보드에선 MFC 명령이 안 먹힐 수 있음)
        
    2. ConfigProVal.cfg 같은 설정 파일에 **각 포트 duty 보정값(fscale, offset, coeff)**이 맞게 들어 있어야 함.
        
    3. UI나 상위 로직에서 scent_no 하나만 주던 방식을 → 여러 포트 duty를 한 번에 지정하는 방식으로 바꿔야 MFC의 장점이 살아요.
        

---

### 4. 결론

- “NF → MFC” 교체 = **순차 발향 코드**를 **동시 발향 제어 코드**로 바꾸는 것.
    
- 실험에서 말씀하신 fsscale 2.5, offset 41 세팅은 MFC 방식의 보정 로직에 그대로 넣을 수 있고,  
    FrSmart.py 쪽에서 `requestScentWithValues()` 또는 MFC 전용 멀티 duty 전송 함수를 만들면 테스트가 가능합니다.
    

---

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

필요하면 두 파일의 함수 호출 흐름이나 구조체 필드 비교, 주요 함수 예시 및 레지스터 매핑 등을 보다 상세하게도 제공해 드릴 수 있습니다. 궁금한 점 있으시면 언제든 말씀해 주세요.

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/66634ce8-c1f4-4431-91bb-5a76cb668532/ControlPumpNF.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/66634ce8-c1f4-4431-91bb-5a76cb668532/ControlPumpNF.c)
2. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d58a4d75-b0b2-41d9-b35e-10d496144510/ControlPumpNF.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d58a4d75-b0b2-41d9-b35e-10d496144510/ControlPumpNF.h)
3. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/45ee907a-cb21-4f70-b16f-7b7680a3cf9d/ControlPumpMFC.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/45ee907a-cb21-4f70-b16f-7b7680a3cf9d/ControlPumpMFC.c)
4. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d548b631-55d6-491e-ab57-4419dc78cd9d/ControlPumpMFC.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d548b631-55d6-491e-ab57-4419dc78cd9d/ControlPumpMFC.h)