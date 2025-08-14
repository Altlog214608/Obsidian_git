

---

## 1. SW → 시리얼 전송 (FrSmart / dsComm)

- **FrSmart.py**에서 `requestScentNo()` 같은 함수가 호출됩니다.
    
- 내부에서 **dsComm.py**의 `sendMsgForEmitClean()` 같은 메시지 생성 함수를 부릅니다.
    
- 이 함수가 MODBUS 규격에 맞는 바이트 패킷(슬레이브 ID, 기능 코드, 주소, 데이터, CRC16)을 만들고 `write_data()`로 시리얼 포트에 씁니다.
    
- 이 시점에서 PC OS의 시리얼 드라이버를 거쳐 장치(MCU)로 명령이 전송됩니다.
    

---

## 2. MCU → SW 응답 (ControlPumpMFC / DecodeData_MFC)

- 장치가 MODBUS 응답 패킷을 보내면, PC 쪽 시리얼 수신 버퍼에 들어갑니다.
    
- 수신된 바이트들은 `s_serial.read[]`라는 전역 버퍼에 저장됩니다.
    
- **DecodePacket()**
    
    - 수신된 패킷이 에러(기능코드 + 0x80)인지 확인.
        
    - 정상이라면 `DecodeData_MFC()`로 넘김.
        
- **DecodeData_MFC(add, rdreg)**
    
    1. `s_serial.read`에서 데이터 부분만 잘라 `rdata[]` 배열로 복사.
        
    2. `switch(add)` 문으로 MODBUS의 시작 주소(`add`)를 판단.
        
        - **MFC_NSB**: 솔레노이드 보드 개수를 읽어 `s_mfc.tnsbd`, `tnmfl` 설정, duty/status/dutyzero 메모리 할당.
            
        - **MFC_SPS**: 보드 포트 상태 값을 `s_mfc.status[]`에 저장.
            
        - **MFC_SPA**: 포트 duty 값을 `s_mfc.duty[]`에 저장.
            
        - **MFC_CGP**: 캐리어 가스 압력값 처리.
            
    3. 즉, 여기서 `s_mfc.duty[]`나 `s_mfc.status[]`가 **실시간 응답 값으로 업데이트**됩니다.
        

---

## 3. 핵심 포인트

- **`dutyzero`**: 설정 파일(ConfigProVal.cfg)에서 읽어오는 **무유량(0 flow) 기준 duty 값**입니다. 초기에는 설정 파일 값으로 채워지고, 실시간 응답과는 별개입니다.
    
- **`duty`**: 실제 하드웨어 상태를 반영하는 값. `DecodeData_MFC()`에서 MCU 응답 패킷을 읽어 갱신됩니다.
    
- SW에서 보낸 명령 → MCU 실행 → MCU 응답 → `s_mfc.duty` 업데이트 구조입니다.
    

---

원하시면 제가 **이 송수신 과정을 시퀀스 다이어그램**으로 그려서  
FrSmart → dsComm → 시리얼 드라이버 → MCU → ControlPumpMFC 흐름을 한 눈에 보이게 만들어 드릴까요?  
그렇게 하면 데이터가 어디서 생성되고, 어디서 갱신되는지 더 직관적으로 보실 수 있습니다.