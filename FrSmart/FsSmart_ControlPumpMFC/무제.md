COM1 포트 발향 세번

2675 14/08/2025 13:15:28 IRP_MJ_WRITE DOWN  01 10 10 68 00 08 10 00 01 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 5d 44  ...h........(.(.....�.�]D 25 25 COM1  
2676 14/08/2025 13:15:28 IRP_MJ_WRITE UP STATUS_SUCCESS 01 10 10 68 00 08 10 00 01 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 5d 44  ...h........(.(.....�.�]D 25  COM1  


2685 14/08/2025 13:15:29 IRP_MJ_WRITE DOWN  01 10 10 68 00 08 10 00 01 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 5d 44  ...h........(.(.....�.�]D 25 25 COM1  
2686 14/08/2025 13:15:29 IRP_MJ_WRITE UP STATUS_SUCCESS 01 10 10 68 00 08 10 00 01 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 5d 44  ...h........(.(.....�.�]D 25  COM1  


2695 14/08/2025 13:15:29 IRP_MJ_WRITE DOWN  01 10 10 68 00 08 10 00 01 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 5d 44  ...h........(.(.....�.�]D 25 25 COM1  
2696 14/08/2025 13:15:29 IRP_MJ_WRITE UP STATUS_SUCCESS 01 10 10 68 00 08 10 00 01 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 5d 44  ...h........(.(.....�.�]D 25  COM1  


---

## 본문 메시지 해석

## 원본(HEX):

text

`01 10 10 68 00 08 10 00 01 00 01 00 28 00 28 00 03 00 04 01 F4 01 F4 5D 44`

(모니터링 툴에서 DOWN/UP, STATUS_SUCCESS 표시됨)

---

## 1. MODBUS 프로토콜 패킷 구조

- **01** : 슬레이브 주소 (장치 주소, 보통 메인보드 0x01)
    
- **10** : 함수 코드 (0x10 = Write Multiple Registers)
    
- **10 68** : 시작 주소 (0x1068 = 4200) → 명령/파라미터 첫번째 레지스터
    
- **00 08** : 레지스터 개수 (0x08 = 8개 레지스터에 값을 씀)
    
- **10** : 바이트 수 (16, 즉 8레지스터×2바이트)
    

## 2. 데이터 값(레지스터에 쓸 내용, 2바이트=1레지스터씩 순서대로)

- **00 01** : 명령 번호 (아마도 발향 RUN/EMIT 명령, 1)
    
- **00 01** : 향 번호 (1번 향)
    
- **00 28** : 펌프 파워 (0x28=40, 즉 40% 파워)
    
- **00 28** : 세정(클린) 펌프 파워 (0x28=40, 40% 파워)
    
- **00 03** : 발향 시간 (0x03=3초)
    
- **00 04** : 세정 시간 (0x04=4초)
    
- **01 F4** : 발향 delay (0x01F4=500ms)
    
- **01 F4** : 세정 delay (0x01F4=500ms)
    

## 3. CRC (CheckSum)

- **5D 44** : 전체 데이터의 CRC16 (MODBUS RTU 프로토콜 규격)
    

---

## 의미 요약

- 발향/세정 TEST 명령 하나를 보내면서
    
    - 향 번호(1), 발향(40%), 세정(40%), 발향 3초, 세정 4초, 각각 딜레이 500ms라는 파라미터를 동시에 전송합니다.
        
- 소프트웨어(EXE)가 PC의 OS(COM 포트 드라이버)를 통해 실제 보드 장치로 명령을 전송
    
- "글자 깨짐"처럼 보이는 부분은 원래 바이트 스트림이기 때문에 정상적인 현상이고, CRC 등 특수 바이트도 포함되어 있음.
    

---

## 실제 동작 구조와 대응

- MODBUS "Write Multiple Registers" 명령이 NF/MFC 파일의 `WriteMultiRes_NF()`/`WriteMultiReg_MFC()`에서 패킹되는 구조와 완전히 일치합니다.
    
- 보드(펌웨어)는 이 패킷을 해석해서, 지정된 명령/향/파라미터값대로 밸브와 펌프를 제어합니다.
    

---

## 결론

- **당신이 현재 툴에서 본 메시지와 송신(Write) 로그는 실제 소프트웨어가 명령을 만들어 OS(COM포트)를 통해 물리적으로 장치에 TX(송신)한 데이터를 100% 정확하게 캡처한 내용**입니다.
    
- HEX 코드 해석 결과: "1번 향, 40% 발향, 40% 세정, 3초/4초, 500ms딜레이" 명령이 보내진 것.[kuwhawha.tistory+2](https://kuwhawha.tistory.com/286)
    
- CRC 등 특수 바이트 때문에 일부 몽타주나 글자가 깨져보이는 건 정상.
    


COM4포트 발향-세정-발향&세정

211 14/08/2025 13:20:07 IRP_MJ_WRITE DOWN  01 10 10 68 00 08 10 00 01 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 5d 44  ...h........(.(.....�.�]D 25 25 COM4  
217 14/08/2025 13:20:07 IRP_MJ_WRITE UP STATUS_TIMEOUT     COM4  
233 14/08/2025 13:20:07 IRP_MJ_WRITE DOWN  01 10 10 68 00 08 10 00 02 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 1e 45  ...h........(.(.....�.�.E 25 25 COM4  
239 14/08/2025 13:20:07 IRP_MJ_WRITE UP STATUS_TIMEOUT     COM4  
259 14/08/2025 13:20:07 IRP_MJ_WRITE DOWN  01 10 10 68 00 08 10 00 04 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 98 47  ...h........(.(.....�.�G 25 25 COM4  
265 14/08/2025 13:20:07 IRP_MJ_WRITE UP STATUS_TIMEOUT     COM4  


---

## 원본 메시지 분석

각 로그는 PC에서 COM4 포트로 MODBUS "Write Multiple Registers" 명령을 한 번씩 보낸 뒤, 보드 응답(여기서는 타임아웃)이 된 경우입니다.  
각 메시지의 내용을 해석하면 아래와 같습니다.

## 1. 첫 번째 명령 (`IRP_MJ_WRITE DOWN`)

text

`01 10 10 68 00 08 10 00 01 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 5d 44`

- **01** : 장치/슬레이브 주소 (메인보드, 1)
    
- **10** : MODBUS 함수코드 (0x10, Write Multiple Registers)
    
- **10 68** : 시작 레지스터 주소 (0x1068) → 명령 및 파라미터 레지스터
    
- **00 08** : 쓰기 레지스터 수 (8개)
    
- **10** : 데이터 길이 (16바이트 = 8레지스터×2바이트)
    
- **00 01** : 명령 코드 (1=발향)
    
- **00 01** : 향 번호(1)
    
- **00 28** : 펌프 파워 (0x28=40, 40%)
    
- **00 28** : 세정 펌프 파워 (40%)
    
- **00 03** : 발향 기간 (3초)
    
- **00 04** : 세정 기간 (4초)
    
- **01 f4** : 발향 딜레이 (0x01F4 = 500ms)
    
- **01 f4** : 세정 딜레이 (500ms)
    
- **5d 44** : CRC (체크섬)
    

## 2. 두 번째 명령

text

`01 10 10 68 00 08 10 00 02 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 1e 45`

- **00 02** : 명령 코드 (2=세정)
    
- 나머지 파라미터 동일
    

## 3. 세 번째 명령

text

`01 10 10 68 00 08 10 00 04 00 01 00 28 00 28 00 03 00 04 01 f4 01 f4 98 47`

- **00 04** : 명령 코드 (4=발향&세정)
    
- 나머지 파라미터 동일
    

---

## 각 메시지 개요

|명령번호|의미|향번호|펌프파워|세정파워|발향시간|세정시간|발향딜레이|세정딜레이|
|---|---|---|---|---|---|---|---|---|
|1|발향|1|40%|40%|3초|4초|500ms|500ms|
|2|세정|1|40%|40%|3초|4초|500ms|500ms|
|4|발향&세정|1|40%|40%|3초|4초|500ms|500ms|

---

## 뒷부분의 STATUS_TIMEOUT 로그

- **IRP_MJ_WRITE UP STATUS_TIMEOUT**:  
    PC에서 명령을 보냈지만, 연결된 장치(보드)가 없거나 응답이 오지 않아(혹은 응답 대기시간 초과) TIMEOUT 발생  
    → 정상적으로 메시지는 송신되었지만 응답 없음
    

---

## 결론

- 세 번 각각 **발향, 세정, 발향&세정** 명령(각각 명령번호 1, 2, 4)을 PC COM4에서 실제로 보냈고
    
- 모든 명령에 대해 향번호=1, 파워=40(발향/세정 동일), 기간=3/4초, 딜레이=500ms로 세팅됨
    
- 장치에서 응답이 없거나 연결 안 된 경우로 모두 TIMEOUT이 발생함
    
- HEX 바이트 스트림 구조와 명령 내용은 이전에 분석했던 NF 소스의 MODBUS 패킷 구조와 완전히 일치합니다.
    

---

## 한눈정리

- "EXE에서 명령을 송신하면 위와 같은 바이트 패킷이 COM포트로 보내지고, 실제 장치가 연결되어 있으면 응답 패킷이 돌아올 것"
    
- "TIMEOUT은 장치 미연결 또는 응답 지연 시 정상적으로 발생하는 상태"
    

궁금한 바이트 필드나, 응답이 왔을 때의 해석 방식 등 더 알고 싶으신 점 있으면 추가로 질문 주세요!

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/66634ce8-c1f4-4431-91bb-5a76cb668532/ControlPumpNF.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/66634ce8-c1f4-4431-91bb-5a76cb668532/ControlPumpNF.c)
2. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d58a4d75-b0b2-41d9-b35e-10d496144510/ControlPumpNF.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d58a4d75-b0b2-41d9-b35e-10d496144510/ControlPumpNF.h)
3. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/45ee907a-cb21-4f70-b16f-7b7680a3cf9d/ControlPumpMFC.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/45ee907a-cb21-4f70-b16f-7b7680a3cf9d/ControlPumpMFC.c)
4. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d548b631-55d6-491e-ab57-4419dc78cd9d/ControlPumpMFC.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d548b631-55d6-491e-ab57-4419dc78cd9d/ControlPumpMFC.h)
5. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/72041696/452060c7-714d-4d18-9200-0f0f07eb7fda/seukeurinsyas-2025-08-13-093055.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/72041696/452060c7-714d-4d18-9200-0f0f07eb7fda/seukeurinsyas-2025-08-13-093055.jpg)
6. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/e9851302-f6cc-4813-9436-999ace8ca1c5/ConfigCom.cfg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/e9851302-f6cc-4813-9436-999ace8ca1c5/ConfigCom.cfg)
7. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/2c2cd960-faac-4b77-9608-a9da61c6678e/ConfigProVal.cfg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/2c2cd960-faac-4b77-9608-a9da61c6678e/ConfigProVal.cfg)