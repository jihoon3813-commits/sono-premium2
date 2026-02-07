# 구글 시트 연동 가이드

이 문서는 사용자의 랜딩 페이지(HTML/JS 정적 웹사이트)에 구글 스프레드시트의 가전 제품 데이터를 자동으로 불러와서 표시하는 방법을 설명합니다.

이 방식을 사용하면 **구글 시트만 수정해도 홈페이지의 제품 목록이 자동으로 업데이트**되므로 유지보수가 매우 편리합니다.

---

## 1. 구글 스프레드시트 준비 (필수)

제공해주신 시트(`https://docs.google.com/spreadsheets/d/1T2WuRo420n5QThG3T6scs8GKY7zSI9kpr0aWXhtn0Wg/edit`)를 **웹에 게시**해야 외부에서 데이터를 읽어올 수 있습니다.

### 1-1. 시트 데이터 구조 정리
시트의 **첫 번째 줄(Row 1)**을 헤더(항목 이름)로 사용해야 합니다. 아래의 **영어 변수명**을 정확히 입력해주세요. (한글 헤더도 가능하지만, 코딩 편의상 영문을 권장합니다.)

| A열 (Brand) | B열 (Model) | C열 (Name) | D열 (Tag) | E열 (Image) |
| --- | --- | --- | --- | --- |
| brand | model | name | tag | image |
| 삼성 | V15 | 비스포크 냉장고 | 1구좌 전용 | (이미지 주소) |

*   **brand**: 브랜드명 (예: 삼성, LG)
*   **model**: 모델명 (예: V15)
*   **name**: 제품명 (예: 비스포크 냉장고)
*   **tag**: 태그 (예: 1구좌, 2구좌)
*   **image**: 제품 이미지 URL (직접 업로드한 이미지 링크)

### 1-2. 웹에 게시 (Publish to Web)
1.  구글 시트 메뉴에서 **파일 (File) > 공유 (Share) > 웹에 게시 (Publish to web)**를 클릭합니다.
2.  나타나는 팝업에서 **"전체 문서"** 대신 제품 리스트가 있는 **"시트 이름"**을 선택합니다.
3.  형식을 **"웹페이지"**가 아닌 **"쉼표로 구분된 값 (.csv)"**으로 선택합니다. **(중요!)**
4.  **게시 (Publish)** 버튼을 누릅니다.
5.  생성된 **링크(URL)**를 복사합니다. 이 링크가 홈페이지 연동에 필요합니다.

---

## 2. 홈페이지 코드 수정

### 2-1. CSV 파싱 라이브러리 추가
`index.html` 파일의 `<head>` 태그 안이나 `<body>` 닫는 태그 직전에 아래 스크립트를 추가하여 CSV 변환 라이브러리인 PapaParse를 불러옵니다.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
```

### 2-2. CSV 데이터 불러오기 및 렌더링 스크립트 작성
`index.html`의 하단 `<script>` 영역에 아래 코드를 추가합니다. `GOOGLE_SHEET_CSV_URL` 예시 부분에 **1-2단계에서 복사한 CSV 링크**를 넣어야 합니다.

```javascript
// 1. 구글 시트 CSV 링크 (반드시 '웹에 게시 > CSV' 링크여야 함)
// 예: https://docs.google.com/spreadsheets/d/e/2PACX-..../pub?output=csv
const SHEET_URL = "여기에_복사한_CSV_링크를_넣으세요";

// 2. 제품 그리드 컨테이너 선택
const productContainer = document.querySelector('.product-container'); // 또는 해당 그리드 ID

// 3. 데이터 로드 및 렌더링 함수
function loadProducts() {
    if(!SHEET_URL.includes("output=csv")) {
        console.error("CSV 링크가 아닙니다. 구글 시트 '웹에 게시 > CSV' 링크를 확인해주세요.");
        return;
    }

    Papa.parse(SHEET_URL, {
        download: true,
        header: true, // 첫 줄을 헤더로 인식
        complete: function(results) {
            const products = results.data;
            renderProducts(products);
        },
        error: function(err) {
            console.error("데이터 로드 실패:", err);
        }
    });
}

// 4. HTML 생성 함수
function renderProducts(products) {
    productContainer.innerHTML = ""; // 기존 정적 아이템 제거

    products.forEach(item => {
        // 빈 행 제외
        if (!item.name) return;

        // 이미지 없을 시 기본 이미지 처리
        const imgUrl = item.image ? item.image : 'placeholder.jpg';

        const html = `
            <div class="product-item">
                <div class="product-img-box" style="background-image: url('${imgUrl}'); background-size: cover; background-position: center;">
                    <div class="product-tag">${item.tag}</div>
                </div>
                <div class="product-info">
                    <span class="prod-model">${item.brand} / ${item.model}</span>
                    <h4 class="prod-title">${item.name}</h4>
                    <div class="prod-benefit"><i class="fas fa-bolt"></i> PREMIUM POINT +</div>
                </div>
            </div>
        `;
        productContainer.insertAdjacentHTML('beforeend', html);
    });
}

// 실행
document.addEventListener('DOMContentLoaded', loadProducts);
```

---

## 3. 요약 확인
1. 구글 시트의 첫 줄에 `brand`, `model`, `name`, `tag`, `image` 헤더가 있는지 확인하십시오.
2. `파일 > 공유 > 웹에 게시`에서 **CSV 형식**으로 링크를 생성하십시오.
3. 위 코드를 `index.html`에 적용하고 CSV 링크 변수를 교체하면, 시트를 수정할 때마다 홈페이지 제품 목록이 자동으로 변경됩니다.

궁금한 점이 있거나, **제가 직접 코드를 적용해드리길 원하시면** `웹에 게시` 후 생성된 **CSV 링크**를 저에게 알려주세요!
