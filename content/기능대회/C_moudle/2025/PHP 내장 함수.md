# PHP 내장 함수

## 1. 문자열 처리 (String Functions)

문자열을 자르거나, 합치거나, 검색할 때 사용

- **`explode(구분자, 문자열)`**: 문자열을 구분자로 나누어 배열로 반환

  ```php
  $arr = explode(',', 'apple,banana,orange');
  // 결과: ['apple', 'banana', 'orange']
  ```

- **`implode(구분자, 배열)`**: 배열 요소를 특정 문자로 연결해 하나의 문자열로 반환

  ```php
  $str = implode('-', ['010', '1234', '5678']);
  // 결과: "010-1234-5678"
  ```

- **`strlen(문자열)`**: 문자열의 바이트 길이 반환 (한글은 `mb_strlen` 권장)

  ```php
  $len = strlen('hello');
  // 결과: 5
  ```

- **`str_replace(찾을문자, 바꿀문자, 대상)`**: 특정 문자를 다른 문자로 치환

  ```php
  $text = str_replace(' ', '_', 'hello world');
  // 결과: "hello_world"
  ```

- **`strpos(대상, 찾을문자)`**: 문자열에서 특정 문자가 처음 나타나는 위치(인덱스) 반환

  ```php
  $pos = strpos('abcdef', 'c');
  // 결과: 2 (없으면 false 반환)
  ```

- **`trim(문자열)`**: 문자열 앞뒤의 공백(또는 줄바꿈 문자) 제거

  ```php
  $clean = trim('  hello  ');
  // 결과: "hello"
  ```

- **`sprintf(포맷, 값...)`**: 지정한 포맷으로 문자열 생성 (C언어 스타일)

  ```php
  $code = sprintf('Code-%04d', 7);
  // 결과: "Code-0007"
  ```

## 2. 배열 처리 (Array Functions)

- **`count(배열)`**: 배열의 원소 개수 반환

  ```php
  $cnt = count([1, 2, 3]);
  // 결과: 3
  ```

- **`in_array(값, 배열)`**: 배열에 특정 값이 있는지 확인 (true/false)

  ```php
  $exists = in_array('apple', ['apple', 'banana']);
  // 결과: true
  ```

- **`array_merge(배열1, 배열2)`**: 두 개 이상의 배열을 하나로 합침

  ```php
  $merged = array_merge(['a'], ['b', 'c']);
  // 결과: ['a', 'b', 'c']
  ```

- **`array_keys(배열)`**: 배열의 키(Key)만 추출해 새 배열로 반환

  ```php
  $keys = array_keys(['name' => 'Kim', 'age' => 30]);
  // 결과: ['name', 'age']
  ```

- **`array_map(콜백함수, 배열)`**: 배열의 모든 요소에 함수를 적용

  ```php
  $squared = array_map(fn($n) => $n * 2, [1, 2, 3]);
  // 결과: [2, 4, 6]
  ```

- **`array_shift(배열)`**: 배열 맨 앞 요소를 꺼내 반환하고 제거 (나머지 인덱스는 당겨짐)

  ```php
  $stack = ['first', 'second', 'third'];
  $item = array_shift($stack);
  // 결과: $item은 "first", $stack은 ['second', 'third']
  ```

## 3. 변수 검사 및 디버깅 (Variable Handling & Debugging)

변수의 상태를 확인하거나 타입을 검사

- **`isset(변수)`**: 변수가 선언되었고 `null`이 아닌지 확인

  ```php
  $check = isset($undefined_var);
  // 결과: false
  ```

- **`empty(변수)`**: 변수가 비어 있는지 확인 (`""`, `0`, `null`, `false`, `[]` 등은 true)

  ```php
  $isEmpty = empty([]);
  // 결과: true
  ```

- **`var_dump(변수)`**: 변수의 타입과 값을 자세히 출력 (디버깅용)

  ```php
  var_dump(['a' => 1]);
  // 출력: array(1) { ["a"]=> int(1) }
  ```

- **`is_array(변수)`**, **`is_string(변수)`**: 변수의 데이터 타입 확인

  ```php
  if (is_array([1, 2])) { /* 배열이면 실행 */ }
  ```

## 4. JSON 및 데이터 변환 (Data Format)

API 통신이나 데이터 저장에 자주 사용

- **`json_encode(값)`**: 배열이나 객체를 JSON 문자열로 변환

  ```php
  $json = json_encode(['status' => 'ok']);
  // 결과: '{"status":"ok"}'
  ```

- **`json_decode(JSON문자열, true)`**: JSON 문자열을 배열(true) 또는 객체로 변환

  ```php
  $arr = json_decode('{"a":1}', true);
  // 결과: ['a' => 1]
  ```

## 5. 날짜 및 시간 (Date & Time)

- **`date(포맷)`**: 현재 시간(또는 타임스탬프)을 지정한 형식의 문자열로 반환

  ```php
  $now = date('Y-m-d H:i:s');
  // 결과: "2023-10-25 14:30:00" (실행 시간에 따라 다름)
  ```

- **`strtotime(날짜문자열)`**: 날짜/시간 문자열을 유닉스 타임스탬프(숫자)로 변환

  ```php
  $timestamp = strtotime('+1 day');
  // 결과: 내일 이 시간의 타임스탬프 정수값
  ```

## 6. 정규표현식 (Regular Expressions)

복잡한 문자열 패턴을 검사하거나 치환할 때 사용 (라우팅, 유효성 검사 등)

- **`preg_match(패턴, 문자열, [결과배열])`**: 문자열이 정규식 패턴과 일치하는지 확인 (일치하면 1, 아니면 0)

  ```php
  // 이메일 형식인지 간단 체크
  if (preg_match("/@/", "user@email.com")) {
    echo "이메일 맞음";
  }
  ```

- **`preg_replace(패턴, 바꿀문자, 대상문자열)`**: 정규식 패턴에 맞는 부분을 찾아 다른 문자로 치환

  ```php
  // 모든 공백 제거
  $result = preg_replace("/\s+/", "", "H e l l o");
  // 결과: "Hello"
  ```

## 7. 시스템 및 함수 제어 (System & Function Control)

프레임워크나 라이브러리를 만들 때 주로 사용하는 고급 기능

- **`call_user_func_array(함수, 파라미터배열)`**: 파라미터를 배열로 전달해 함수 실행

  ```php
  function add($a, $b) { return $a + $b; }
  $result = call_user_func_array('add', [10, 20]);
  // 결과: 30 (add(10, 20)과 동일)
  ```
