네, 첨부해주신 **이미지(멀티 플로우 제어 프로그램 UI 캡처)**를 기반으로 실제 제어 기능, UI의 구조, 입력값과 하드웨어 대응 관계를 꼼꼼하게 분석해서 아래에 정리해드릴게요.

---

## 1. UI(프로그램) 구조와 실제 기능 대응

## **상단: 명령/펌프/시간 입력**

- **Command**: RUN/STOP/CLEAN 등 동작 명령 선택.
    
- **Pump power (%)**: 현재 펌프의 출력 세기(%) 설정 입력.
    
- **Run Time (s)**: 펌프 동작 시간(초) 설정.
    

→ **실제 MFC 제어 명령에서 CMD, 펌프 파워, 런타임 값이 레지스터에 기록됨**  
 - 레지스터(MFC_CMD, MFC_RPW, MFC_POT 등)로 변환되어 펌웨어로 전달.

## **버튼 (Run, Stop, View pressure 등)**

- **Run**: 명령 실행 → 시리얼로 MODBUS 요청 패킷 전송.
    
- **Stop**: 펌프/밸브 중지 명령 → 중지명령 패킷 전송.
    
- **View pressure**: 가스 압력 상태 읽기 요청.
    

---

## 2. **밸브 라인 (line_11~16, line_21~26)**

## **배열 구조/의미**

- **line_11~16**: 첫 번째 보드의 6개 벨브(포트) duty(출력값)
    
- **line_21~26**: 두 번째 보드의 6개 벨브(포트) duty(출력값)
    
- 각 라인의 값(0~100.00): 해당 포트의 펌프 또는 밸브 개방률(%)  
     예시: line_11=40.00이면 포트1이 40%로 오픈
    

## **보드별/라인별**

- 보드 #1: line_11~16 (1~6번 포트)
    
- 보드 #2: line_21~26 (7~12번 포트)  
     ※ 실제 펌웨어 주소맵에서도 보드별 6포트 단위로 duty 레지스터를 관리(0x1086~0x108B, 0x108E~0x1093)
    

## **Total (우측 하단)**

- **Total**: 모든 라인 duty 값의 합산.
    
- **100.00(기준)**: 전체 출력값의 합이 100이 되게끔 배분해야함(= 믹싱 비율 총합).
    

## **예시 동작**

- line_11=40.00, line_13=30.00, line_21=30.00 → total=100.00
    
- 즉, 여러 벨브를 동시에 다른 세기로 제어할 때 전체 sum이 100이 되도록 조정
    

---

## 3. **아래 Response/Port 확인**

- **Response**: 펌웨어에서 온 MODBUS 응답 확인 영역(UI에서 값 보여줌)
    
- **value(0x0FBD) = ...**:
    
    - 이건 실제로 연결된 솔레노이드 보드 수나 센서값 등, MODBUS 레지스터 값을 읽은 결과를 의미
        
    - 연결 성공/통신 확인/상태 표시(원격 상태 진단, 디버그용)
        

---

## 4. 실제 제어 알고리즘과 연계

- **사용자가 위 UI에서 명령/세기/시간 입력 → 라인별 duty 배분 → Run 버튼 클릭**
    
    - 각 보드/포트별 duty값이 구조체(duty 배열)에 저장되고,
        
    - `SetDutyBoard_MFC(보드번호)` 함수로 각 보드 duty값이 MODBUS 멀티 레지스터 쓰기 명령으로 팩킹됨
        
    - pump power, run time 등은 개별 레지스터(`MFC_RPW`, `MFC_POT`)에 기록
        
    - Total이 100 미만/초과면 경고 또는 자동 배분(실제 SW 로직에 따라 다름)
        
    - 시리얼 통신으로 펌웨어에 패킷 전송 → 실제 하드웨어 구동
        

---

## **정리 포인트**

- 라인별 입력값: 보드별/포트별 밸브 duty(%)
    
- Total = 전체 믹싱 합산(100이 되게끔 설정 권장)
    
- 명령, 펌프파워, 런타임: 동작 설정값 → 각각 지정 레지스터로 기록
    
- Response: 통신 상태/응답값 표시(진단/에러/상태 체크)
    
- **실제 제어 흐름이 MFC 소스코드의 duty배열 → SetDutyBoard → MODBUS 패킷 → 펌웨어로 연결됨**
    

---

## **실제 현장 목적**

- 다양한 향료/라인(12개)을 원하는 비율(합 100)로 분사
    
- 각 라인은 별도의 밸브(포트)와 직결되고, 명령/파워/시간/비율 전부 소프트웨어에서 실시간 조정 가능
    
- 실질적 하드웨어 제어는 MFC MODBUS 프로토콜 패킷 통해 구동
    

---

필요하면 **이 UI 입력값 ↔ 실제 MODBUS 레지스터 ↔ MFC 소스코드 흐름**을 표나 시퀀스 다이어그램으로 정리해서 보여드릴 수 있습니다.

추가적으로, 실제 사용중 WARN/ERROR, 자동합산/배분 로직, 펌웨어 응답 예시 등도 설명해드릴 수 있습니다!  
궁금한 부분 더 짚어주시면 바로 이어서 설명드릴게요.

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/66634ce8-c1f4-4431-91bb-5a76cb668532/ControlPumpNF.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/66634ce8-c1f4-4431-91bb-5a76cb668532/ControlPumpNF.c)
2. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d58a4d75-b0b2-41d9-b35e-10d496144510/ControlPumpNF.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d58a4d75-b0b2-41d9-b35e-10d496144510/ControlPumpNF.h)
3. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/45ee907a-cb21-4f70-b16f-7b7680a3cf9d/ControlPumpMFC.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/45ee907a-cb21-4f70-b16f-7b7680a3cf9d/ControlPumpMFC.c)
4. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d548b631-55d6-491e-ab57-4419dc78cd9d/ControlPumpMFC.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d548b631-55d6-491e-ab57-4419dc78cd9d/ControlPumpMFC.h)
5. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/72041696/452060c7-714d-4d18-9200-0f0f07eb7fda/seukeurinsyas-2025-08-13-093055.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/72041696/452060c7-714d-4d18-9200-0f0f07eb7fda/seukeurinsyas-2025-08-13-093055.jpg)
6. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/e9851302-f6cc-4813-9436-999ace8ca1c5/ConfigCom.cfg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/e9851302-f6cc-4813-9436-999ace8ca1c5/ConfigCom.cfg)
7. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/2c2cd960-faac-4b77-9608-a9da61c6678e/ConfigProVal.cfg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/2c2cd960-faac-4b77-9608-a9da61c6678e/ConfigProVal.cfg)