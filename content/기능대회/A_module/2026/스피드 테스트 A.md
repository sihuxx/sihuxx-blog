# 스피드 테스트 A

## A3

```html
<input type="radio" name="r" id="dark" hidden>
<input type="radio" name="r" id="light" checked hidden>

<label for="dark">다크 모드</label>
<label for="light">라이트 모드</label>
```

```css
body:has(#light:checked) {background-color: #fff;}
body:has(#dark:checked) {background-color: #222;}
#dark:checked ~ [for="dark"] {display: none;}
#light:checked ~ [for="light"] {display: none;}
```

## A4

```html
<form action="">
  <input type="text" required placeholder="이름">
  <input type="email" required placeholder="이메일">
  <input type="password" required placeholder="비밀번호">
  <input type="password" required placeholder="비밀번호 확인">
  <button>회원가입</button>
</form>
```

## A5

```html
<div class="notices">
  <div><div><span class="imp">중요</span><p>2026년 WorldSkills 대회 일정 안내</p></div><p>2026-01-15</p></div>
  <div><div><span>공지</span><p>웹디자인 기술대회 참가신청 마감</p></div><p>2026-01-10</p></div>
  <div><div><span>안내</span><p>온라인 교육과정 개설 안내</p></div><p>2026-01-08</p></div>
  <div><div><span>공지</span><p>2026년 1분기 교육일정 발표</p></div><p>2026-01-05</p></div>
  <div><div><span>안내</span><p>홈페이지 점검 일정 안내</p></div><p>2026-01-03</p></div>
</div>
```

## A6

```html
<footer>
  <div class="top">
    <div class="fc">
      <h3>WorldSkills Korea</h3>
      <p>세계기능올림픽대회를 통해 기능인재 양성과 <br>
        기능의 우수성을 널리 알리고 있습니다.</p>
    </div>
    <div>
      <div>
        <p class="fb">바로가기</p>
        <p>기능경기대회</p>
        <p>교육과정</p>
        <p>자료실</p>
        <p>공지사항</p>
      </div>
      <div>
        <p class="fb">고객지원</p>
        <p>FAQ</p>
        <p>1:1 문의</p>
        <p>이용약관</p>
        <p>개인정보처리방침</p>
      </div>
    </div>
  </div>
  <div class="bot">
    <div>
      <p>주소: 서울특별시 강남구 테헤란로 123 WorldSkills 빌딩</p>
      <p>전화: 02-1234-5678 | 이메일: info@worldskills.kr</p>
    </div>
    <p>© 2026 WorldSkills Korea. All rights reserved.</p>
  </div>
</footer>
```

## A7

```html
<h1>우리의 서비스</h1>
<div class="container">
  <div class="fc">
    <div class="img"></div>
    <div class="title fb">기능경기대회</div>
    <div class="des">국내외 기능경기대회를 통해 기능인재를 발굴하고 우수한 기능의 가치를 널리 알리고 있습니다.</div>
    <div class="btn">자세히 보기</div>
  </div>
  <div class="fc">
    <div class="img"></div>
    <div class="title fb">교육과정</div>
    <div class="des">체계적인 교육과정을 통해 실무능력을 갖춘 전문기능인을 양성하고 있습니다.</div>
    <div class="btn">자세히 보기</div>
  </div>
  <div class="fc">
    <div class="img"></div>
    <div class="title fb">국제교류</div>
    <div class="des">세계 각국과의 기능교류를 통해 글로벌 기능인재 네트워크를 구축하고 있습니다.</div>
    <div class="btn">자세히 보기</div>
  </div>
  <div class="fc">
    <div class="img"></div>
    <div class="title fb">진로지원</div>
    <div class="des">기능인들의 성공적인 진로개발과 취업지원을 위한 다양한 프로그램을 운영합니다.</div>
    <div class="btn">자세히 보기</div>
  </div>
  <div class="fc">
    <div class="img"></div>
    <div class="title fb">기술혁신</div>
    <div class="des">4차 산업혁명 시대에 맞는 첨단기술 교육과 혁신적인 기능개발을 지원합니다.</div>
    <div class="btn">자세히 보기</div>
  </div>
  <div class="fc">
    <div class="img"></div>
    <div class="title fb">연구개발</div>
    <div class="des">기능분야의 지속적인 연구를 통해 새로운 기술과 교육방법을 개발하고 있습니다.</div>
    <div class="btn">자세히 보기</div>
  </div>
</div>
```

## A8

```html
<div class="container fc">
  <header>
    <h2>장바구니</h2>
  </header>
  <main class="fc">
    <div>
      <div>
        <img src="./images/tshirt.jpg" alt="">
        <div class="fc">
          <p class="title">기능경기 가이드북</p>
          <p>종류: 웹디자인</p>
          <p>2026년 최신판</p>
        </div>
      </div>
      <div>
        <div class="count">
          <span class="box">-</span>
          <span>2</span>
          <span class="box">+</span>
        </div>
        <div>
          <p>25,000원</p>
          <span class="del">×</span>
        </div>
      </div>
    </div>
    <div>
      <div>
        <img src="./images/book.jpg" alt="">
        <div class="fc">
          <p class="title">기능경기 가이드북</p>
          <p>종류: 웹디자인</p>
          <p>2026년 최신판</p>
        </div>
      </div>
      <div>
        <div class="count">
          <span class="box">-</span>
          <span>1</span>
          <span class="box">+</span>
        </div>
        <div>
          <p>15,000원</p>
          <span class="del">×</span>
        </div>
      </div>
    </div>
    <div>
      <div>
        <img src="./images/laptop-case.jpg" alt="">
        <div class="fc">
          <p class="title">WorldSkills 노트북 파우치</p>
          <p>색상: 블랙</p>
          <p>15인치 호환</p>
        </div>
      </div>
      <div>
        <div class="count">
          <span class="box">-</span>
          <span>1</span>
          <span class="box">+</span>
        </div>
        <div>
          <p>35,000원</p>
          <span class="del">×</span>
        </div>
      </div>
    </div>
    <hr>
    <div>
      <p>상품금액</p>
      <p>75,000원</p>
    </div>
    <div>
      <p>배송비</p>
      <p>3,000원</p>
    </div>
    <hr>
    <div>
      <p class="fb">총 결제금액</p>
      <p class="fb">78,000원</p>
    </div>
    <div class="btns">
      <button>쇼핑 계속하기</button>
      <button>주문하기</button>
    </div>
  </main>
</div>
```

## A9

```html
<input type="radio" name="slide" id="s1" hidden checked>
<input type="radio" name="slide" id="s2" hidden>
<input type="radio" name="slide" id="s3" hidden>

<div class="slide">
  <ul>
    <li>
      <div class="card"><img src="./images/1.jpg"><h3>삼나무와 초록 밀밭</h3><p class="sub">Cypresses and Green Wheat Field</p><div><p>Vincent van Gogh</p><p>1889</p></div></div>
      <div class="card"><img src="./images/2.jpg"><h3>모델과 화가</h3><p class="sub">The Painter and His Model</p><div><p>Pablo Picasso</p><p>1928</p></div></div>
      <div class="card"><img src="./images/3.jpg"><h3>작은 거리</h3><p class="sub">The Little Street</p><div><p>Johannes Vermeer</p><p>1657–1658</p></div></div>
    </li>
    <li>
      <div class="card"><img src="./images/4.jpg"><h3>빈도 알토비티의 초상</h3><p class="sub">Portrait of Bindo Altoviti</p><div><p>Raphael</p><p>1515</p></div></div>
      <div class="card"><img src="./images/5.jpg"><h3>나르시스의 변형</h3><p class="sub">The Metamorphosis of Narcissus</p><div><p>Salvador Dalí</p><p>1937</p></div></div>
      <div class="card"><img src="./images/6.jpg"><h3>베노아의 성모</h3><p class="sub">The Benois Madonna</p><div><p>Leonardo da Vinci</p><p>1478–1480</p></div></div>
    </li>
    <li>
      <div class="card"><img src="./images/7.jpg"><h3>과일 바구니를 든 소년</h3><p class="sub">Boy with a Basket of Fruit</p><div><p>Caravaggio</p><p>1593–1594</p></div></div>
      <div class="card"><img src="./images/8.jpg"><h3>들쥐</h3><p class="sub">Young Hare</p><div><p>Albrecht Dürer</p><p>1502</p></div></div>
      <div class="card"><img src="./images/9.jpg"><h3>톨레도 풍경</h3><p class="sub">View of Toledo</p><div><p>El Greco</p><p>1596–1600</p></div></div>
    </li>
  </ul>
  <label for="s1" class="nav prev p1">&lt;</label>
  <label for="s2" class="nav next n1">&gt;</label>

  <label for="s1" class="nav prev p2">&lt;</label>
  <label for="s3" class="nav next n2">&gt;</label>

  <label for="s2" class="nav prev p3">&lt;</label>
  <label for="s3" class="nav next n3">&gt;</label>
</div>
```

```css
.nav {display: none;}

#s1:checked ~ .slide ul {transform: translateX(100vw);}
#s2:checked ~ .slide ul {transform: translateX(0vw);}
#s3:checked ~ .slide ul {transform: translateX(-100vw);}

#s1:checked ~ .slide .p1, #s1:checked ~ .slide .n1 {display: flex;}
#s2:checked ~ .slide .p2, #s2:checked ~ .slide .n2 {display: flex;}
#s3:checked ~ .slide .p3, #s3:checked ~ .slide .n3 {display: flex;}
```

## A10

```html
<div class="shape"></div>
```

```css
.shape {width: 420px; height: 420px; background-color: red; animation: ani 4s ease-in-out infinite;}
@keyframes ani {
  0%, 100% {clip-path: polygon(50% 0%, 50% 0%, 100% 100%, 100% 100%, 0% 100%, 0 100%);} /* 삼각형 */
  25% {clip-path: polygon(20% 5%, 80% 5%, 80% 85%, 80% 85%, 20% 85%, 20% 85%);}         /* 사각형 */
  50% {clip-path: polygon(50% 0%, 100% 38%, 82% 100%, 82% 100%, 18% 100%, 0% 38%);}     /* 오각형 */
  75% {clip-path: polygon(25% 0%, 75% 0%, 100% 50%, 75% 100%, 25% 100%, 0% 50%);}       /* 육각형 */
}
```

## A11

```html
<header>
  <input type="checkbox" id="menu" hidden>
  <label class="ham" for="menu">≡</label>
  <nav>
    <a href="#">Home</a>
    <a href="#">About</a>
    <a href="#">Services</a>
    <a href="#">Contact</a>
  </nav>
</header>
```

```css
header {background-color: antiquewhite; height: 100px; display: flex; align-items: center; justify-content: center;}
nav {display: flex; align-items: center; justify-content: space-between; width: 400px; gap: 20px;}
.ham {display: none;}
nav a {padding: 15px 0;}
@media (max-width: 768px) {
  nav {display: none; flex-direction: column; position: absolute; top: 100px; left: 0; width: 100%; background-color: antiquewhite;}
  .ham {display: block;}
  #menu:checked ~ nav {display: flex;}
}
```

## A12

```html
<div class="circle circle01"></div>
<div class="circle circle02"></div>
<div class="circle circle03"></div>
<div class="circle circle04"></div>
```

```css
.circle { width: 30px; height: 30px; position: absolute; background-color: black; border-radius: 50%; }
.circle01 { animation: loading 2s infinite; background-color: rgb(255, 49, 49); }
.circle02 { animation: loading02 2s infinite; background-color: rgb(255, 174, 0); }
.circle03 { animation: loading03 2s infinite; background-color: rgb(0, 190, 143); }
.circle04 { animation: loading04 2s infinite; background-color: rgb(22, 51, 42); }

@keyframes loading {
  0% { transform: translate(0, 0); }
  25% { transform: translate(-50px, 0px); }
  50% { transform: translate(-50px, 50px); }
  75% { transform: translate(0px, 50px); }
  100% { transform: translate(0, 0); }
}
@keyframes loading02 {
  0% { transform: translate(-50px, 0px); }
  25% { transform: translate(-50px, 50px); }
  50% { transform: translate(0px, 50px); }
  75% { transform: translate(0, 0); }
  100% { transform: translate(-50px, 0px); }
}
@keyframes loading03 {
  0% { transform: translate(-50px, 50px); }
  25% { transform: translate(0px, 50px); }
  50% { transform: translate(0px, 0px); }
  75% { transform: translate(-50px, 0px); }
  100% { transform: translate(-50px, 50px); }
}
@keyframes loading04 {
  0% { transform: translate(0px, 50px); }
  25% { transform: translate(0px, 0px); }
  50% { transform: translate(-50px, 0px); }
  75% { transform: translate(-50px, 50px); }
  100% { transform: translate(0px, 50px); }
}
```

## A13

```html
<input type="radio" hidden name="check" id="login">
<label for="login">로그인</label>
<div class="notices">
  <p>계정 로그인</p>
  <p>안전한 서비스를 위해 로그인이 필요합니다.</p>
  <p>계속하려면 아래에 등록된 이메일과 비밀번호를 입력해주세요.</p>
  <p>로그인 계속</p>
  <input type="radio" hidden name="check" id="close">
  <label for="close">닫기</label>
</div>
```

```css
body:has(#close:checked) > .notices {display: none;}
#login:checked ~ .notices {display: flex;}
```

## A14

```html
<ul>
  <button class="tip1" data-tip="설정 버튼을 눌러 서비스를 설정할 수 있습니다">설정</button>
  <button class="tip2" data-tip="로그아웃되고 메인 페이지로 이동합니다">로그아웃</button>
</ul>
```

```css
[class*="tip"] {position: relative; cursor: pointer;}
.tip1::after {opacity: 0; content: attr(data-tip); width: max-content; height: 20px; position: absolute; font-size: 12px; transform: translate(-50%, 160%);}
.tip1:hover::after {opacity: 1;}
.tip1:focus::after {opacity: 1;}
.tip2::after {opacity: 0; content: attr(data-tip); width: max-content; height: 20px; position: absolute; font-size: 12px; transform: translate(-50%, 160%);}
.tip2:hover::after {opacity: 1;}
.tip2:focus::after {opacity: 1;}
```

## A15

```html
<div class="container">
  <img src="./card1.jpg">
  <img src="./card2.jpg">
  <img src="./card3.jpg">
  <img src="./card4.jpg">
</div>
```

```css
.container {display: grid; grid-template-columns: repeat(4, 1fr);}
@media (min-width: 768px) and (max-width: 1023px) {.container {grid-template-columns: repeat(2, 1fr);}}
@media (max-width: 767px) {.container {grid-template-columns: repeat(1, 1fr);}}
```

## A16

```html
<div class="container">
  <header>
    <div class="step1"></div>
    <div class="line1"></div>
    <div class="step2"></div>
    <div class="line2">
      <div class="sline"></div>
      <div class="sline"></div>
    </div>
    <div class="step3"></div>
  </header>
  <main>
    <div class="box">
      <div class="sub">STEP 1</div>
      <div class="title">Card Details</div>
      <div class="state1">Completed</div>
    </div>
    <div class="box">
      <div class="sub">STEP 2</div>
      <div class="title">Form Reviews</div>
      <div class="state2">In Progress</div>
    </div>
    <div class="box">
      <div class="sub">STEP 3</div>
      <div class="title">Authentication</div>
      <div class="state3">Pending</div>
    </div>
  </main>
</div>
```
