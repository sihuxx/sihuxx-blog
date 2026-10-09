#nodejs
## server.js
```javascript
var http = require('http'); 
var url = require("url"); // url 변수 선언 후 url 라이브러리 불러옴

var app = http.createServer(function(req, res) { 
    // req: 요청, res: 응답
    var URL = req.url;
    var path = url.parse(URL, true).pathname; // pathname 대신 path 입력 시: path와 query 합친 값 불러옴
    var query = url.parse(URL, true).query;
    // 각각 url, path, query 선언 후 불러옴
    // path, query는 .parse를 통해 객체 형태로 저장

    res.writeHead(200); // 페이지 표시 전 

    if (path == "/") { // path 존재 X: create server 출력
        res.write("create server");
    } else if (path == "/data") { 
        res.write("You ID : " + query.id) // query의 어떤 요소 선택할 땐 query.id와 같은 방법으로 불러옴
    } else { // path 아무렇게나 입력 시: Page not found. 출력
        res.write("Page not found.")
    }

    res.end(); // 페이지 표시 후
})
app.listen(80);
console.log("서버 작동"); 
```

## function 내부 매개변수

- `req`: 클라이언트가 서버에 요청하는 개체
- `res`: 서버가 클라이언트에게 응답하는 개체
## url 메소드

- `url.parse(URL, true).pathname`: path 값 불러옴 (`pathname` 대신 `path` 입력 시: path와 query 합친 값 불러옴)
- `url.parse(URL, true).query`: query 값 불러옴
- path, query는 `.parse`를 통해 객체 형태로 저장

## path, query 예시

`http://localhost/data?id=sihu`

- `path` = `/data`
- `query` = `{ id: "sihu" }`

## /data 입력 시 동작

- `localhost` 뒤 `/data` 입력: `You ID : ~~` 출력
- `data` 뒤에 `?id=nodejs` (쿼리문) 입력: `You ID : nodejs` 출력
- `?id`(속성)`=sihu`(값)