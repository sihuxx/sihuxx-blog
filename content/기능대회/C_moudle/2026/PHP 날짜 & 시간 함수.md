# PHP 날짜/시간 함수

## 1. `time()`

현재 Unix 타임스탬프(정수) 반환

```php
$ts = time(); // 예: 1717372800
```

- 반환값: `int` (1970-01-01 00:00:00 UTC 기준 경과 초)
- 인수 없음

## 2. `date(string $format, ?int $timestamp = null)`

타임스탬프를 지정한 포맷의 **문자열**로 변환

```php
echo date('Y-m-d H:i:s');               // 2026-06-03 12:00:00 (현재)
echo date('Y-m-d', 1717372800);         // 특정 타임스탬프 포맷
echo date('D, d M Y', strtotime('next monday')); // Mon, 08 Jun 2026
```

### 주요 포맷 문자

| 문자 | 의미 | 예시 |
| --- | --- | --- |
| `Y` | 4자리 연도 | 2026 |
| `y` | 2자리 연도 | 26 |
| `m` | 월 (01–12) | 06 |
| `n` | 월 (1–12, 선행 0 없음) | 6 |
| `d` | 일 (01–31) | 03 |
| `j` | 일 (1–31, 선행 0 없음) | 3 |
| `H` | 시 24h (00–23) | 14 |
| `i` | 분 (00–59) | 05 |
| `s` | 초 (00–59) | 09 |
| `D` | 요일 약자 | Mon |
| `l` | 요일 전체 | Monday |
| `N` | ISO 요일 숫자 (1=월 ~ 7=일) | 3 |
| `U` | Unix 타임스탬프 | 1717372800 |
| `t` | 해당 월의 총 일수 | 30 |
| `W` | ISO 주차 | 23 |

## 3. `strtotime(string $datetime, ?int $baseTimestamp = null)`

날짜 **문자열**을 Unix 타임스탬프 `int`로 변환 (실패 시 `false`)

```php
strtotime('2026-06-03');           // 1748908800
strtotime('+7 days');              // 현재 + 7일
strtotime('next monday');          // 다음 월요일
strtotime('last day of this month'); // 이번 달 마지막 날
strtotime('-1 month', strtotime('2026-03-31')); // 2026-02-28
```

### 지원하는 표현식 예시

| 표현 | 의미 |
| --- | --- |
| `+1 day` / `-3 weeks` | 상대적 가감 |
| `next monday` | 다음 특정 요일 |
| `last friday` | 지난 특정 요일 |
| `first day of next month` | 다음 달 1일 |
| `last day of february 2026` | 특정 월 마지막 날 |
| `yesterday` / `today` / `tomorrow` | 어제/오늘/내일 |

> 주의: `strtotime('2026-02-30')`처럼 존재하지 않는 날짜는 오버플로우됨

## 4. `DateTime` 클래스

객체지향 방식. 메서드 체이닝, 타임존 처리에 유리하며 불변 버전은 `DateTimeImmutable`.

```php
$dt = new DateTime('2026-06-03 12:00:00');
$dt = new DateTime('now', new DateTimeZone('Asia/Seoul'));

echo $dt->format('Y-m-d H:i:s');   // 포맷 출력
$dt->modify('+1 month');            // 날짜 수정 (원본 변경)
$dt->setDate(2026, 12, 25);         // 날짜 직접 설정
$dt->setTime(9, 30, 0);             // 시간 직접 설정

// Unix 타임스탬프로 생성
$dt = (new DateTime())->setTimestamp(1717372800);

// 타임스탬프 추출
$ts = $dt->getTimestamp();
```

## 5. `DateTimeImmutable` 클래스

`DateTime`과 동일하지만 수정 시 **새 객체를 반환**(원본 불변)

```php
$dt  = new DateTimeImmutable('2026-06-03');
$dt2 = $dt->modify('+1 year');  // $dt는 그대로, $dt2가 새 객체

echo $dt->format('Y');   // 2026
echo $dt2->format('Y');  // 2027
```

- 함수형 프로그래밍, 안전한 날짜 연산에 권장

## 6. `DateInterval`

날짜/시간 **간격**을 표현하는 클래스. `DateTime::diff()`의 결과이기도 함.

```php
// 1년 2개월 3일 간격 생성 (ISO 8601)
$interval = new DateInterval('P1Y2M3D');  // P=Period, T=Time 구분자
$interval = new DateInterval('PT3H30M'); // 3시간 30분

// 날짜에 간격 추가
$dt = new DateTime('2026-01-01');
$dt->add(new DateInterval('P1M'));   // 2026-02-01
$dt->sub(new DateInterval('P7D'));   // 7일 빼기

// 두 날짜 차이 계산
$d1 = new DateTime('2026-01-01');
$d2 = new DateTime('2026-06-03');
$diff = $d1->diff($d2);

echo $diff->days;    // 153 (총 일수)
echo $diff->m;       // 5 (월 차이 부분)
echo $diff->format('%R%a days'); // +153 days
```

## 7. `DatePeriod`

시작~종료 사이를 **반복 순회**할 때 사용

```php
$start    = new DateTime('2026-06-01');
$end      = new DateTime('2026-06-10');
$interval = new DateInterval('P1D'); // 1일 간격

$period = new DatePeriod($start, $interval, $end);

foreach ($period as $date) {
  echo $date->format('Y-m-d') . "\n";
}
// 2026-06-01 ~ 2026-06-09 (종료일 미포함)
```

## 8. `mktime()` / `checkdate()`

```php
// mktime(hour, min, sec, month, day, year)
$ts = mktime(0, 0, 0, 12, 25, 2026); // 2026-12-25 00:00:00 타임스탬프

// 날짜 유효성 검사
checkdate(2, 29, 2026);  // false (2026은 윤년 아님)
checkdate(2, 29, 2024);  // true  (2024는 윤년)
```

## 9. 타임존 처리

```php
// 전역 설정
date_default_timezone_set('Asia/Seoul');

// 객체별 설정
$tz = new DateTimeZone('America/New_York');
$dt = new DateTime('now', $tz);

// 타임존 변환
$dt->setTimezone(new DateTimeZone('Asia/Tokyo'));
echo $dt->format('Y-m-d H:i:s T');
```

## 10. 날짜 비교 방법

### 변환 흐름

```
문자열 "2026-06-03"
    │
    ▼ strtotime()       string → int
    │
Unix timestamp 1748822400
    │
    ▼ date()            int → string
    │
포맷 문자열 "2026-06-03"
```

### 비교 방식별 코드 패턴

#### `strtotime`: 정수 비교 (가장 단순)

```php
$a = strtotime('2026-01-01');
$b = strtotime('2026-06-03');

if ($a < $b) {
  // a가 더 이른 날짜
}
```

- 정수 비교라 빠름
- 타임존은 전역 설정에 의존하므로 주의

#### `DateTime`: 객체 연산자 비교

```php
$a = new DateTime('2026-01-01');
$b = new DateTime('2026-06-03');

if ($a < $b) { ... }   // 이전
if ($a == $b) { ... }  // 동일
if ($a > $b) { ... }   // 이후
```

- 객체 자체를 비교 연산자로 직접 사용 가능

#### `diff()`: 세부 차이 계산

```php
$a = new DateTimeImmutable('2026-01-01');
$b = new DateTimeImmutable('2026-06-03');
$diff = $a->diff($b);

echo $diff->days;   // 153 (총 일수)
echo $diff->m;      // 5   (월 차이 부분)
echo $diff->h;      // 0   (시간 차이 부분)
echo $diff->invert; // 0 = 양수, 1 = 음수 (a > b인 경우)
```

- `DateInterval` 객체를 반환하므로 일/월/시간 단위로 세부 접근 가능

### 날짜 연산과 불변성 차이

```
기준: 2026-06-03
│
├─ DateTime::modify('+1 month')
│      원본 객체 자체가 2026-07-03으로 변경 ← 부작용 있음
│
├─ DateTimeImmutable::modify('+1 month')
│      원본은 그대로 2026-06-03 유지
│      새 객체만 2026-07-03 반환       ← 안전
│
└─ strtotime('+1 month', $ts)
       int 반환, 원본 변수 $ts는 유지
       월말 오버플로우 주의 (예: 1월 31일 +1개월 → 3월 3일)
```

## 11. 함수 비교

| 항목 | `time()` | `date()` | `strtotime()` | `DateTime` | `DateTimeImmutable` |
| --- | --- | --- | --- | --- | --- |
| 반환 타입 | `int` | `string` | `int\|false` | 객체 | 객체 |
| 날짜 비교 | O (`<` `>`) | X | O (정수 비교) | O (연산자/diff) | O (연산자/diff) |
| 날짜 연산 | X | X | 제한적 | O | O |
| 타임존 지원 | X | 전역 설정 | 전역 설정 | O (객체별) | O (객체별) |
| 불변성 | - | - | - | X (원본 변경) | O (새 객체 반환) |
| 파싱 유연성 | - | - | 높음 | 보통 | 보통 |
| 권장 용도 | 타임스탬프 취득 | 빠른 포맷 | 문자열 파싱, 단순 연산 | 일반 연산 | **안전한 날짜 연산** |

## 12. 실용 패턴

```php
// 이번 달 1일 ~ 말일
$first = new DateTimeImmutable('first day of this month');
$last  = new DateTimeImmutable('last day of this month');

// N일 전/후
$future = (new DateTimeImmutable())->modify('+30 days');

// 두 날짜 비교
$d1 = new DateTimeImmutable('2026-01-01');
$d2 = new DateTimeImmutable('2026-06-03');
if ($d2 > $d1) { /* $d2가 더 나중 */ }

// 날짜 차이 (일수)
$diff = $d1->diff($d2)->days; // 153

// 문자열 → DateTime 안전하게
$dt = DateTimeImmutable::createFromFormat('d/m/Y', '03/06/2026');
if ($dt === false) { /* 파싱 실패 처리 */ }
```
