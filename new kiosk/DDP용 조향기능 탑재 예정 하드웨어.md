

---

# 📌 1. 문서 개요

- **목적**: 조향장치 프로젝트 하드웨어 제원 정의
    
- **MCU**: STM32F446RET6 (Cortex-M4F, 180MHz)
    
- **기능**
    
    - 2축 스테퍼 제어 (X/Y)
        
    - RS485 통신
        
    - 오리진 센서 입력
        
    - 36채널 펌프 분사 제어
        

---

# ⚙ 2. 시스템 동작 구조

## 🧭 전체 동작 흐름

1. 공병을 T-rail에 적재
    
2. Y축 후방 이동 (원점 방향)
    
3. 2mm 후진 (오리진 스위치 이탈)
    
4. X/Y축 이동 → 노즐 1~36번 순차 위치
    
5. 각 노즐에서 정량 분사 (CMD_MAKE_PERFUME 명령 기반)
    

👉 즉, **공병을 이동시키면서 36개 노즐을 순차적으로 분사하는 구조**

---

# 🧩 3. 주요 하드웨어 구성

|구성요소|설명|
|---|---|
|T-rail|공병 이송용 레일|
|스테퍼모터|28H30H0604A2 (1.8°, 0.8A/상), 리드스크류 직결|
|전원|MEAN WELL 12V SMPS|
|노즐 매니폴드|1~36채널, 튜브로 유체 공급|
|릴레이|SN74HC574 x5 (36채널 사용)|
|드라이버|L298 모터 드라이버|

---

# 🧠 4. MCU 및 메모리

### MCU

- STM32F446RET6
    
- LQFP64
    
- Cortex-M4F + FPU
    
- 최대 180MHz
    

### 메모리

- Flash: 512KB
    
- RAM: 128KB
    
- 기본 Heap: 512B
    
- 기본 Stack: 1KB
    

---

# 🔌 5. 핀 및 GPIO 구성 요약

### ✔ 스테퍼 모터

- X축: PA4~PA7
    
- Y축: PB4~PB7
    

### ✔ 오리진 센서

- X: PA11 (EXTI)
    
- Y: PB1 (EXTI)
    

### ✔ RS485

- TX/RX: PA9/PA10
    
- 방향제어: PA8
    

### ✔ 디버그

- SWDIO: PA13
    
- SWCLK: PA14
    

---

# 🛠 6. 주변장치 구성

## UART

- USART1 → RS485
    
- USART2 → USB 시리얼 (호스트 통신)
    

## DMA

- USART1/2 RX Circular Mode
    

## 타이머

- TIM5 → 펌프 PWM (PA0)
    
- TIM6 → 베이스 타이머
    

---

# 🔄 7. 모터 및 펌프 제어 구조

## 🌀 스테퍼 제어

- L298 드라이버 사용
    
- Full/Half Step 지원
    
- 50 pulse = 1mm
    
- 절대 펄스 이동값 사용 (예: X=3000)
    

## 💧 펌프 제어

- TIM5 PWM으로 속도 제어
    
- PC8/PC9 방향 제어
    
- ml → 시간 환산 후 동작
    
- Relay574로 채널 선택
    

흐름:

```
Pump_QueueMl → Pump_StartQueued → Pump_Tick
```

---

# 📡 8. 통신 프로토콜

## 프레임 구조 (AA 33 프로토콜)

```
AA 33 | ID | CMD | LEN | DATA | CRC16 | FC
```

- CRC16 Modbus
    
- Little Endian
    
- 두 가지 형식 (LEN 있음 / 없음)
    

---

## 주요 명령

|명령|코드|설명|
|---|---|---|
|MAKE_PERFUME|0x80|향수 제조 (실제 구현됨)|
|MOVE_*|0x10~0x12|미구현|
|PUMP_START/STOP|0x20/21|미구현|

현재 **실질 구현은 MAKE_PERFUME 중심**

---

## 응답 코드

|코드|의미|
|---|---|
|ACK (0xC0)|수신 확인|
|DONE (0xC1)|완료|
|ERR (0xC2)|오류|

에러 코드:  
BAD_FRAME, BAD_CRC, UNSUP, BUSY 등

---

# 🖥 9. 개발 환경

- STM32CubeIDE 6.15.0
    
- ARM GCC 14.2.1
    
- OpenOCD 0.12
    
- HAL 기반
    
- 링커: STM32F446RETX_FLASH.ld
    

---

# 🎯 전체 구조 한 줄 요약

> STM32F446 기반 2축 스테퍼 + 36채널 펌프 제어 향수 자동 분사 장치이며, AA33 프로토콜 기반 UART/RS485 통신으로 제어되는 구조

---
