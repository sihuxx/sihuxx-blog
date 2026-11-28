#nodejs
## server.js

```javascript
var http = require('http');
var fs = require('fs');

var app = http.createServer(function (req, res) {
    res.writeHead(200);
    res.write(fs.readFileSync("index.html"));
    res.end();
    fs.appendFileSync("log.txt", "접속\t" + Date() + "\n") // 접속 기록
})
app.listen(3330); // localhost:3330
console.log("서버 작동");
```

## fs 메소드

### 파일 읽는 메소드

- `readFile`: 비동기
- `readFileSync`: 동기

### 파일 끝부분에 계속 기록하는 메소드

- `appendFile`: 비동기
- `appendFileSync`: 동기

### writeFile

- `fs.writeFileSync`
- `appendFile` 대신 `writeFile` 메소드 사용 시 파일 내용이 누적되어 추가되는 게 아닌, 이전 내용이 없어지고 새로운 내용이 기록

## 동기 / 비동기

- **동기**: 한 줄씩 차례로, 앞의 작업이 끝나야 다음 작업 실행
- **비동기**: 앞의 작업이 끝날 때까지 기다리지 않고, 다음 작업도 바로 실행

## index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    index.html 파일입니다.
</body>
</html>
```

## log.txt

```
접속	Sun Nov 02 2025 17:19:59 GMT+0900 (대한민국 표준시)
접속	Sun Nov 02 2025 17:19:59 GMT+0900 (대한민국 표준시)
```

log.txt 파일: 서버 한번 접속 시마다 두 번씩 기록

---

## 라이브러리 문서 활용

1. node js 공식 홈페이지 접속
2. DOCS 탭 클릭 후 버전 선택
3. 왼쪽 탭에서 찾아보고 싶은 라이브러리 선택해 문서 조회

- 라이브러리 튜토리얼 ~ 메소드 사용법 등 자세한 사항 찾아볼 수 있음
- 간단한 형태 예제 제공