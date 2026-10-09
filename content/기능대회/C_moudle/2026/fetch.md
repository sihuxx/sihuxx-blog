# Fetch

## 전송 방식 3가지

### 1. URLSearchParams (단순 데이터)

```javascript
const res = await fetch("/url", {
  method: "POST",
  body: new URLSearchParams({ idx, name })
});
```

| 항목 | 내용 |
| --- | --- |
| 언제 | 단순 key-value 데이터 |
| 장점 | header 자동, 코드가 짧음 |
| 단점 | 파일 불가, 중첩 객체 불가 |

**PHP 전송 형태**

```
idx=1&name=kim
```

```php
$_POST["idx"]  // 1
$_POST["name"] // kim
```

### 2. FormData (파일 포함)

```javascript
const formData = new FormData();
formData.append("idx", idx);
formData.append("file", input.files[0]);

const res = await fetch("/url", {
  method: "POST",
  body: formData
});
```

| 항목 | 내용 |
| --- | --- |
| 언제 | 파일 업로드를 포함할 때 |
| 장점 | 파일 전송 가능, header 자동 |
| 단점 | 코드가 길어짐, 중첩 객체 불가 |

**PHP 전송 형태**

```
--boundary
Content-Disposition: form-data; name="idx"
1
--boundary
Content-Disposition: form-data; name="file"; filename="img.png"
(바이너리 데이터)
```

```php
$_POST["idx"]               // 1
$_FILES["file"]["name"]     // img.png
$_FILES["file"]["tmp_name"] // 임시 저장 경로
```

### 3. JSON (중첩 객체)

```javascript
const res = await fetch("/url", {
  method: "POST",
  headers: {"Content-Type": "application/json"},
  body: JSON.stringify({ idx, user: { name: "kim" } })
});
```

| 항목 | 내용 |
| --- | --- |
| 언제 | 복잡한 데이터, REST API 통신 |
| 장점 | 중첩 객체/배열 가능 |
| 단점 | header 필수, PHP `$_POST` 사용 불가, 파일 불가 |

**PHP 전송 형태**

```json
{"idx": 1, "user": {"name": "kim"}}
```

```php
$data = json_decode(file_get_contents("php://input"));
$data->idx        // 1
$data->user->name // kim
```

## 비교표

| | URLSearchParams | FormData | JSON |
| --- | --- | --- | --- |
| 단순 데이터 | O | O | O |
| 파일 | X | O | X |
| 중첩 객체 | X | X | O |
| header 자동 | O | O | X |
| PHP `$_POST` | O | O | X |

## 응답 처리

```javascript
res.json()   // JSON 파싱
res.text()   // 일반 텍스트
res.blob()   // 파일/이미지
```

## async/await 기본 구조

```javascript
async function send() {
  const res = await fetch("/url", {
    method: "POST",
    body: new URLSearchParams({ idx })
  });
  const data = await res.json();
}
```

> 결과가 나오는 데 시간이 걸리는 작업은 **`await` 필수**
