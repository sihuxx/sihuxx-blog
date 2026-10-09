# SQL 함수와 문법

## 01 함수

### 1. 집계 함수

데이터를 묶어서 계산할 때 사용, `GROUP BY`와 함께 자주 쓰임

- `COUNT(*)`: 전체 행의 개수 (NULL 포함)
- `COUNT(컬럼)`: 해당 컬럼에 값이 있는 행의 개수 (NULL 제외)
- `SUM(컬럼)`: 총합
- `AVG(컬럼)`: 평균
- `MAX(컬럼)`: 최댓값
- `MIN(컬럼)`: 최솟값
- `GROUP_CONCAT(컬럼)`: 여러 행의 문자열을 쉼표(,)로 연결해 한 줄로 만듦

### 2. 문자열 함수

- `CONCAT(A, B)`: A와 B를 이어 붙임
- `SUBSTRING(문자, 시작, 개수)`: 시작 위치부터 개수만큼 자름 (= `SUBSTR`)
- `LEFT(문자, N)`: 왼쪽부터 N글자
- `RIGHT(문자, N)`: 오른쪽부터 N글자
- `LENGTH(문자)`: 바이트 수 (한글은 3바이트)
- `CHAR_LENGTH(문자)`: 글자 수 (한글도 1글자)
- `UPPER(문자)` / `LOWER(문자)`: 대문자 / 소문자 변환
- `TRIM(문자)`: 양쪽 공백 제거
- `REPLACE(문자, A, B)`: 문자열 안의 A를 B로 변경

### 3. 날짜/시간 함수

대여 시스템에서 필수

- `NOW()`: 현재 날짜와 시간 (연월일 시분초)
- `CURDATE()`: 현재 날짜 (연월일)
- `YEAR(날짜)` / `MONTH(날짜)` / `DAY(날짜)`: 연, 월, 일 추출
- `DATEDIFF(날짜1, 날짜2)`: 날짜1에서 날짜2를 뺀 일수 차이
- `DATE_ADD(날짜, INTERVAL 1 DAY)`: 날짜에 1일 더함 (MONTH, YEAR 가능)
- `DATE_SUB(날짜, INTERVAL 1 DAY)`: 날짜에서 1일 뺌
- `DATE_FORMAT(날짜, '형식')`: 원하는 형식으로 출력 (예: `%Y-%m-%d`)

### 4. 숫자/수학 함수

- `ABS(숫자)`: 절댓값
- `ROUND(숫자, 자릿수)`: 반올림
- `CEIL(숫자)`: 올림
- `FLOOR(숫자)`: 내림
- `MOD(A, B)`: A를 B로 나눈 나머지
- `RAND()`: 0~1 사이의 난수

### 5. 논리 및 NULL 처리

- `IF(조건, 참일때, 거짓일때)`: 조건에 따라 다른 값을 출력
- `IFNULL(컬럼, 대체값)`: 컬럼 값이 NULL이면 대체값 출력
- `COALESCE(A, B, C...)`: NULL이 아닌 첫 번째 값을 반환
- `CASE WHEN 조건 THEN 값 ELSE 값 END`: 여러 조건을 겹쳐 쓰는 조건문

## 02 문법

### 1. JOIN (테이블 합치기)

흩어진 데이터를 **공통된 컬럼(Key)** 기준으로 연결해 하나의 테이블처럼 조회

#### INNER JOIN (교집합)

- 두 테이블 **모두에 데이터가 있는 경우**만 결합
- 가장 많이 사용
- 예: 책을 대여한 기록이 있는 유저만 조회 (대여 안 한 유저는 제외)

```sql
SELECT u.name, b.book_title
FROM user u
INNER JOIN rental r
  ON u.idx = r.user_idx; -- 연결 고리 (유저번호가 같으면 결합)
```

#### LEFT JOIN (왼쪽 기준 전체)

- **왼쪽 테이블은 전부 출력**, 오른쪽은 매칭되면 출력하고 없으면 `NULL`
- 예: 모든 유저를 조회하되, 대여 기록이 없으면 빈 값으로 표시

```sql
SELECT u.name, r.rental_date
FROM user u
LEFT JOIN rental r
  ON u.idx = r.user_idx;
-- 결과: 대여 기록 없는 유저는 rental_date가 NULL
```

#### JOIN 비교

| 종류 | 설명 | 결과 |
| --- | --- | --- |
| **INNER JOIN** | 교집합 (A ∩ B) | 양쪽 모두 있는 데이터만 출력 |
| **LEFT JOIN** | 왼쪽 기준 (A) | 왼쪽은 전부, 오른쪽은 없으면 NULL |
| **RIGHT JOIN** | 오른쪽 기준 (B) | 오른쪽은 전부, 왼쪽은 없으면 NULL (잘 안 씀) |

### 2. GROUP BY (데이터 묶기)

특정 컬럼 기준으로 **그룹화**해 합계, 평균, 개수 등을 구할 때 사용 (집계 함수는 위 `01 함수` 참고)

```sql
SELECT 그룹컬럼, 집계함수(계산할컬럼)
FROM 테이블명
GROUP BY 그룹컬럼;
```

예: 유저별 대여 횟수

```sql
SELECT
    user_idx,
    COUNT(*) as rental_count -- 2. 개수를 센다
FROM rental
GROUP BY user_idx;           -- 1. 유저 번호끼리 묶는다
```

### 3. HAVING (그룹화 후 필터링)

- `WHERE`: 묶기 **전**에 거름
- `HAVING`: 묶은 **결과(집계값)** 를 거름
- 예: 대여 횟수가 5회 이상인 유저만 조회

```sql
SELECT user_idx, COUNT(*) as cnt
FROM rental
GROUP BY user_idx
HAVING cnt >= 5; -- (O) 그룹화된 결과인 cnt를 조건으로 사용
-- WHERE cnt >= 5 -- (X) 에러. WHERE는 집계함수 사용 불가
```

### 4. 실전 응용 (JOIN + GROUP BY)

유저별로 빌린 책의 총 권수와 책 제목들을 한 줄로 조회

```sql
SELECT
    u.name,                            -- 유저 이름
    COUNT(r.idx) as total_books,       -- 빌린 횟수
    GROUP_CONCAT(b.title) as book_list -- 빌린 책 제목들 (콤마로 연결)
FROM user u
INNER JOIN rental r ON u.idx = r.user_idx   -- 1. 유저 + 대여 기록 연결
INNER JOIN book b   ON r.book_idx = b.idx   -- 2. 대여 기록 + 책 정보 연결
GROUP BY u.idx;                             -- 3. 유저별로 묶기
```

### 5. 작성 순서와 실행 순서

코드를 **작성하는 순서**와 DB가 **실행하는 순서**가 다름. 이를 알면 에러 원인 파악이 쉬움.

| 작성 순서 | 실행 순서 | 의미 |
| --- | --- | --- |
| 1. `SELECT` | 6. `SELECT` | 어떤 컬럼을 보여줄지 |
| 2. `FROM` | 1. `FROM` | 어느 테이블에서 |
| 3. `JOIN` | 2. `JOIN` | 무엇과 합칠지 |
| 4. `WHERE` | 3. `WHERE` | **(그룹 전)** 누구를 제외할지 |
| 5. `GROUP BY` | 4. `GROUP BY` | 어떻게 묶을지 |
| 6. `HAVING` | 5. `HAVING` | **(그룹 후)** 누구를 제외할지 |
| 7. `ORDER BY` | 7. `ORDER BY` | 정렬 순서 |
| 8. `LIMIT` | 8. `LIMIT` | 몇 개만 출력할지 |
