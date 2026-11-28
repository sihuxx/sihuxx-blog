#nodejs
## form.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <form action="/login" method="post">
        아이디 <input name="id" type="text">
        비밀번호 <input name="password" type="password">
        <button type="submit">로그인</button>
    </form>
</body>

</html>
```

## server.js

```javascript
var http = require('http')
var fs = require('fs')
var url = require('url')
var querystring = require('querystring')

var app = http.createServer(function(req, res) {
    var URL = req.url
    var path = url.parse(URL, true).pathname

    if (path == "/") { // path가 없을 때
        res.writeHead(200) // 서버 상태 전송
        res.write(fs.readFileSync("form.html")) // 브라우저에서 localhost:550 접속 시 form.html 파일 읽은 후 화면에 씀
    } if (path == "/login") {  // path가 있을 때 (form 태그에서 submit 버튼을 통해 login path가 붙었을 때)
        var body = '' // POST 요청으로 들어오는 데이터를 문자열로 저장할 변수
        var query = ''

        req.on('data', function(data) { // data 읽을 때 수행하는 메소드
            body += data;
        })
        req.on('end', function() { // data를 다 읽었을 때 수행하는 메소드
            query = querystring.parse(body) // body 문자열을 객체 형태로 변환
            res.writeHead(200, {'Content-Type':'text/html; charset=utf8'})
            res.write("아이디 : " + query.id + '<br>비밀번호 : ' + query.password)
            res.end()
        })
    }
})
app.listen(550) // localhost:550
console.log("서버 작동");
```

form.html에서 submit을 하면 login으로 id와 password 넘겨줌

## querystring 라이브러리

POST 방식의 BODY를 가져올 때 이용하는 라이브러리

## req.on 메소드

- `req.on('data', ...)`: data 읽을 때 수행하는 메소드
    - 서버는 POST 요청 데이터를 조각(chunk) 단위로 받음
    - 조각이 여러 번 올 수 있음 -> 매번 들어오는 조각 `+=`로 이어 붙임
- `req.on('end', ...)`: data를 다 읽었을 때 수행하는 메소드
    - `querystring.parse(body)`: body 문자열을 객체 형태로 변환
    - `{'Content-Type':'text/html; charset=utf8'}`: Content-Type을 HTML + UTF-8로 설정 => 한글 깨지지 않음
    - form 태그 input의 `name` 요소(속성): input 안 값(값) 형태를 `query` 변수에 저장하여 불러옴

---

## GET

```
HEADER
URI https://host/path?id=abc&password=1234
```

GET방식은 URI가 query문 포함하기 때문에 URI 이용해서 데이터 가져오기 가능

## POST

```
HEADER
URI https://host/path
BODY id:abc, password:1234
```

POST방식은 query문이 URI에 없고 BODY 영역에 있어 BODY의 내용을 따로 읽는 방식으로 데이터를 가져와야 함