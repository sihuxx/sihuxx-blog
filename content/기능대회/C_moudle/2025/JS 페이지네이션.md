# 책 대여 유저 조회 (JS 페이지네이션)

#기능대회

## 1. PHP (Backend Data)

DB에서 데이터를 조회해 준비하는 단계. 페이지네이션을 프론트엔드(JS)에서 처리하므로 조건에 맞는 **전체 데이터를 한 번에** 가져옴.

```php
<?php
  require_once './header.php';
  require_once './lib.php';
  checkUser("admin");

  $store_idx = $_GET["idx"];

  // [DB 조회]
  // LIMIT 없이 전체 데이터 조회 (자르기는 JS가 담당)
  // 필요한 정보: 유저 정보, 책 정보, 지점 정보
  $user = db::fetchAll("
    SELECT u.*, ub.*, b.title, s.idx as store_id, u.idx as user_id
    FROM user u
    INNER JOIN user_book ub ON u.idx = ub.user_idx
    INNER JOIN book b ON b.idx = ub.book_idx
    INNER JOIN stores s ON s.idx = ub.store_idx
    WHERE s.idx = $store_idx
  ");
?>
```

## 2. HTML (UI Structure)

화면의 뼈대. 데이터가 들어갈 `tbody`는 비워 두고 버튼에 이벤트를 연결.

```html
<main class="view-box">
  <header>
    <h1>책 대여 유저 조회 (표)</h1>
    <p>표로 책 대여 유저를 조회하세요.</p>
  </header>

  <div class="btn-box">
    <button onclick="filterData('반납됨')" class="white-btn btn">반납됨</button>
    <button onclick="filterData('대여중')" class="white-btn btn">대여중</button>
  </div>

  <table class="nomal-table">
    <thead>
      <tr>
        <th>상태</th>
        <th>닉네임</th>
        <th>아이디</th>
        <th>책 이름</th>
        <th>대여일</th>
        <th>반납 기한</th>
        <th>관리</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <div class="table-control">
    <span class="left white-btn btn" onclick="changePage(-1)">&lt;</span>

    <p class="page-info">1 / 1</p>

    <span class="right white-btn btn" onclick="changePage(1)">&gt;</span>
  </div>
</main>
```

## 3. JS

### 3-1. 변수 선언 및 할당

PHP에서 넘어온 데이터를 JS 변수로 저장하고, 조작할 HTML 요소를 선택하는 단계

- **`userData`**: 전체 원본 데이터 (변하지 않음)
- **`data`**: 화면에 보여줄 현재 데이터 (필터링 시 변경)
- **`pageIndex`**: 현재 페이지 번호 (0부터 시작)

```javascript
// 1. PHP 데이터를 JS로 가져오기
const userData = <?= json_encode($user) ?>;

// 2. HTML 요소 선택 (테이블 본문, 페이지 번호, 버튼들)
const tableBody = document.querySelector("table tbody");
const pageInfo = document.querySelector(".page-info");
const btns = document.querySelectorAll(".btn-box button");

// 3. 페이지네이션 설정
const limit = 5;          // 한 페이지에 5개씩 보기
let pageIndex = 0;        // 현재 페이지 (0 = 1페이지)
let data = [...userData]; // 필터링을 위해 원본 복사
```

### 3-2. 필터링 기능 (대여중 / 반납됨)

상단 버튼을 눌렀을 때 데이터를 거르는 로직. **필터링 후에는 반드시 페이지를 1페이지(0)로 되돌려야 함.**

```javascript
// 버튼 스타일 변경 (클릭한 버튼만 색 적용)
btns.forEach(btn => {
  btn.addEventListener("click", () => {
    btns.forEach(e => e.classList.add("white-btn"));
    btn.classList.remove("white-btn");
  })
})

// 실제 데이터 필터링 로직
function filterData(val) {
  // '대여중'이면 is_rental이 1인 것, 아니면 0인 것만 남김
  if (val == "대여중") {
    data = userData.filter(n => n.is_rental == 1);
  } else {
    data = userData.filter(n => n.is_rental == 0);
  }

  // [중요] 데이터 개수가 바뀌었으므로 첫 페이지로 리셋
  pageIndex = 0;

  // 화면 다시 그리기
  renderUser();
}
```

### 3-3. 화면 그리기 (렌더링)

**핵심 코드.** 현재 페이지(`pageIndex`)에 해당하는 데이터 5개만 `slice`로 잘라 테이블에 넣고, 페이지 번호(`1 / 3`)도 여기서 계산.

```javascript
function renderUser() {
  // 1. 자를 범위 계산
  // 예: 0페이지면 0~5번, 1페이지면 5~10번
  const start = pageIndex * limit;
  const end = (pageIndex + 1) * limit;

  // 2. 데이터 자르기
  const viewData = data.slice(start, end);

  // 3. HTML 생성 후 삽입
  tableBody.innerHTML = viewData.map(n => `
    <tr>
      <td>
        <span class="user-type" style="background-color: ${n.is_rental == '1' ? "#9cdc12" : "#ff7474"};">
          ${n.is_rental == '1' ? "대여중" : "반납됨"}
        </span>
      </td>
      <td>${n.name}</td>
      <td>${n.id}</td>
      <td>${n.title}</td>
      <td>${n.rental_date}</td>
      <td>${n.period}일</td>
      <td>
        <a href="profile.php?idx=${n.user_id}" class="btn white-btn">프로필 보기</a>
      </td>
    </tr>
  `).join(''); // 배열을 문자열로 합치기

  // 4. 페이지 번호 업데이트 (데이터가 없으면 최소 1페이지 표시)
  const totalPage = Math.ceil(data.length / limit) || 1;
  pageInfo.textContent = `${pageIndex + 1} / ${totalPage}`;
}
```

### 3-4. 페이지 이동 (Next / Prev)

`<`, `>` 버튼을 눌렀을 때 페이지 번호를 변경. **없는 페이지로 넘어가지 않도록 막는 예외 처리**가 핵심.

```javascript
function changePage(num) {
  // num은 -1(이전) 또는 1(다음)
  const nextPage = pageIndex + num;

  // 마지막 페이지 번호 (총 페이지 - 1)
  const maxPage = Math.ceil(data.length / limit) - 1;

  // [방어 코드]
  // 1. 0보다 작아지면 안 됨 (첫 페이지에서 이전 클릭 시)
  // 2. 마지막 페이지보다 커지면 안 됨 (마지막 페이지에서 다음 클릭 시)
  if (nextPage < 0 || (nextPage > maxPage && maxPage >= 0)) return;

  // 통과하면 페이지 변경 후 다시 그리기
  pageIndex = nextPage;
  renderUser();
}
```

### 3-5. 실행 및 이벤트 연결

화살표 버튼에 클릭 이벤트를 연결하고, 처음 로드될 때 목록이 비지 않도록 `renderUser()`를 한 번 실행

```javascript
// 화살표 버튼 클릭 이벤트 연결
document.querySelector(".right").onclick = () => changePage(1);
document.querySelector(".left").onclick = () => changePage(-1);

// 최초 1회 실행 (화면 로딩 시)
renderUser();
```

## 4. 로직 상세

### 데이터 자르기 공식 (`slice`)

`pageIndex`가 바뀔 때마다 배열의 어느 범위를 가져올지 결정

| 페이지(`pageIndex`) | 시작(`start`) | 끝(`end`) | 데이터 범위 |
| --- | --- | --- | --- |
| **0** (1쪽) | `0 * 5 = 0` | `1 * 5 = 5` | `arr[0]` ~ `arr[4]` |
| **1** (2쪽) | `1 * 5 = 5` | `2 * 5 = 10` | `arr[5]` ~ `arr[9]` |
| **2** (3쪽) | `2 * 5 = 10` | `3 * 5 = 15` | `arr[10]` ~ `arr[14]` |

### 페이지 이동 보호 (`changePage`)

버튼을 연타해도 에러가 나지 않게 하는 안전장치

1. **왼쪽 한계**: `nextPage < 0`이면 이미 첫 페이지이므로 중단
2. **오른쪽 한계**: `nextPage > maxPage`이면 마지막 페이지이므로 중단

### 필터링 시 초기화 (`filterData`)

3페이지를 보다가 필터를 눌러 데이터가 줄었는데 `pageIndex`가 여전히 2이면, 해당 범위에 데이터가 없어 아무것도 표시되지 않는 버그 발생. 따라서 `pageIndex = 0`(1페이지로 강제 이동)이 필수.
