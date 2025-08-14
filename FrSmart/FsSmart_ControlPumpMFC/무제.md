네! `ConfigProVal.cfg` 파일에서 **Proportional Valve Configuration** 부분은 솔레노이드 밸브(비례밸브) 제어용 보정값 설정을 의미하며, 실제 소프트웨어에서 각 밸브의 duty(개방률) 값을 하드웨어가 정확하게 동작하도록 보정해주는 기준이 됩니다.

아래에 각 설정의 의미와 용도를 해석해서 정리해드릴게요.

---

## 1. fscale (full scale)

- `fscale = 2.5`
    
    - 입력 duty(비율) 값을 보정계 산출할 때 사용되는 **입력 스케일 기준값**.
        
    - 0~100% 입력 기준에서 환산할 때, 내부 보정공식에서 배율로 활용됨.
        

## 2. offset (오프셋)

- `offset = 41.0`
    
    - **보정 곡선의 0점 위치 결정**.
        
    - 0% 개방에 해당하는 실제 하드웨어 duty 값 (즉, 밸브를 완전히 닫을 때 컨트롤러가 출력하는 duty 신호의 기준점).
        
    - 소프트웨어에서 곡선 보정 공식에 항상 포함되는 값.
        

## 3. coeff (다항식 계수)

- `coeff = 41.2590, 2.4498, 13.7190, -9.1858, 1.3973, 0.6380`
    
    - **5차 다항식 보정 계수(a5,a4,a3,...a0)**.
        
    - 실제 밸브의 동작 곡선이 완전히 선형이 아니라서, 개방 비(%) 입력을 하드웨어 신호(duty)로 바꿀 때 오차를 줄이기 위해 다항식으로 보정함.
        
    - 아래와 같은 수식에 적용
        
        text
        
        `fduty = a5*x^5 + a4*x^4 + a3*x^3 + a2*x^2 + a1*x + a0`
        
        (x: 소프트웨어 내부 duty %, fscale과 offset 적용)
        

## 4. zduty (zero flow, absolute duty value)

- **1st manifold**
    
    - `zduty = 39.0, 44.0, 36.0, 43.0, 38.0, 41.0`
        
        > 첫 번째 매니폴드(솔레노이드 보드)의 6개 밸브(포트)별 "완전 닫힘(제로 유량)" 상태일 때 실제 하드웨어가 요구하는 duty(신호) 값.
        
- **2nd manifold**
    
    - `zduty = 38.0, 41.0, 37.0, 39.0, 38.0, 39.0`
        
        > 두 번째 매니폴드(솔레노이드 보드) 6개 밸브(포트)별 완전 닫힘 시의 기준 duty 값.
        
- 실제로 소프트웨어에서 각 채널의 duty를 제어할 때
    
    - 입력값(duty%)을 fscale/offset/coeff 다항식으로 보정한 뒤
        
    - 각 보드/포트별 zduty 값만큼 더하거나, 기준으로 삼아 실제 하드웨어에 맞게 duty 신호를 출력함.
        
    - → 시스템의 “xxx % 개방”이 실제 밸브에서는 "zduty + fduty" 식으로 변환됨.
        

---

## 🔹 정리

- **이 파일은 모든 밸브의 개방/닫힘 보정 기준(곡선, 오프셋, 스케일, 채널별 zero 기준값)을 정의한다.**
    
- 소프트웨어의 duty(%) 입력 → 실제 밸브의 물리적 동작으로 정밀하게 대응하도록 한다.
    
- zduty는 각 포트마다 실제 하드웨어의 '완전히 닫힘'을 보장하기 위한 값.
    
- coeff/fscale/offset는 0%~100% 제어 범위를 제대로 재현하도록 곡선 보정의 핵심.
    

---

**예시 계산**  
만약 소프트웨어에서 포트1에 대해 duty값을 50%로 입력하면,

1. 50을 fscale, offset, coeff을 적용해 다항식으로 변환
    
2. 포트1의 zduty(39.0)를 더해 하드웨어 신호 값 확정(예: 50% → 41.2590*X^5 + ... + zduty)
    
3. 이 값이 실제 MODBUS 패킷의 바이트 필드에 채워져 펌웨어로 전송됨
    

---

## 궁금한 보정공식이나 실제 소프트웨어/코드 적용 예시, 추가 설명 필요하시면 언제든 말씀하세요!

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/66634ce8-c1f4-4431-91bb-5a76cb668532/ControlPumpNF.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/66634ce8-c1f4-4431-91bb-5a76cb668532/ControlPumpNF.c)
2. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d58a4d75-b0b2-41d9-b35e-10d496144510/ControlPumpNF.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d58a4d75-b0b2-41d9-b35e-10d496144510/ControlPumpNF.h)
3. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/45ee907a-cb21-4f70-b16f-7b7680a3cf9d/ControlPumpMFC.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/45ee907a-cb21-4f70-b16f-7b7680a3cf9d/ControlPumpMFC.c)
4. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d548b631-55d6-491e-ab57-4419dc78cd9d/ControlPumpMFC.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d548b631-55d6-491e-ab57-4419dc78cd9d/ControlPumpMFC.h)
5. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/72041696/452060c7-714d-4d18-9200-0f0f07eb7fda/seukeurinsyas-2025-08-13-093055.jpg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/72041696/452060c7-714d-4d18-9200-0f0f07eb7fda/seukeurinsyas-2025-08-13-093055.jpg)
6. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/e9851302-f6cc-4813-9436-999ace8ca1c5/ConfigCom.cfg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/e9851302-f6cc-4813-9436-999ace8ca1c5/ConfigCom.cfg)
7. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/2c2cd960-faac-4b77-9608-a9da61c6678e/ConfigProVal.cfg](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/2c2cd960-faac-4b77-9608-a9da61c6678e/ConfigProVal.cfg)