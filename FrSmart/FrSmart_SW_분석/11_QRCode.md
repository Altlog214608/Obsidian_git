

***

## 1. FrSmart.py에서 QR코드 동작 원리와 방법

- FrSmart.py 파일에서 QR코드는 주로 `dsQRcode` 모듈을 import 해서 사용합니다.
- 이 QR코드 생성 관련 코드는 `setUiFindExpDlg` 함수 안에서 등장합니다.  
  - 함수에서 `dsQRcode.generate_qr_code`를 호출해 QR코드 이미지를 생성합니다.
  - 생성된 QR코드 이미지를 `QPixmap`으로 받아서 `QLabel` 위젯에 띄웁니다.
  - QR코드를 우측 상단에 위치시키고, UI에 보여줍니다.

- 즉, FrSmart.py는 QR코드 생성은 별도의 모듈(`dsQRcode.py`)에 위임하고, UI에 보여주기만 담당합니다.

***

## 2. dsQRcode.py 파일 코드 분석

```python
import qrcode
import random
from urllib.parse import urlencode

base_url = "https://hashswim.github.io/perfume_mobile_page/"
```
- qrcode: QR코드 생성 라이브러리
- random: QR코드 URL에 넣을 인덱스를 무작위로 생성하기 위해 사용
- urlencode: URL 쿼리 스트링 생성용
- base_url: QR코드에 포함할 기본 URL

***

### generate_qr_code 함수

```python
def generate_qr_code(idx=None, save_path=None):
    """
    QR 코드를 생성하고 이미지로 저장하는 함수
    Args:
        idx: QR 코드에 포함할 인덱스 (None이면 랜덤 생성)
        save_path: 저장할 파일 경로 (None이면 기본 경로 사용)
    Returns:
        str: 생성된 QR 코드 이미지 파일 경로
    """

    # 0~7 사이의 인덱스를 자동으로 할당
    if idx is None:
        idx = random.randint(0, 7)
    print(f"자동 할당된 idx: {idx}")

    params = {"idx": idx}

    full_url = f"{base_url}?{urlencode(params)}"
    print("생성할 URL:", full_url)

    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_L,
        box_size=10,
        border=4,
    )

    qr.add_data(full_url)
    qr.make(fit=True)

    img = qr.make_image(fill_color="black", back_color="white")

    if save_path is None:
        save_path = f"qrcode_idx_{idx}.png"

    img.save(save_path)
    print(f"QR 코드가 '{save_path}'로 저장되었습니다.")

    return save_path
```

***

#### 세부 설명

- `idx=None`: 인덱스를 안 주면 0~7 사이 임의 값을 사용
- `params = {"idx": idx}`: URL 쿼리스트링에 idx 값을 넣음 (예: "?idx=3")
- `full_url`은 base_url에 idx 쿼리를 붙인 최종 URL 생성
- `qrcode.QRCode()`는 QR코드 객체 생성, 파라미터 설정
  - version=1: 21x21 크기의 QR코드 (작은 크기)
  - error_correction: 오류정정 레벨 선택(L은 가장 기본)
  - box_size: QR코드 한 칸 크기(픽셀)
  - border: QR코드 테두리 칸 수
- `qr.add_data(full_url)`: QR코드에 URL 데이터를 넣음
- `qr.make(fit=True)`: QR코드 그리기 준비
- `qr.make_image()`: 이미지로 QR코드 변환 (흑백)
- 저장 경로가 없으면 기본으로 `"qrcode_idx_{idx}.png"` 로 저장
- 이미지 저장 후 저장 경로 반환

***

### 메인 실행부 (파일 바로 실행 시)

```python
if __name__ == "__main__":
    idx = random.randint(0, 7)
    print(f"자동 할당된 idx: {idx}")

    params = {"idx": idx}
    full_url = f"{base_url}?{urlencode(params)}"
    print("생성할 URL:", full_url)

    qr = qrcode.QRCode(
        version=1,
        error_correction=qrcode.constants.ERROR_CORRECT_L,
        box_size=10,
        border=4,
    )

    qr.add_data(full_url)
    qr.make(fit=True)

    img = qr.make_image(fill_color="black", back_color="white")
    img.save(f"qrcode_idx_{idx}.png")
    print(f"QR 코드가 'qrcode_idx_{idx}.png'로 저장되었습니다.")
```

- 별도 인자 없이 실행하면 0~7 사이 랜덤 idx를 생성
- 위와 같은 방식으로 QR코드 생성 및 저장
- 동일 기능을 함수로 분리하여 재사용 용이

***

## 3. 동작 흐름 정리

- FrSmart.py 내에서 QR코드를 필요로 하면 `generate_qr_code(idx)` 호출하여 URL에 idx를 넣은 QR코드를 생성
- 생성된 이미지 경로를 받아 UI 내 QLabel에 이미지를 보여줌
- QR코드에는 `https://hashswim.github.io/perfume_mobile_page/?idx=n` 형식의 URL이 들어감
- 사용자가 QR코드를 스마트폰으로 스캔하면, 해당 URL(인덱스에 따른 페이지 또는 정보)에 접속할 수 있음

***

필요 시 Pillow 라이브러리 설치가 되어 있어야 `qrcode` 모듈 내 이미지 저장이 정상 동작합니다.

***

요약하자면, dsQRcode.py는 인덱스 값을 URL 쿼리로 포함한 QR코드를 만들고 이미지 파일로 저장하는 역할을 하며, FrSmart.py는 이 모듈을 호출해 생성한 QR코드를 UI에 보여주는 역할을 합니다.  

