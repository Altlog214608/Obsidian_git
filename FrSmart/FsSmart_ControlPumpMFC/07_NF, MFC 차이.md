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

