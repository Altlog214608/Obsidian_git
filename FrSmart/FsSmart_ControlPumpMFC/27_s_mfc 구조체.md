`s_mfc` 구조체는 MFC 장치의 동작 파라미터(채널별 duty, 상태, 보정계수 등)를 보관하는 전역 메모리 공간이고, 실제 값이 들어가는 시점은 **MFC 초기화 및 설정 로직**이 실행될 때입니다.

---

## 1. 값이 채워지는 시점

`ControlPumpMFC.c` 내부 흐름을 보면:

1. **InitProcess_MFC()**
    
    - MFC 초기화 함수.
        
    - 내부에서 `InitConfig_MFC()` → `ReadConfigProVal_MFC()` 호출.
        
2. **ReadConfigProVal_MFC()**
    
    - `ConfigProVal.cfg` 파일을 읽어서  
        `s_mfc.fscale`, `s_mfc.offset`, `s_mfc.coeff[0..5]`, `s_mfc.dutyzero[]` 등 구조체 멤버에 값을 채움.
        
    - 즉, 보정계수·기본 duty 값은 여기서 세팅됨.
        
3. **SetDutyBoard_MFC() / ReadDutyBoard_MFC()**
    
    - 채널별 현재 duty 값(`s_mfc.duty[]`)을 읽거나 새로 쓸 때 갱신.
        
4. **DecodeData_MFC()**
    
    - 장비로부터 읽은 MODBUS 응답을 해석해 `s_mfc.status[]`나 `s_mfc.duty[]` 갱신.
        

즉, `s_mfc` 값은 **설정 파일 읽기**와 **장비 통신 응답 처리** 단계에서 채워집니다.

---

## 2. `unsigned short *duty` / `*status`의 의미

- `*`는 **포인터(pointer)**를 의미합니다.
    
- 예를 들어:
    
    ```c
    unsigned short *duty;
    ```
    
    는 “`unsigned short` 타입의 데이터를 가리키는 주소”를 저장하는 변수입니다.
    
- 여기서는 채널이 여러 개이기 때문에,  
    `malloc` 같은 동적 메모리 할당으로 채널 개수만큼 배열을 만들고,  
    그 배열의 첫 번째 요소 주소를 `s_mfc.duty`에 저장합니다.
    
- 사용 예:
    
    ```c
    s_mfc.duty[0]   // 첫 번째 채널 duty 값
    s_mfc.duty[5]   // 여섯 번째 채널 duty 값
    ```
    
    이런 식으로 배열처럼 접근 가능.
    

---

## 3. 안정장치(보호 로직)

MCU 쪽에서는 보통 아래 상황에서 값 제한 로직이 있습니다:

- **fscale**, **offset** 값을 파일에서 읽어 초기화  
    → 비정상 값이면 기본값으로 대체.
    
- duty 계산 시:
    
    - 계산 결과가 음수면 0으로,
        
    - 최대 허용 duty(예: 100% → *100)보다 크면 최대값으로 제한.
        
- 이런 제한 로직이 `SetDutyBoard_MFC()`나 duty 계산 부분에서 들어갑니다.
    

---

원하시면 제가 이 구조를 기반으로

- `s_mfc` 초기화·갱신 과정 함수 호출 순서도
    
- 그리고 수정된 **zduty / fscale / coeff 기반 엑셀 계산표**  
    두 가지를 함께 만들어서 드릴 수 있습니다.
    

그렇게 정리해 드릴까요?