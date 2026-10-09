# PHP 라우터와 파라미터 처리

## 1. 라우터 코드

URL 파싱 문제를 보완해, **쿼리 스트링(`?`)** 이나 **해시(`#`)** 가 붙어도 에러 없이 동작하는 코드

```php
<?php

class router {
  static $routes = [];

  // 경로 등록 및 정규식 변환
  static function path($reqM, $uri, $hdl) {
    // {변수}를 정규식 포맷으로 변경
    $uri = preg_replace('#\{(.*?)\}#', '([^\/]+)', $uri);
    return self::$routes[] = [$reqM, "#^$uri$#", $hdl];
  }

  // 요청 처리 및 핸들러 실행 (핵심 부분)
  static function handleRequest() {
    $REQUEST_METHOD = $_SERVER["REQUEST_METHOD"];

    // [중요] URL에서 경로(Path)만 추출 (파라미터, 해시 제거)
    // 예: "/item/list?sort=price" -> "/item/list"
    $path = parse_url($_SERVER["REQUEST_URI"], PHP_URL_PATH);

    foreach(self::$routes as $r) {
      [$reqM, $uri, $hdl] = $r;
      if($reqM !== $REQUEST_METHOD) continue;

      // 추출한 $path와 등록된 $uri 비교
      if(preg_match($uri, $path, $matches)) {
        array_shift($matches); // 전체 매칭 문자열 제거
        call_user_func_array($hdl, $matches); // 핸들러 실행
        return "suc";
      }
    }
    // 일치하는 라우트가 없을 경우
    move("/");
  }
}

// Helper 함수
function get($uri, $hdl) { router::path("GET", $uri, $hdl); }
function post($uri, $hdl) { router::path("POST", $uri, $hdl); }
```

## 2. `parse_url`을 쓰는 이유

`explode('?', ...)`로 문자열을 자르는 것보다 `parse_url(...)`이 나은 이유

1. **안정성**: `/item/list#top`처럼 `#`(앵커)가 붙어도 404 없이 `/item/list`로 정확히 연결
2. **유연성**: `?sort=price&page=2`처럼 뒤에 무엇이 붙든 **경로(Path)** 만 보고 라우팅
3. **표준 준수**: PHP 내장 함수로 URL 표준 규격에 맞게 처리하므로 특수문자 오류 방지

## 3. 실전 사용법 (Controller & View)

라우터가 경로를 잡으면, 실제 데이터(`?sort=...`)는 함수 내부에서 `$_GET`으로 처리

### 라우터 설정 (index.php)

```php
get("/item/list", function() {
  // 1. URL 파라미터 가져오기 (?sort=price 부분)
  // 값이 없으면 기본값('latest') 사용
  $sort = $_GET['sort'] ?? 'latest';
  $page = $_GET['page'] ?? 1;

  // 2. 뷰(화면)에 데이터 전달
  view("item/list", [
    "sort" => $sort,
    "page" => $page
  ]);
});
```

### 뷰 파일 (item/list.php)

```html
<p>현재 정렬 기준: <?php echo $sort; ?></p>
<p>현재 페이지: <?php echo $page; ?></p>
```

## 4. 슬래시(`/`)와 물음표(`?`)의 구분

URL 설계 기준 비교

| 구분 | **Path Variable (슬래시 `/`)** | **Query String (물음표 `?`)** |
| --- | --- | --- |
| **형식** | `/item/view/15` | `/item/list?sort=price` |
| **의미** | **자원의 식별 (Identity)** | **옵션, 필터, 정렬 (Option)** |
| **용도** | 특정 게시물, 카테고리, 회원 | 정렬(Sort), 검색(Search), 페이지(Page) |
| **필수 여부** | **필수** (없으면 페이지가 안 뜸) | **선택** (없으면 기본값으로 표시) |
| **SEO** | 고유 페이지로 인식되어 유리 | 중복 페이지로 볼 수 있음 |
| **예시** | 게시글 상세 `/post/100`<br>사용자 프로필 `/user/chanyouk` | 가격순 정렬 `?sort=price`<br>검색 `?q=사과`<br>페이지 `?page=3` |

- **페이지 자체가 달라지는 경우** (상세 페이지, 카테고리 등): 슬래시(`/`)
- **페이지 내용은 같고 순서/필터만 바뀌는 경우** (정렬, 검색, 페이징): 물음표(`?`)
