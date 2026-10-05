#nodejs 
## server.js

```javascript
var http = require('http'); // http라는 변수 선언 후 라이브러리 불러옴

var app = http.createServer(function(req, res) { 
    // app이라는 변수 선언 후, createServer 메소드로 서버 생성

    res.writeHead(200); // wrtieHead 메소드 통해 서버 상태 전송 (200: 괜춘, 404: 서버 찾을 수 X)
    res.write("create server"); // Write 메소드 통해 클라이언트에 메소드 전송
    res.end(); // end 메소드로 응답 종료
})
app.listen(80); // listen 메소드: 서버에서 클라이언트의 접속을 기다리는 기능
console.log("서버 작동");
```
## http 메소드

|메소드|설명|
|---|---|
|`createServer`|서버 생성 (`app`이라는 변수 선언 후 사용)|
|`writeHead`|서버 상태 전송 (200: 괜춘, 404: 서버 찾을 수 X)|
|`write`|클라이언트에 내용 전송|
|`end`|응답 종료|
|`listen`|서버에서 클라이언트의 접속을 기다리는 기능|

## 포트 번호

- `80`: 통신포트 번호, http의 경우 Default 포트번호가 80번
- 만약 괄호 안에 3330을 써넣었다면 `localhost:3330`을 통해 서버 접속 가능

## 실행 / 종료

- `node server.js`: 서버 작동
- `ctrl + c`: 서버 종료

---

## request (요청문)

클라이언트가 서버에 "이런 내용을 줘"라고 요청하는 문장

### 요청문 구조

`스키마 :// IP주소(도메인) : 포트번호 / 경로 ? 쿼리문`

### 네이버에서 세종대왕을 서치했을 때

- `https :// www.naver.com :`
- `https :// search.naver.com` (변경된 도메인) `: search.naver` (경로) `? where ~~... query=세종대왕` (쿼리문)

### 쿼리문

- `속성 = 값 & 속성 = 값 ...` 의 구조로 이루어진 문장
- 이렇게 http 규칙으로 서버에 요청하는 요청문 = **URL**

---

## 클라이언트가 요청문 (request)을 서버에 전송

- **GET문**: 검색창처럼 세부사항이 보여진 채 요청하는 방식
- **POST문**: 쿼리문을 숨긴 채 요청하는 방식 (아이디/비밀번호가 보이면 안되는 로그인과 같은 경우)

서버는 클라이언트가 요청 (request)하면 요청한 세부내용에 따라서 페이지/다운로드 등을 클라이언트에 제공하는 구조