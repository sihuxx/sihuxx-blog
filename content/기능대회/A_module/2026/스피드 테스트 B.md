# 스피드 테스트 B

## lib

```js
const $ = (e) => document.querySelector(e);
const $$ = (e) => [...document.querySelectorAll(e)];
const newEl = (a, t) => Object.assign(document.createElement(a), t)
```

## B1

```js
const input = $("input")
const getPsw = pw => {
  const l = pw.length
  const u = /[A-Z]/.test(pw)
  const n = /[0-9]/.test(pw)
  const s = /[!@#$%^&*]/.test(pw)

  input.style.borderColor =
    !l ? "black" :
    (l >= 8 && u && n && s) ? "green" :
    (l >= 6 && (u || n)) ? "yellow" :
    "red"
}
input.addEventListener("input", (event) => {
  getPsw(input.value)
})
```

### 로직

- 변수: `l` 암호 길이, `u` 대문자 포함 여부, `n` 숫자 포함 여부, `s` 특수문자 포함 여부
- 테두리 색 결정 순서
  - 입력 없음: `검은색`
  - 길이 8 이상 + 대문자 + 숫자 + 특수문자 모두 포함: `초록색`
  - 길이 6 이상 + 대문자 또는 숫자 포함: `노란색`
  - 그 외: `빨간색`
- `input` 이벤트가 발생할 때마다 위 함수 실행
- input에 `outline: none;` 처리 필수

## B2

```js
const input = $("input");
const addBtn = $(".addBtn");
const list = $("ul");
addBtn.addEventListener("click", () => {
  const li = newEl("li", {
    innerHTML: `<p>${input.value}</p><button class="des" onclick="parentElement.remove()">삭제</button>`
  })
  list.appendChild(li)
  input.value = ""
})
```

## B3

```js
let x = 0, y = 0;
const box = $(".box");
const moves = {
  ArrowUp: [0, -10], ArrowDown: [0, 10],
  ArrowLeft: [-10, 0], ArrowRight: [10, 0]
}
window.addEventListener('keydown', (e) => {
  if(!moves[e.key]) return;
  const [dx, dy] = moves[e.key]
  const limitX = (window.innerWidth - box.offsetWidth) / 2
  const limitY = (window.innerHeight - box.offsetHeight) / 2
  x = Math.max(-limitX, Math.min(limitX, x + dx));
  y = Math.max(-limitY, Math.min(limitY, y + dy));
  box.style.transform = `translate(${x}px, ${y}px)`
})
```

### 로직

- **좌표**: `x`, `y` 변수로 현재 위치 관리
- **방향 매핑**: `moves` 객체에 화살표 키별 이동 거리 저장
- **키 확인**: 화살표 키가 아니면 `return`으로 중단
- **범위 제한**: `limitX`, `limitY`를 계산해 화면 밖으로 나가지 않도록 차단
- **좌표 갱신**: `Math.max`, `Math.min`을 조합한 clamp 로직으로 제한 범위 안에서만 `x`, `y` 변경
- **렌더링**: 계산된 `x`, `y`를 CSS `transform: translate()`에 대입해 박스 이동

## B4

```js
const time = $(".time")
setInterval(() => time.textContent = new Date().toLocaleTimeString().slice(2), 10)
```

### 로직

- 10밀리초마다 현재 시간을 가져와 '오전/오후' 글자를 `slice`로 제거하고 화면에 실시간 갱신

## B5

```js
const input = $("input")
const btn = $("button")

btn.addEventListener("click", () => {
  document.body.style.backgroundColor = input.value
})
```

## B6

```js
const $itemZone = $(".items")
const $items = $$(".dragItem li")
const $dragZone = $(".dragZone")

$items.forEach(item => {
  item.setAttribute('draggable', true)
  item.addEventListener("dragstart", (e) => {
    e.dataTransfer.setData("text/html", "")
    e.currentTarget.classList.add("dragging")
  })
});

$dragZone.addEventListener("dragover", (e) => {e.preventDefault()})

$dragZone.addEventListener("drop", (e) => {
  e.preventDefault()
  const draggingItem = document.querySelector(".dragging")
  if(draggingItem) $dragZone.appendChild(draggingItem)
  $("p").style.display = 'none'
})

$(".init").addEventListener("click", () => {
  $(".dragItem").append(...$$("li"));
  $("p").style.display = 'block';
});
```

### 로직

1. **dragstart (출발)**
   - `e.dataTransfer.setData`로 드래그 데이터 설정
   - 끌고 있는 요소(`currentTarget`)에 `.dragging` 클래스를 붙여 움직이는 요소를 표시
2. **dragover (경로)**
   - `e.preventDefault()`로 해당 영역에 놓는 것을 허용
3. **drop (도착)**
   - `.dragging` 클래스로 대상을 찾아 존재하면 `$dragZone` 안으로 이동(`appendChild`)
   - 화면의 안내 문구(`<p>`)는 숨김
4. **init (복구)**
   - 드래그 존에 들어간 `li`를 원래 목록(`.dragItem`)으로 복귀
   - 숨겼던 안내 문구를 `block`으로 다시 표시

## B7

```js
const inputs = $$("input")

inputs.forEach((input, i) => {
  input.addEventListener("keydown", (e) => {
    if(e.key === "Backspace") {
      if(i > 0 && input.value === "") {
        inputs[i - 1].focus()
      }
    }
  })
  input.addEventListener("input", () => {
    input.value = input.value.replace(/[^0-9]/g, "")
    if(input.value.length === 1 && i < inputs.length - 1) {
      inputs[i + 1].focus()
    }
    if(inputs.every(input => input.value.length > 0)) {
      $("button").style.display = 'block'
    }
  })
})
```

### 로직

1. **백스페이스 입력 시**
   - 현재 칸이 비어 있고 첫 번째 칸이 아니면 이전 입력창(`inputs[i - 1]`)으로 포커스 이동
2. **값 입력 시**
   - `replace(/[^0-9]/g, "")`로 숫자만 남김
   - 한 글자가 입력되었고 마지막 칸이 아니면 다음 입력창(`inputs[i + 1]`)으로 포커스 이동
3. **전체 입력 확인**
   - 모든 input이 채워지면 숨겨진 버튼을 표시

## B8

```js
const logs = $(".log")

function createNotice(value) {
  const newDiv = newEl("div", {
    textContent: value === "grn" ? "성공하였습니다" : "실패하였습니다",
    className: value
  })
  logs.append(newDiv)

  // 클릭 시 메시지 삭제
  newDiv.addEventListener("click", () => {
    newDiv.remove()
  })
  // 5초 후 메시지 삭제
  setTimeout(() => {
    newDiv.remove()
  }, 5000)
}
```

```css
.grn {background-color: greenyellow;}
.pink {background-color: pink;}
```

### 로직

- **알림 생성 (`createNotice`)**
  - `value`가 `grn`이면 "성공하였습니다", 아니면 "실패하였습니다"를 담은 div 생성
  - CSS 클래스명에 `value`를 그대로 사용해 색상 처리
- 클릭하면 삭제, 5초 뒤 자동 삭제

## B9

```js
class star {
  constructor() {
    this.datas = JSON.parse(localStorage.getItem("star")) ?? this.getData()
    this.render()
  }
  update(newData) {
    this.datas = newData
    this.render()
    localStorage.setItem("star", JSON.stringify(this.datas))
  }
  render() {
    container.innerHTML = ''

    this.datas.map(e => {
      const newDiv = newEl("div", {
        innerHTML: `<h1>${e.name}</h1><p>${e.desc}</p><div class="star">${e.isStar ? "★" : "☆"}</div>`
      })
      newDiv.querySelector(".star").addEventListener('click', () => {
        e.isStar = !e.isStar
        this.update(this.datas)
      })
      container.append(newDiv)
    })
  }
  async getData() {
    const data = await fetch("./data.json").then(res => res.json())
    this.update(data)
  }
}

new star
```

### 로직

1. **초기 세팅 (`constructor`)**
   - `localStorage`에 저장된 데이터가 있으면 사용, 없으면 `getData()`로 새로 가져옴
   - 이후 `render()`로 목록 출력
2. **데이터 가져오기 (`getData`)**
   - 로컬 데이터가 없을 때 실행되는 비동기(`async`) 함수
   - 외부 JSON을 가져와 `update()` 호출
3. **화면 그리기 (`render`)**
   - 기존 리스트를 비우고 `this.datas`를 `map`으로 순회하며 출력
   - 별표(★/☆) 클릭 시 `isStar` 값을 반전(`true` ↔ `false`)하고 `update()` 실행
4. **저장 및 갱신 (`update`)**
   - 데이터가 바뀔 때마다 호출
   - 변경 데이터를 `localStorage`에 JSON으로 저장하고 `render()`로 화면 갱신

## B10

```js
const canvas = document.querySelector("canvas");
const ctx = canvas.getContext("2d");
const saveBtn = document.getElementById("save")
const clearBtn = document.getElementById("clear")
ctx.lineWidth = 2.5;
let painting = false;

function stopPainting() {
  painting = false;
}
function startPainting() {
  painting = true;
  ctx.beginPath();
}
function onMouseMove(event) {
  const x = event.offsetX;
  const y = event.offsetY;
  if (!painting) {
    ctx.moveTo(x, y);
  } else {
    ctx.lineTo(x, y);
    ctx.stroke();
  }
}
saveBtn.addEventListener("click", () => {
  const a = document.createElement("a")
  a.href = canvas.toDataURL("image/png")
  a.download = "image.png"
  a.click()
})
clearBtn.addEventListener("click", () => {
  ctx.clearRect(0, 0, canvas.width, canvas.height)
})
canvas.addEventListener("mousemove", onMouseMove);   // 마우스 움직일 때
canvas.addEventListener("mousedown", startPainting); // 마우스 누를 때
canvas.addEventListener("mouseup", stopPainting);    // 마우스 뗄 때
canvas.addEventListener("mouseleave", stopPainting); // 캔버스 밖으로 나갔을 때
```

### 로직

1. **그리기 상태 관리 (`painting`)**
   - 마우스를 누르면(`mousedown`) `true`, 떼거나(`mouseup`) 캔버스를 벗어나면(`mouseleave`) `false`
2. **선 그리기 (`onMouseMove`)**
   - `offsetX`, `offsetY`로 현재 마우스 위치를 가져옴
   - 마우스만 움직일 때(`!painting`): 시작점(`moveTo`)을 마우스 위치로 이동
   - 누르고 움직일 때: 이전 위치에서 현재 위치까지 선을 잇고(`lineTo`) 실제로 그림(`stroke`)
3. **이미지 저장 (`saveBtn`)**
   - 캔버스 내용을 데이터 URL(`canvas.toDataURL("image/png")`)로 변환
   - 임시 `<a>` 태그를 만들어 클릭하면 `image.png`로 다운로드
4. **캔버스 비우기 (`clearBtn`)**
   - `clearRect`로 캔버스 전체 영역(`0, 0, canvas.width, canvas.height`)을 지워 초기화

## B11

```js
let startTime = 0, elapsed = 0, id = null, run = false

function timer(now) {
  if(!run) return

  const time = now - startTime + elapsed

  minutes.innerHTML = String(Math.floor(time / 60000)).padStart("2", 0)
  second.innerHTML = String(Math.floor((time % 60000) / 1000)).padStart("2", 0)
  mili.innerHTML = String(Math.floor(time % 1000)).padStart("3", 0)

  id = requestAnimationFrame(timer)
}

function render() {
  run = !run

  if(run) {
    startTime = performance.now()
    container.innerHTML = `<button onclick="render()">중지</button>`
    id = requestAnimationFrame(timer)
  } else {
    elapsed += performance.now() - startTime
    cancelAnimationFrame(id)
    container.innerHTML = `<button onclick="render()">계속</button>`
  }
}
```

### 전역 변수

- `startTime`: 버튼을 누른 기준 시점
- `elapsed`: 중지할 때까지 누적된 이전 기록
- `run`: 시계가 동작 중인지(`true`) 멈췄는지(`false`)

### `render()` - 시작/중지 전환

버튼을 클릭할 때 실행되며 시계의 모드를 전환

- **시작 시 (`run: true`)**
  - `startTime`을 현재 시각으로 고정
  - `timer` 함수를 호출해 시계 동작 시작
- **중지 시 (`run: false`)**
  - 핵심: `현재 - 시작`만큼 흐른 시간을 `elapsed`에 누적
  - `cancelAnimationFrame`으로 반복 종료

### `timer(now)` - 계산과 출력

브라우저가 약 16ms마다 자동 실행하며 화면을 실시간으로 갱신

- **시간 계산**: `(지금 시각 - 시작 시각) + elapsed`, 멈췄던 지점부터 이어서 흐르게 함
- **단위 변환**: 분(`/60000`), 초(`%60000 / 1000`), 밀리초(`%1000`)로 분리
- **화면 출력**: `padStart`로 `00:00:000` 형태 구성
- **반복**: 함수 끝에서 `requestAnimationFrame`으로 자기 자신을 다시 예약

## B12

```js
const file = $("#file")
const img = $("#img");
const btns = $("#btns")

file.onchange = (e) => img.src = URL.createObjectURL(e.target.files[0])
btns.onclick = (e) => e.target.dataset.filter && (img.style.filter = e.target.dataset.filter)
```

### 로직

1. **이미지 미리보기 (`onchange`)**
   - 파일을 선택하면 `URL.createObjectURL`로 임시 주소 생성
   - `URL.createObjectURL`: 내 컴퓨터의 파일을 브라우저에서만 쓸 수 있는 임시 주소로 변환하는 기능
   - 이 주소를 `img`의 `src`에 넣어 선택한 사진을 바로 표시
2. **필터 적용 (`onclick`)**
   - 버튼 영역(`btns`)에 이벤트 위임을 사용해, 클릭된 요소에 `dataset.filter`(HTML의 `data-filter`)가 있는지 확인
   - 값이 있으면 이미지의 CSS `filter`로 적용 (예: `grayscale(100%)`)

## B13

```js
const video = $("video")
$(".play").onclick = () => video.play()                   // 재생
$(".stop").onclick = () => video.pause()                  // 일시정지
$(".tenPlus").onclick = () => video.currentTime += 10     // 10초 빨리감기
$(".tenMin").onclick = () => video.currentTime -= 10      // 10초 되감기
$(".sound").onclick = () => video.muted = !video.muted    // 음소거
```

## B14

```js
const [canvas, legend, lbl, val] = ["#canvas", ".legend", "#labelInput", "#valueInput"].map($);
const ctx = canvas.getContext("2d");
let data = [];

$("button").onclick = () => {
  if (!lbl.value || val.value <= 0) return;
  data.push({ k: lbl.value, v: +val.value, c: `hsl(${Math.random() * 360}, 70%, 60%)` });
  [lbl.value, val.value] = ["", ""];
  render();
};

function render() {
  const total = data.reduce((s, e) => s + e.v, 0);
  let start = -Math.PI / 2;
  legend.innerHTML = "";
  ctx.clearRect(0, 0, 500, 500);

  data.forEach(({k, v, c}) => {
    const angle = (v / total) * Math.PI * 2;

    ctx.beginPath();
    ctx.moveTo(250, 250);
    ctx.arc(250, 250, 150, start, start + angle);
    ctx.fillStyle = c;
    ctx.fill();

    legend.innerHTML += `
      <div class="legend-item">
        <div class="legend-color" style="background:${c}"></div>
        <span>${k} (${(v / total * 100).toFixed(1)}%)</span>
      </div>`;

    start += angle;
  });
}
```

### 로직

1. **데이터 저장 (`onclick`)**
   - 검사 및 변환: 항목명과 값이 있을 때만 `data` 배열에 추가, 앞의 `+`로 입력값을 숫자로 변환
   - 색상: `hsl`과 랜덤 함수로 항목마다 다른 색 생성
   - 초기화: 입력창을 비움
2. **그래프 계산 (`render`)**
   - 초기화: `clearRect`로 이전 그림을 지워 겹침 방지
   - 각도: 전체 합계(`total`) 대비 비율을 구해 원 전체(2π)에서 차지할 각도를 결정
3. **그리기 및 범례 (`render`)**
   - 부채꼴: 중심(`250, 250`)에서 `arc`로 조각을 그리고, `start`에 끝 각도를 누적해 다음 조각을 이어 붙임
   - 범례: `innerHTML`로 색상 박스와 퍼센트(`toFixed(1)`)를 목록으로 출력

## B15

```js
const $ = (selector) => document.querySelector(selector);
const $$ = (selector) => document.querySelectorAll(selector);

let state = { limit:10, page:1 }
const datas = await fetch('./sample-data.csv').then(res => res.text());
const rows = datas.split('\n').slice(1).map( line => `<tr>${line.split(',').map( cell => `<td>${cell}</td>` ).join('')}</tr>` );

const $tableBody = $('#tableBody');
const $paginationButtons = $$('.pagination-btn');
const $prevBtn = $('.prev-btn');
const $nextBtn = $('.next-btn');

function setState(newState){
  state = { ...state, ...newState }; // page가 새로 저장되게 스프레드로 합침
  render();
}

$paginationButtons.forEach( $btn => $btn.addEventListener('click', () => {
  setState({ page: Number($btn.textContent) }); // 페이지 버튼
}));

$prevBtn.addEventListener('click', () => {
  setState({ page: state.page - 1 }); // 이전 버튼
});

$nextBtn.addEventListener('click', () => {
  setState({ page: state.page + 1 }); // 다음 버튼
});

function render(){
  const range = state.limit * (state.page - 1);
  $tableBody.innerHTML = rows.slice(range, range + state.limit).join('');
  // 1페이지: slice(0, 10) → 0 ~ 9번 데이터
  // 2페이지: slice(10, 20) → 10 ~ 19번 데이터

  $prevBtn.disabled = state.page === 1; // 현재 페이지가 1일 때 이전 버튼 비활성화
  $nextBtn.disabled = state.page === 5; // 현재 페이지가 5일 때 다음 버튼 비활성화

  $paginationButtons.forEach(($btn,index) => {
    $btn.classList.remove('active');
    $btn.classList.remove('page-info');

    $btn.textContent = index + 1;
    if(state.page === index + 1) $btn.classList.add('active');
    if(state.page ===  1 && index === 1 || state.page === 5 && index === 3){
      $btn.classList.add('page-info');
      $btn.textContent = '...'
    }
  })
}

render()
```

### 로직

1. **데이터 파싱 및 가공**
   - CSV 읽기: `fetch`로 가져온 텍스트를 줄 바꿈(`\n`)과 콤마(`,`) 기준으로 분리해 `<tr>`, `<td>` 문자열로 미리 생성
   - 마지막 페이지(5페이지)는 코드에 고정값으로 지정됨
2. **상태 관리 (`setState`)**
   - 페이지 번호를 바꿀 때마다 `state`를 갱신하고 `render()` 호출
3. **화면 그리기 (`render`)**
   - 데이터 슬라이싱: 전체 행 중 현재 페이지 범위(`range` ~ `range + limit`)만 잘라 테이블 본문에 삽입
   - 버튼 비활성화: 1페이지면 '이전', 5페이지면 '다음' 버튼을 `disabled`
   - 페이지 번호 표시: 현재 페이지에 `active` 클래스를 붙이고, 첫/끝 페이지에서는 일부 번호를 `...`으로 표시
4. **이벤트 연결**
   - 숫자 버튼, 이전 버튼, 다음 버튼에 클릭 이벤트를 걸어 페이지 번호를 지정하거나 증감

## B16

```js
const { todos } = await fetch('./todos.json').then( res => res.json() );
const services = {
  "전체": () => todos,
  "진행중": () => todos.filter(todo => !todo.completed),
  "완료": () => todos.filter(todo => todo.completed),
  "높은 우선순위": () => todos.filter(todo => todo.priority === 'high'),
};
const priority = {
  'high': {'class': 'priority-high', 'text': '높음'},
  'medium': {'class': 'priority-medium', 'text': '보통'},
  'low': {'class': 'priority-low', 'text': '낮음'},
}

let state = { activeFilter: '전체' };
const $todoList = $('#todoList');
const $filterButtons = $$('.filter-btn');

$('#totalCount').textContent = todos.length;
$('#completedCount').textContent = services['완료']().length;
$('#pendingCount').textContent = services['진행중']().length;

$filterButtons.forEach( $btn => $btn.addEventListener('click', () => {
  state.activeFilter = $btn.textContent;
  render();
}));

function render(){
  $filterButtons.forEach( $btn =>  $btn.classList.toggle('active', $btn.textContent === state.activeFilter))
  $todoList.innerHTML = '';
  const $todoItems = services[state.activeFilter]().map( todo =>
    createElement('div',{
      className:`todo-item ${todo.completed ? 'completed' : ''}`,
      innerHTML: `
        <div class="todo-header">
          <h3 class="todo-title">${todo.title}</h3>
          <div class="todo-badges">
            <span class="badge ${priority[todo.priority].class}">${priority[todo.priority].text}</span>
            <span class="badge status-badge">${todo.completed ? '완료' : '진행중'}</span>
          </div>
        </div>
        <p class="todo-description">${todo.description}</p>
        <div class="todo-footer">
          <div class="date-info">
            <span>마감: ${todo.dueDate}</span>
            <span>생성: ${todo.createdAt}</span>
          </div>
        </div>
      `
    })
  );
  $todoList.append(...$todoItems);
}

render()
```

### 로직

1. **데이터 준비 및 초기화**
   - 데이터 로드: `fetch`로 JSON을 가져와 `todos` 배열에 저장
   - 필터 엔진 (`services`): "전체", "진행중", "완료", "높은 우선순위" 조건별 필터 함수를 객체로 정리
   - 우선순위 매핑: `high`, `medium`, `low`를 한글 텍스트와 CSS 클래스로 변환하는 객체 정의
2. **상태 및 통계 업데이트**
   - 대시보드 숫자: 로드 시 전체, 완료, 진행 중 개수를 계산해 상단에 표시
   - 상태 관리: 현재 선택된 필터를 `state.activeFilter`에 저장
3. **필터링 및 렌더링 (`render`)**
   - 버튼 활성화: 선택한 필터 버튼에만 `active` 클래스 부여
   - 요소 생성: 선택된 필터의 데이터를 `map`으로 순회하며 `createElement`로 HTML 구조 생성
   - 화면 반영: `innerHTML = ''`로 기존 목록을 비우고 새 항목을 한 번에 추가
