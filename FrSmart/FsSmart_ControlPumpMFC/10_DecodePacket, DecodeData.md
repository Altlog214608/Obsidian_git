좋아요 👍  
올려주신 `DecodeData_MFC()`와 `DecodePacket()` 함수는 **펌웨어에서 온 응답 패킷을 해석해서, PC 프로그램 내부 변수나 UI에 반영하는 역할**을 합니다.  
하나씩 차근차근 풀어보겠습니다.

---

## 1. `int DecodeData_MFC(unsigned short add, unsigned short rdreg)`

## 함수 목적

- **MODBUS 응답 버퍼**(`s_serial.read[]`)에서  
    지정한 주소(`add`)부터 읽어온 레지스터 값을 **정수/실수로 변환**해서  
    `s_mfc` 구조체에 저장하고, 필요하면 UI도 업데이트하는 함수입니다.
    

---

## 동작 순서

1. **변수 준비**
    
	
    ```c
    cunsigned short *rdata; 
    rdata = malloc(rdreg * sizeof(short)); 
    index = 3; 
    rindex = 0; 
    nodata = s_serial.read[2] / 2; // 수신 바이트 개수 → 레지스터 개수
    ```
    
    - 응답 패킷 첫 3바이트: `[슬레이브ID, 함수코드, 데이터바이트수]`
        
    - `nodata`: 읽은 바이트 개수를 2로 나눈 값 = 레지스터 개수
        
2. **디버그 문자열에 주소 출력 준비**
	```c
	sprintf(datastr,"value[0x%04X] = ", add);   
	```
    
    
    
	    
3. **응답 데이터 바이트 → 16비트 레지스터 배열 변환**
    ```c
    for(i=0; i < nodata; i++) {     
    rdata[rindex] = (short)s_serial.read[index]; index++;   
	rdata[rindex] <<= 8;    
	rdata[rindex] |= (short)s_serial.read[index]; index++;    
	sprintf(datastr,"%s %d", datastr, rdata[rindex]);    rindex++; 
	}
    ```
    
    
    
    - MODBUS는 **빅엔디안**이므로 high byte 먼저, low byte 나중
        
    - `s_serial.read[]`에서 두 바이트씩 읽어 하나의 `unsigned short` 값으로 합침
        
    - `datastr`에 값들을 이어 붙여서 디버그 문자열 완성
        
4. **UI 업데이트 (단, 압력 값(MFC_CGP)는 그래프에만 표시)**
	```c
    if (add != MFC_CGP)     SetCtrlVal(control_handle, CPL_RES, datastr);
    ```
    
    - MFC_CGP(Carrier Gas Pressure) 데이터일 때는 UI 텍스트에 대신 그래프를 그림
        
5. **주소별 데이터 처리 분기**
    ```c
    switch (add) {     
	    case MFC_NSB: // 솔레노이드 보드 개수        
	    s_mfc.tnsbd  = rdata[0];          // 보드 개수        
	    s_mfc.tnmfl  = s_mfc.tnsbd * 6;   // 전체 라인(포트) 수        
	    // 동적 메모리 할당: duty[], status[], lhindex[], dutyzero[]        
	    break;     
	    case MFC_SPS: // 각 포트 상태        
	    for(i=0; i < s_mfc.tnsbd * 6; i++)            
	    s_mfc.status[i] = rdata[i];        
	    break;     
	    case MFC_SPA: // 각 포트 duty 값        
	    for(i=0; i < s_mfc.tnsbd * 8; i++)           
	     s_mfc.duty[i] = rdata[i];        
	     break;     
	    case MFC_CGP: // Carrier Gas Pressure        
	    ddata[0] = (double)rdata[0] * 0.1; // 소수로 변환(kPa 단위)   
	    SetCtrlVal(acq_handle, DPNL_PRES, ddata[0]); // UI 표시        
	    PlotStripChart(acq_handle, DPNL_SCHART, ddata, 1, 0, 0, VAL_DOUBLE); // 차트에 플롯        
	    break; 
	    }
    ```
    
    
    
    
6. **할당 메모리 해제**
    ```c
    free(rdata);
    ```
    
    

---

📌 **정리**  
`DecodeData_MFC()` = **받은 응답 패킷 → 값 변환 → 구조체 업데이트 & UI 반영**

- MODBUS 2바이트 단위 데이터를 C의 `unsigned short` 정수로 변환
    
- 읽은 주소(`add`)에 따라 적절히 구조체 변수(`s_mfc`)에 저장
    
- 경우에 따라 UI 텍스트나 차트 업데이트
    

---

## 2. `void DecodePacket(void)`

## 함수 목적

- 수신한 패킷이 **정상 응답인지 에러 응답인지 판별**하고,
    
- 에러일 경우 `Error_MFC()` 호출하여 처리.
    

---

## 동작 순서

1. **UI에 원시 수신 데이터 표시**
    
    c
    
    `SetCtrlVal(control_handle, CPL_RES, s_serial.read);`
    
    - `s_serial.read` 버퍼 자체를 텍스트 표시 (아마 디버그용 Raw Data 확인)
        
2. **MODBUS 에러 응답 체크**
    
    c
    
    `if (s_serial.read[1] & 0x80) {     errindex = s_serial.read[1] - 0x80;    Error_MFC(errindex); }`
    
    - MODBUS 프로토콜에서 `함수코드 | 0x80` 형태면 **예외 응답(Exception Response)**
        
        - 예: 요청이 `0x03`인데 응답이 `0x83`이면 Function Code 3에서 에러 발생
            
    - `errindex` = 실제 에러 코드 값
        
    - `Error_MFC()`를 호출해 해당 에러 코드에 맞는 메시지를 표시
        
3. **정상 응답이면 그냥 리턴**
    
    - 데이터 해석은 별도로 `DecodeData_MFC()`에서 진행
        

---

📌 **정리**  
`DecodePacket()` = **수신 패킷의 에러 여부 판정 + 에러 처리 호출**

- MODBUS 예외 응답이면 에러 처리 함수 호출
    
- 정상 응답이면 다음 단계에서 `DecodeData_MFC()`로 해석을 진행
    

---

## 두 함수의 관계

1. **데이터 수신**
    
    - `ReadFromCom()`이나 pySerial의 `readline()` 같은 함수로 `s_serial.read[]` 버퍼 채움
        
2. **1차 처리 (DecodePacket)**
    
    - 예외 응답인지 판별
        
    - 에러면 `Error_MFC()` 호출 후 종료
        
3. **2차 처리 (DecodeData_MFC)**
    
    - 정상 데이터라면 `add`(주소)에 따라 값을 변환/저장
        
    - `s_mfc` 구조체 업데이트
        
    - UI 컨트롤 업데이트
        

---

💡 **쉽게 비유하기**

- `DecodePacket()`은 "이 편지가 반송인가 정상 우편물인가?"만 보고 반송이면 반송 사유 안내문 띄움.
    
- `DecodeData_MFC()`는 우편물의 내용을 봉투 뜯어서 실제 데이터로 정리하고, 필요한 데이터 창에 기록하거나 그래프에 반영하는 역할.
    

---

원하시면, 제가 **MODBUS Request → 펌웨어 응답 → DecodePacket → DecodeData_MFC** 과정까지 포함한 전체 시퀀스를 그림으로 그려드릴 수 있습니다. 그러면 송수신·해석 흐름이 한눈에 들어오실 거예요.  
그렇게 진행해 드릴까요?