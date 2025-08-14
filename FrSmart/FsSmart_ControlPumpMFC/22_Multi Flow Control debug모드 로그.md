
---

## MODBUS 송신 패킷 해석

## 1. `send[ 8] = 01 04 0F A0 00 04 F2 FF`

- **01** : 슬레이브 ID (장치 주소, 일반적으로 1 = 메인보드)
    
- **04** : function code (0x04, Read Input Registers; 입력 레지스터 읽기 명령)[kuwhawha.tistory+2](https://kuwhawha.tistory.com/286)
    
- **0F A0** : 시작 레지스터 주소 (0x0FA0, 즉 firmware version 주소)
    
- **00 04** : 읽을 레지스터 개수 (4 word)
    
- **F2 FF** : CRC (체크섬)
    

## 의미

- 메인보드의 펌웨어 버전 정보를 4개 레지스터(총 8바이트)만큼 읽으라는 명령을 전송한 것.
    

---

## 2. `send[ 8] = 01 04 0F BD 00 01 A2 FA`

- **01** : 슬레이브 ID
    
- **04** : function code (Read Input Registers)
    
- **0F BD** : 시작 주소 (0x0FBD, NF_NSB/MFC_NSB) → 솔레노이드 보드 개수 읽는 레지스터ControlPumpNF.h+1
    
- **00 01** : 1 word(2바이트) 읽기
    
- **A2 FA** : CRC
    

## 의미

- 연결된 솔레노이드 보드 개수를 1 word(2바이트)만큼 읽으라는 명령을 보낸 것.
    

---

## 3. `nbr of sol. board = 0`

- 방금 위 명령(0x0FBD) 읽기 요청에 대한 응답이 "없음"(read = ), 혹은 0으로 해석됨.
    
- 즉 **현재 연결된 솔레노이드 보드가 감지되지 않거나 0개로 인식**된 상태.
    

---

## 4. `send[ 8] = 01 04 0F BF 00 01 03 3A`

- **01** : 슬레이브 ID
    
- **04** : function code (Read Input Registers)
    
- **0F BF** : 시작 주소 (0x0FBF) → carrier gas pressure 등, 압력 센서 값 읽는 레지스터ControlPumpMFC.h+1
    
- **00 01** : 1 word 읽기
    
- **03 3A** : CRC
    
- 이 이하의 반복은 동일 주소, 레지스터 수로 연속적으로 압력 관련 센서 값을 읽으려는 시도임.
    

---

## 전체 로그 분석 및 의미

- PC 프로그램이 시리얼 연결 후, **메인보드, 솔레노이드 보드, 압력 센서 상태 등 기본 정보를 계속 polling**(조회)하는 루프가 동작 중.
    
- 각 패킷은 MODBUS RTU 규격에 맞춘 "입력 레지스터 읽기(write)" 명령이며,  
    슬레이브(board)가 연결돼 있다면 read에 응답 데이터가 채워졌을 것.
    
- "**nbr of sol. board = 0**"은 **솔레노이드 보드가 인식되지 않음**; 보드 미연결 상태이거나 전원이 꺼져 있을 때 발생.
    

---

## 결론

- 각각의 send 로그는
    
    - 0x0FA0: 펌웨어 버전 읽기
        
    - 0x0FBD: 솔레노이드 보드 개수 읽기
        
    - 0x0FBF: 캐리어가스 압력 등 센서값 읽기
        
- read 값이 비어 있고, "nbr of sol. board = 0"이 뜨는 것은 **장치 연결 혹은 응답 없음(보드가 연결되지 않았음/미동작)**을 의미합니다.
    

