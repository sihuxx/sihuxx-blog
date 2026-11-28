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

## form 태그 동작

- `input` 태그 내 `name` 속성이 서버에 전달
- `id`, `password` 값은 서버코딩 시 활용
- `button` 태그 `type`을 `submit`이라 작성하면 버튼을 눌렀을 때 `form` 태그 내의 `action`, `method` 요소에 의해 페이지 이동

## method: GET

`http://localhost:3000/login?id=sihu&password=1234`

- `input` 태그 내 `name` 속성은 query문의 요소로 나타남
- `id=sihu&password=1234` -> `id`, `password`: 속성 / `sihu`, `1234`: 속성에 대한 값
- 주소 창에 정보가 나타나는 이유: `form` 태그의 `method` 속성을 GET으로 설정했기 때문

## method: POST

`http://localhost:3000/login`

- `form` 태그 `method` `post`로 설정 시 경로만 변경됨
- 주소창에선 아이디와 비밀번호를 어떤 값을 입력했는지 알 수 없음

---

## server.js

```javascript
var http = require('http');
var fs = require('fs');

var app = http.createServer(function (req, res) {
    res.writeHead(200);
    res.write(fs.readFileSync("form.html"));
    res.end();
})

app.listen(3000);
console.log("서버 작동");
```