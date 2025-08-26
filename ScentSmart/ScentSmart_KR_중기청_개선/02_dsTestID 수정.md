완전 맞아요. `id_test_data`엔 처음에 **12문항**이 들어 있고, `rebuild_to_3choice_and_sample_8()`를 부르면 그 12문항을 가공해서 **3지선다 + 셔플 + 8문항**으로 만든 뒤

```python
id_test_data = rebuilt[:NUM_QUESTIONS]
```

여기서 **최종 8문항으로 갈아끼웁니다.** 이제 코드 전체를 **블록별/줄별**로 차근차근 설명할게요.

---

# 파일 상단: 임포트 & 전역 상태

```python
import dsText
import random
import copy
```

- `dsText`: UI에 쓰는 문자열 리소스(라벨/메시지 번역 등). 이 파일 안에선 직접 쓰지 않아도 모듈 인터페이스상 같이 임포트해둔 것으로 보입니다.
    
- `random`: 보기(오답)와 문항 셔플/샘플링에 사용.
    
- `copy`: 원본 문항을 **deepcopy**로 보존하기 위해 사용.
    

```python
id_time_count = 0
id_test_index = 0 # 인지검사 현재 번호
id_temp_response = ""
```

- **검사 진행 상태** 변수들:
    
    - `id_time_count`: 인지검사 경과 시간(초)을 카운트할 때 사용.
        
    - `id_test_index`: 현재 몇 번째 문항을 보여주는 중인지(0-based).
        
    - `id_temp_response`: 사용자가 현재 선택하거나 입력 중인 응답 임시 저장.
        

```python
id_results_title = ['문항', '정답', '선택', '정답여부', '응답(주관식)', '정답여부(주관식)']
id_results = []
I_score = 0
```

- 결과 테이블의 **헤더**와 **실제 결과 저장 리스트**, **점수**.
    
    - `id_results`는 매 문항 끝날 때 `['문항번호','정답','선택','O/X','주관식','O/X']` 같은 한 줄씩 append.
        
    - `I_score`는 정답 개수 누적.
        

---

# 문항 데이터

```python
id_test_data = [
  {...},  # scent_no 1
  ...
  {...}   # scent_no 12
]
```

- **최초 로딩 시점**의 문항 풀.
    
- 각 문항에 `choice1~choice10`까지 들어 있습니다(= 보기 후보 10개).
    
- `answer`는 정답, `scent_no`는 장치 채널(또는 향 ID), `outlet_no`는 분사 포트.
    

```python
id_test_data2 = [
  {...},  # scent_no 1
  ...
  {...}   # scent_no 12
]
```

- **4지선다 원본** 버전. 현재 로직에선 쓰지 않지만, 필요 시 참고/비교용으로 남겨둔 데이터.
    

---

# 튜닝용 상수

```python
NUM_SELECTED_OPTIONS = 3
NUM_QUESTIONS = 8
```

- `NUM_SELECTED_OPTIONS`: 한 문항에 **화면에 보여줄 보기 개수**(정답1 + 오답2 = 3).
    
- `NUM_QUESTIONS`: **최종 출제 문항 수**(셔플 후 8문항만 사용).
    

> 나중에 4지선다로 바꾸고 싶으면 `NUM_SELECTED_OPTIONS = 4`만 바꾸면 됩니다. 10문항으로 늘리고 싶으면 `NUM_QUESTIONS = 10`.

---

# 원본 백업 버퍼

```python
try:
    id_test_data_full
except NameError:
    id_test_data_full = None
```

- **모듈 재로딩/다중호출 시에도** “원본 12문항(지금은 choice10까지 확장된 버전)”을 한 번만 백업하기 위한 변수.
    
- 첫 호출 때만 원본을 저장하고, 이후부터는 항상 **같은 원본을 기반으로** 3지선다/셔플을 다시 만듭니다(= **시험 시작마다 랜덤 재구성** 가능).
    

---

# 보기 수집 유틸

```python
def _collect_choices(q, max_n=20):
    """문항 딕셔너리에서 choice1..choiceN을 모두 수집 (중복 제거)."""
    seen = set()
    choices = []
    for i in range(1, max_n+1):
        k = f"choice{i}"
        if k in q and q[k] and q[k] not in seen:
            seen.add(q[k])
            choices.append(q[k])
    return choices
```

- **한 문항 `q`에서** `choice1`부터 `choice20`까지(최대치 넉넉히) **존재하는 보기들을 전부** 긁어옵니다.
    
- `seen`으로 **중복 제거**합니다.
    
- 결과: 그 문항의 “보기 후보 풀(list)”.
    

---

# 핵심: 12문항 → (각 문항) 3지선다로 축소 → 셔플 → 8문항 선택

```python
def rebuild_to_3choice_and_sample_8():
    global id_test_data, id_test_data_full
```

- 전역 `id_test_data`(가공 대상)와 `id_test_data_full`(원본 보존)을 **이 함수에서 수정**하겠다는 선언.
    

```python
    if id_test_data_full is None:
        id_test_data_full = copy.deepcopy(id_test_data)
```

- **최초 1회만** 현재의 `id_test_data`(= 12문항×choice10까지 확장본)를 깊은 복사로 **영구 원본**(`id_test_data_full`)에 저장.
    
- 이후 재호출 시 원본을 덮지 않음 → **항상 같은 12문항을 기준**으로 매번 랜덤 샘플링만 새로 함.
    

```python
    rebuilt = []
    global_answers = [q['answer'] for q in id_test_data_full if 'answer' in q]
```

- `rebuilt`: 새로 만들어질 **3지선다 문항 리스트**(잠시 후 여기에 12개가 들어가고, 마지막에 8개만 남김).
    
- `global_answers`: 모든 문항의 정답 목록(오답 후보가 부족할 때 **보충**용).
    

```python
    for q in id_test_data_full:
        ans = q['answer']
        pool = _collect_choices(q, max_n=20)
```

- **12문항을 순회**.
    
- `ans`: 현재 문항의 정답.
    
- `pool`: 현재 문항의 **보기 후보**(choice1~choice10, 중복 제거).
    

```python
        if ans not in pool:
            pool = [ans] + pool
```

- 혹시라도(데이터 오류 등) 정답이 보기에 없다면 **강제로 포함**해 일관성 보장.
    

```python
        wrongs = [c for c in pool if c != ans]
        need = (NUM_SELECTED_OPTIONS - 1)
```

- `wrongs`: 정답을 뺀 **오답 후보 리스트**.
    
- `need`: 뽑아야 할 오답 수(3지선다이므로 2개).
    

```python
        if len(wrongs) < need:
            supl = [x for x in global_answers if x != ans and x not in wrongs]
            while len(wrongs) < need and supl:
                take = random.choice(supl)
                supl.remove(take)
                wrongs.append(take)
```

- **오답 후보가 모자라면**(이론상 드뭅니다) 전체 정답 풀에서(자기 정답 제외) **중복 없는 오답**을 보충.
    
- `while`로 하나씩 추가하되, 보충 풀이 떨어지면 멈춤(실제 데이터에선 충분히 많으니 문제 없음).
    

```python
        final_opts = [ans] + random.sample(wrongs, need)
        random.shuffle(final_opts)
```

- 최종 보기 3개 구성: **정답 1개 + 오답 2개 랜덤 샘플**.
    
- `random.shuffle`로 **보기 순서 랜덤화**.
    

```python
        new_q = {
            'scent_no': q.get('scent_no', None),
            'outlet_no': q.get('outlet_no', 1),
            'answer': ans,
            'choice1': final_opts[0],
            'choice2': final_opts[1],
            'choice3': final_opts[2],
        }
        rebuilt.append(new_q)
```

- **3지선다 버전 문항**을 새로 만들고 `rebuilt`에 추가.
    
    - 포인트: 이제 각 문항은 **choice1~3만** 존재합니다(UI도 1~3만 바인딩하면 됨).
        

```python
    random.shuffle(rebuilt)
    id_test_data = rebuilt[:NUM_QUESTIONS]
```

- 12개 문항(이미 3지선다로 축소된 상태)의 **문항 순서 셔플**.
    
- **앞에서 8개만** 잘라서 `id_test_data`에 **갈아끼움**(**여기가 핵심!**).
    
    - 이 순간부터, 바깥에서 `dsTestID.id_test_data`를 읽으면 **오직 8문항×3지선다**만 보입니다.
        
    - 원본 12문항은 `id_test_data_full`에 그대로 살아 있습니다.
        

---

# 실제 동작 흐름(요지)

1. 앱에서 인지검사를 시작할 때, **가장 먼저**
    
    ```python
    dsTestID.rebuild_to_3choice_and_sample_8()
    ```
    
    를 호출합니다.
    
2. 그러면 `id_test_data_full`(12문항×10보기)은 **유지**,  
    `id_test_data`는 **3지선다로 줄고(정답 포함 보장) + 셔플 + 8문항만** 남습니다.
    
3. UI 세팅 함수는 **항상 `choice1~3`만** 라벨/버튼에 바인딩하고, **4번 위젯은 숨김**으로 고정합니다.
    
4. 저장 시 총문항은
    
    ```python
    total_q = len(dsTestID.id_test_data)   # 8
    worksheet.write('G3', f'={total_q}-G2')
    ```
    
    처럼 **동적 계산**으로 처리하면, 나중에 10문항/12문항으로 바꿔도 자동 적용됩니다.
    

---

# 팁 & 주의점

- **재현성**: 테스트마다 보기 조합/문항 순서가 달라지지 않게 하려면, 검사 시작 전에 `random.seed(고정값)`을 한 번 주면 됩니다.
    
- **데이터 안전성**: 현재 구조에선 오답 후보가 매우 많아서 `random.sample(wrongs, need)`에서 에러 날 일이 없습니다. 혹시 커스텀 데이터로 바꾸면서 오타/중복으로 풀이 줄어드는 경우엔 보충 로직이 메꿉니다.
    
- **id_test_data2**: 참고용 원본입니다. 실제 로직은 `id_test_data_full`과 `id_test_data`만 쓰니, 실수로 `id_test_data2`를 UI 쪽에서 참조하지 않도록 주의하세요.
    

---

필요하면, **ScentSmart.py** 쪽의

- `startIdentification()`에 한 줄 넣는 위치,
    
- `setResponseUiTestIdentification()`에서 1~3만 바인딩하도록 바꾼 예시,
    
- `unselectResponseIdentification()`의 체크/하이라이트 해제
    

를 지금 코드 기준으로 딱 맞춰 델타 패치 형식으로 써드릴게요.