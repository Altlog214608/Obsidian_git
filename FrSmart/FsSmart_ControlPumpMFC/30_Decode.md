

---

# 1) `DecodeData_MFC(unsigned short add, unsigned short rdreg)`

```c
int DecodeData_MFC(unsigned short add, unsigned short rdreg)
{
    int i,index,rindex,nodata;
    unsigned short *rdata;
    unsigned char datastr[256];
    double ddata[2];
```

- `add`: 이번에 읽은(혹은 쓰고나서 확인하려는) **시작 레지스터 주소**. 스위치문에서 어떤 의미의 데이터인지 분기할 때 씀.
    
- `rdreg`: **읽은 레지스터 개수**(단위: 16‑bit word). 메모리 할당 크기 등에 사용하려는 의도.
    
- `rdata`: 응답 바이트들을 **16비트 값 배열**로 재구성해서 담을 임시 버퍼.
    
- `datastr`: 디버그용 문자열 버퍼(값 출력).
    
- `ddata`: 실수 스케일링 등의 일시 저장용.
    

```c
    rdata = (unsigned short *) malloc(rdreg * sizeof(short));
```

- 응답을 옮겨 담을 공간을 **rdreg 개수만큼** 확보.  
    (권장: `sizeof(unsigned short)`가 더 명확함. 지금은 대부분 2바이트라 동작상 문제는 없지만 이식성 면에서 주의.)
    

```c
    index = 3;
    rindex = 0;
```

- `index=3` → **MODBUS 응답 형식**에서  
    `[0]=slave id, [1]=func, [2]=byte count, [3..] 데이터 ...` 이므로 **데이터 시작 지점은 3**.
    
- `rindex`는 `rdata[]` 채울 때 쓰는 인덱스.
    

```c
    nodata = (int) (s_serial.read[2]/2);
```

- `s_serial.read[2]`는 **바이트 수(byte count)**. 이걸 **word 개수**로 바꾸려고 `/2`.
    
    - 예: 바이트 수 12 → word 6개.
        

```c
    sprintf(datastr,"value[0x%04X] = ",add);
```

- 출력용 문자열 시작: "value[0x108E] = ..." 같은 형식.
    

```c
    for(i=0;i<nodata;i++) {
        rdata[rindex] = (short) s_serial.read[index];  index++;
        rdata[rindex] <<= 8;
        rdata[rindex] |= (short) s_serial.read[index]; index++;

        sprintf(datastr,"%s %d",datastr,rdata[rindex]);
        rindex++;
    }
```

- **바이트 → 16비트 값**으로 재조립(빅엔디안):
    
    1. 상위바이트를 넣고,
        
    2. 왼쪽으로 8비트 쉬프트,
        
    3. 하위바이트를 OR 해서 **하나의 `unsigned short` 값**으로 만든다.  
        → MODBUS 응답 데이터가 **High, Low** 순서로 오기 때문에 이렇게 조립.
        
- 각 워드를 `datastr`에 이어붙여 디버그 문자열로 만든다.
    

> ⚠️ **주의 포인트 ①**: `nodata`(실제 들어온 word 개수)가 `rdreg`(할당 크기)보다 큰 경우 **오버런 위험**.  
> 지금 코드는 `nodata`만큼 루프를 돌고 `rdata`는 `rdreg` 크기로만 할당했기 때문에,  
> **안전하게 하려면** `for (i=0; i<min(nodata, rdreg); i++)`가 맞다.

```c
    if (add != MFC_CGP) SetCtrlVal(control_handle,CPL_RES,datastr);    // print
```

- 읽은 값들을 UI(예: 텍스트 박스)에 출력.  
    (압력 `MFC_CGP`일 때는 아래 케이스에서 따로 그래프 그리므로 여기선 스킵)
    

```c
    switch (add) {
        case MFC_NSB :  //# of sol board 
                        s_mfc.tnsbd = rdata[0];  
                        s_mfc.tnmfl = s_mfc.tnsbd * 6;
                        
                        s_mfc.duty     = (unsigned short *)malloc(s_mfc.tnsbd * 6 * sizeof(short));  //2-byte reserved
                        s_mfc.status   = (unsigned short *)malloc(s_mfc.tnsbd * 6 * sizeof(short));  //line status
                        s_mfc.lhindex  =            (int *)malloc(s_mfc.tnsbd * 6 * sizeof(int));    //line index
                        s_mfc.dutyzero =         (double *)malloc(s_mfc.tnsbd * 6 * sizeof(double)); //duty @ zero flowrate
                        if (s_serial.debug) printf("nbr of sol. board = %d\n",s_mfc.tnsbd);
                        break;    
```

- **MFC_NSB**(Number of Solenoid Boards) 응답:
    
    - 보드 수(`tnsbd`)를 rdata[0]에서 읽는다.
        
    - **한 보드에 6채널**이므로 총 라인수 `tnmfl = tnsbd*6`.
        
    - 그 길이에 맞춰 `duty/status/lhindex/dutyzero` 배열을 **동적 할당**.
        
    - 이후부터는 이 배열들을 **배열처럼** 접근(`s_mfc.duty[k]`) 가능.
        
    - (권장: `sizeof(unsigned short)` 등 타입 통일, 재초기화 시 기존 메모리 `free` 필요)
        

```c
        case MFC_SPS :  //board port status
                        for(i=0;i<s_mfc.tnsbd * 6;i++) s_mfc.status[i] = rdata[i];
                        break;
```

- **MFC_SPS**(Status) 응답: **보드수×6개**의 상태 워드를 `status[]`에 채운다.
    

```c
        case MFC_SPA :  //port duty
                        for(i=0;i<s_mfc.tnsbd * 8;i++) s_mfc.duty[i] = rdata[i];
                        break; 
```

- **MFC_SPA**(Absolute port duty) 응답: **현재 채널들의 듀티 값**을 `duty[]`에 채운다.
    
- **주의 포인트 ②**: 여기 **`*8`**은 눈에 띄는 부분이야. 위에서 모든 배열 할당을 **`tnsbd*6`** 크기로 했는데,  
    여기서 **`tnsbd*8`까지** 쓴다면 **버퍼 오버런**이 발생할 수 있어.
    
    - 실제 장치가 8채널 보드라면 **할당 크기도 8로** 맞춰야 하고,
        
    - 6채널 장치라면 이 루프는 **반드시 6으로** 고쳐야 함.  
        (지금 테스트에서 문제가 없었던 건 우연이거나 `rdreg`가 6에 맞게 들어와서 루프가 덜 돌았기 때문일 수 있어.)
        

```c
        case MFC_CGP :  //carrier gas pressure
                        ddata[0] = (double) rdata[0] * 0.1;
                        SetCtrlVal(acq_handle,DPNL_PRES,ddata[0]);
                        PlotStripChart (acq_handle,DPNL_SCHART,ddata,1,0,0,VAL_DOUBLE);
                        break;
        default : ;
    }
```

- **MFC_CGP**: 압력값 스케일을 `×0.1` 해서 UI에 표시하고 스트립차트에 찍는다.
    

```c
    free(rdata);
    return 0;
}
```

- 임시 버퍼 해제. (좋음)
    

---

# 2) `DecodePacket(void)`

```c
void DecodePacket(void)
{
    int errindex;
    
    SetCtrlVal(control_handle,CPL_RES,s_serial.read);    
```

- 수신 버퍼 `s_serial.read` 전체를 UI 컨트롤에 그대로 출력.  
    (보통은 헥사덤프 문자열로 변환해 보여주지만, 여기서는 컨트롤 타입에 따라 바로 표시하는 듯)
    

```c
    if (s_serial.read[1] & 0x80) {
        errindex = s_serial.read[1] - 0x80;
        Error_MFC(errindex);
    }
```

- **MODBUS 예외 응답** 체크:
    
    - 정상 응답이면 **기능코드**(예: 0x03, 0x06, 0x10)가 오지만,
        
    - 예외 응답이면 **기능코드 + 0x80**(예: 0x83, 0x90)이 온다.
        
    - 이 경우 `(code - 0x80)`을 에러 인덱스로 넘겨 `Error_MFC()` 호출.
        

```c
    return;
}
```

- 끝.
    
- **중요**: `DecodePacket()` 자체가 `DecodeData_MFC()`를 직접 호출하진 않아.  
    보통은 “누가 어떤 주소(예: 0x108E)를 얼마나 읽으라고 요청했는지”를 알고 있는 **상위 로직**에서,  
    **그 주소(`add`)와 `rdreg`을 전달해 `DecodeData_MFC(add, rdreg)`** 를 호출하는 식으로 짝을 맞춰.  
    (그래야 `switch(add)` 분기가 맞게 동작하거든.)
    

---

## MODBUS 응답 포맷 복습 (왜 index=3부터인가?)

- RTU Read 응답의 전형:
    
    ```
    [0]=SlaveID
    [1]=Function
    [2]=ByteCount (N)
    [3..(3+N-1)]=Data bytes (High,Low, High,Low, ...)
    [3+N]=CRC Lo
    [4+N]=CRC Hi
    ```
    
- 그래서 **데이터 시작 오프셋은 3**, 바이트 수는 `s_serial.read[2]`.  
    워드 개수는 그 절반(`nodata = byte_count/2`).
    

---

## 요약: 너가 이해한 흐름이 맞아

- PC(FrSmart/dsComm)에서 **write**로 보낸 MODBUS 프레임이 MCU에 도착하고,
    
- MCU가 응답을 돌려주면 PC 쪽 `s_serial.read[]` 버퍼에 들어옴.
    
- 그걸 `DecodePacket()`이 먼저 검사(예외/표시)하고,
    
- 실제 값 파싱은 **`DecodeData_MFC(add, rdreg)`**에서 `s_mfc.duty/status` 등으로 **해석 및 반영**된다.
    

---

## 추천 보완(버그 가능성/안전성)

1. **MFC_SPA 루프의 `*8`**
    
    - 현 구조(보드당 6채널)라면 `*6`이 맞음. 8채널 보드가 설계에 있다면 **할당도 8로** 동기화 필요.
        
2. **`rdata` 크기 방어**
    
    - `nodata`가 `rdreg`보다 클 때를 대비해 `min(nodata, rdreg)`로 읽기.
        
3. **`sizeof(unsigned short)` 사용**
    
    - 가독성과 이식성 향상.
        
4. **`datastr`에 계속 `sprintf` 덧붙이기**
    
    - 길이 초과 방지 위해 `snprintf` 권장.
        

---

필요하면, 이 해석대로 **수신 버퍼 → rdata 조립 → MFC_NSB/SPS/SPA 분기**까지 한 눈에 보이는 **시퀀스 다이어그램**을 그려줄게.  
또는, `DecodeData_MFC()`를 안전하게 바꾼 **개선 버전 코드**(오버런 방지/8→6 동기화/`snprintf`)도 함께 정리해줄 수 있어.