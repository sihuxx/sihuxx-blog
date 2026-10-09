# CRUD (데이터 기본 동작)

데이터베이스가 수행하는 **4가지 기본 작업**

- **C**reate (생성): `INSERT` - 데이터를 새로 넣음
- **R**ead (조회): `SELECT` - 데이터를 가져와서 봄
- **U**pdate (수정): `UPDATE` - 기존 데이터를 고침
- **D**elete (삭제): `DELETE` - 데이터를 지움

## 핵심 구문

### `INTO`

- 용도: `INSERT` 문
- 의미: 데이터를 **어느 테이블에** 넣을지 지정
- 예: `INSERT INTO user ...` (user 테이블에 삽입)

### `FROM`

- 용도: `SELECT`, `DELETE` 문
- 의미: 데이터를 **어느 테이블에서** 가져오거나 지울지 지정
- 예: `SELECT * FROM post` (post 테이블에서 조회)

### `WHERE`

- 용도: `SELECT`, `UPDATE`, `DELETE` 문
- 의미: **어떤 조건**에 맞는 데이터만 처리할지 필터링 (가장 중요)
- 예: `WHERE id = 'sihu'` (아이디가 sihu인 데이터만)

### `ORDER BY`

- 용도: `SELECT` 문
- 의미: 가져온 데이터를 **어떤 순서로** 정렬할지 지정
- 예: `ORDER BY date DESC` (날짜 최신순 정렬)

## 예시 코드

```sql
-- 1. Create (INTO 사용)
INSERT INTO user (id, name) VALUES ('sihu', '시후');

-- 2. Read (FROM, WHERE, ORDER BY 사용)
SELECT * FROM post WHERE writer = 'sihu' ORDER BY date DESC;

-- 3. Update (WHERE 사용)
UPDATE post SET title = '수정함' WHERE idx = 3;

-- 4. Delete (FROM, WHERE 사용)
DELETE FROM post WHERE idx = 3;
```
