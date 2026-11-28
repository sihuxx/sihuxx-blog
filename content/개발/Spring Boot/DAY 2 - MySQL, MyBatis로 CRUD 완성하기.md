## 1. 프로젝트 개요

- 질문을 등록·조회·수정·삭제하는 CRUD REST API 서버
- 데이터는 MySQL(`school_ai` DB의 `questions` 테이블)에 저장 → 서버를 재시작해도 데이터 유지
- DB 접근은 MyBatis 어노테이션 Mapper로 처리
- 답변은 고정 문구("AI 답변 준비중입니다.")
- `static/index.html`, `app.js`로 브라우저 화면에서 직접 등록·수정·삭제 가능

|항목|내용|
|---|---|
|언어|Java 25|
|프레임워크|Spring Boot 4.2.0|
|빌드 도구|Gradle|
|의존성|`spring-boot-starter-webmvc`, `spring-boot-starter-validation`, `mybatis-spring-boot-starter`, `mysql-connector-j`|
|DB|MySQL (`jdbc:mysql://127.0.0.1:3306/school_ai`)|
|포트|8081|

## 2. 폴더 구성

```
school-spring-ai-study/
├── build.gradle
├── settings.gradle
├── README.md / DB-연결.md / 검증.md
├── 퍼블리싱/                      # 일차별 HTML·JS 교안 자료
└── src/main/
    ├── java/com/example/rag/
    │   ├── RagApplication.java
    │   ├── controller/QuestionController.java
    │   ├── service/QuestionService.java
    │   ├── repository/QuestionMapper.java
    │   ├── model/Question.java
    │   ├── dto/QuestionRequest.java
    │   ├── exception/ApiErrorHandler.java
    │   └── practice/ObjectPractice.java
    └── resources/
        ├── application.yaml
        └── static/
            ├── index.html
            └── app.js
```

|패키지|역할|
|---|---|
|`controller`|HTTP 요청 수신, 응답 반환|
|`service`|비즈니스 로직, 트랜잭션 처리|
|`repository`|DB에 SQL 실행 (MyBatis Mapper)|
|`model`|DB 테이블과 대응되는 데이터 클래스|
|`dto`|요청 body 형식과 검증 규칙|
|`exception`|예외 → 에러 응답 변환|
|`practice`|객체 생성 연습용 코드|

## 3. 요청 흐름

```
브라우저(app.js) → QuestionController → QuestionService → QuestionMapper → MySQL
                         │                    │
                         │                    └─ 없는 id → ResponseStatusException(404)
                         └─ 검증 실패 → MethodArgumentNotValidException(400)
                                              ↓
                                       ApiErrorHandler → {"message": "..."} 응답
```

1. 브라우저가 `/api/questions`로 요청 전송
2. `QuestionController`가 요청 수신, `@Valid`로 입력값 검사
3. `QuestionService`가 질문 객체 생성, 트랜잭션 안에서 Mapper 호출
4. `QuestionMapper`가 SQL 실행해 MySQL에 저장·조회·수정·삭제
5. 결과를 JSON으로 응답
6. 검증 실패(400), 없는 질문(404), DB 오류(503)는 `ApiErrorHandler`가 에러 메시지로 응답

## 4. API

|메서드|주소|설명|응답|
|---|---|---|---|
|POST|`/api/questions`|질문 등록|201 + 등록된 질문|
|GET|`/api/questions`|질문 전체 조회|200 + 질문 목록|
|PUT|`/api/questions/{id}`|질문 수정|200 + 수정된 질문|
|DELETE|`/api/questions/{id}`|질문 삭제|204 (body 없음)|

요청 예시 (POST, PUT 공통)

```json
{ "content": "도서관은 언제 여나요?" }
```

응답 예시

```json
{ "id": 1, "content": "도서관은 언제 여나요?", "answer": "AI 답변 준비중입니다." }
```

## 5. Java 파일 설명

### RagApplication.java

- **역할**: 스프링 부트 애플리케이션 실행 시작점

```java
package com.example.rag; // 루트 패키지. controller, service 등 하위 패키지까지 스캔 대상

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// @SpringBootApplication = 아래 3개를 합친 어노테이션
// - @Configuration: 이 클래스를 스프링 설정 클래스로 등록
// - @EnableAutoConfiguration: build.gradle 의존성을 보고 필요한 설정 자동 적용
//   (내장 톰캣, JSON 변환기, MySQL DataSource, MyBatis 설정 등)
// - @ComponentScan: 하위 패키지의 @RestController, @Service, @Mapper 등을 찾아 빈으로 등록
@SpringBootApplication
public class RagApplication {

    // 자바 프로그램 실행 시 가장 먼저 호출되는 메서드
    public static void main(String[] args) {
        // 스프링 컨테이너 생성 → DB 연결 설정 → 빈 등록·주입 → 내장 서버 실행(8081 포트)
        SpringApplication.run(RagApplication.class, args);
    }

}
```

### QuestionController.java

- **역할**: HTTP 요청을 받아 서비스에 전달하고 결과를 응답으로 반환

```java
package com.example.rag.controller;

import com.example.rag.dto.QuestionRequest;
import com.example.rag.model.Question;
import com.example.rag.service.QuestionService;

import java.util.List;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

// @RestController = @Controller + @ResponseBody
// 메서드 반환값을 HTML 화면이 아닌 JSON 데이터로 변환해 응답
@RestController
// 이 클래스의 모든 메서드 주소 앞에 /api/questions 공통으로 붙음
@RequestMapping("/api/questions")
public class QuestionController {

    // 컨트롤러가 사용할 서비스 객체 (final → 생성 후 교체 불가)
    private final QuestionService service;

    // 생성자 주입(DI): 스프링이 만들어 둔 QuestionService 빈을 넣어 줌. 직접 new 하지 않음
    public QuestionController(QuestionService service) { this.service = service; }

    // POST /api/questions → 질문 등록
    @PostMapping
    // 성공 시 기본 200 대신 201 Created 응답
    @ResponseStatus(HttpStatus.CREATED)
    // @RequestBody: 요청 JSON {"content": "..."} → QuestionRequest 객체로 변환
    // @Valid: QuestionRequest의 @NotBlank, @Size 검사. 실패 시 이 메서드 실행 없이 400 응답
    public Question create(@Valid @RequestBody QuestionRequest request) {
        // 질문 내용만 꺼내 서비스에 전달, 저장된 Question을 JSON으로 응답
        return service.create(request.content());
    }

    // GET /api/questions → 질문 전체 조회
    @GetMapping
    // List 반환 → JSON 배열 [...]로 변환
    public List<Question> findAll() { return service.findAll(); }

    // PUT → 질문 수정
    // 주의: 주소에 "/{id}"가 빠져 있음 → @PathVariable id를 받을 수 없음
    //       app.js는 PUT /api/questions/{id}로 요청하므로 @PutMapping("/{id}")로 수정 필요
    @PutMapping
    // @PathVariable: 주소의 {id} 부분을 long id로 받음
    public Question update(@PathVariable long id, @Valid @RequestBody QuestionRequest request) {
        return service.update(id, request.content());
    }

    // DELETE /api/questions/{id} → 질문 삭제
    @DeleteMapping("/{id}")
    // 성공 시 204 No Content (응답 body 없음)
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable long id) {
        service.delete(id);
    }
}
```

### QuestionService.java

- **역할**: 질문 생성·조회·수정·삭제 로직 처리, 트랜잭션 관리

```java
package com.example.rag.service;

import com.example.rag.model.Question;
import com.example.rag.repository.QuestionMapper;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ResponseStatusException;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.TransactionTemplate;

import java.util.List;

// 스프링 빈으로 등록 → 컨트롤러에 주입
// 데이터는 DB에 저장하므로 메모리 Map과 synchronized 불필요
@Service
public class QuestionService {

    // DB에 SQL을 실행하는 MyBatis Mapper
    private final QuestionMapper mapper;
    // 코드로 트랜잭션 범위를 직접 지정하는 도구
    private final TransactionTemplate transaction;

    // 생성자 주입: Mapper와 TransactionTemplate 모두 스프링이 넣어 줌
    public QuestionService(QuestionMapper mapper, TransactionTemplate transaction) {
        this.mapper = mapper;
        this.transaction = transaction;
    }

    // 질문 등록
    public Question create(String content) {
        // id는 DB가 AUTO_INCREMENT로 부여하므로 임시로 0
        // strip(): 앞뒤 공백 제거
        Question question = makeQuestion(0, content.strip());
        // transaction.execute(): 안의 작업을 하나의 트랜잭션으로 실행
        // 중간에 예외가 나면 전부 취소(롤백), 정상 종료되면 확정(커밋)
        return transaction.execute(status -> {
            // INSERT 실행. 실행 후 DB가 만든 id가 question.id에 자동으로 채워짐
            mapper.insert(question);
            return question;
        });
    }

    // 질문 전체 조회
    // readOnly = true: 읽기 전용 트랜잭션 → 데이터 변경 없음을 명시, DB 최적화 가능
    @Transactional(readOnly = true)
    public List<Question> findAll() {
        return mapper.findAll();
    }

    // 질문 수정
    public Question update(long id, String content) {
        // 해당 id 질문이 없으면 404 예외
        if(mapper.findById(id) == null) {
            throw notFound();
        }
        // 수정할 내용으로 새 질문 객체 생성
        Question updated = makeQuestion(id, content.strip());
        return transaction.execute(status -> {
            // UPDATE 결과가 0행이면 그사이 삭제된 것 → 404 예외 + 롤백
            if(mapper.update(updated) == 0) {
                throw notFound();
            }
            return updated;
        });
    }

    // 질문 삭제
    public void delete(long id) {
        // DELETE 결과가 0행이면 없는 질문 → 404 예외
        if(mapper.delete(id) == 0) throw notFound();
    }

    // 고정 답변을 넣어 질문 객체 생성 (클래스 내부 전용)
    private Question makeQuestion(long id, String content) {
        return new Question(id, content, "AI 답변 준비중입니다.");
    }

    // 404 Not Found 예외 생성 → ApiErrorHandler.status()가 처리
    private ResponseStatusException notFound() {
        return new ResponseStatusException(HttpStatus.NOT_FOUND, "질문을 찾을 수 없습니다.");
    }
}
```

### QuestionMapper.java

- **역할**: MyBatis로 `questions` 테이블에 SQL을 실행하는 인터페이스

```java
package com.example.rag.repository;

import com.example.rag.model.Question;
import org.apache.ibatis.annotations.*;

import java.util.List;

// @Mapper: MyBatis가 이 인터페이스의 구현체를 자동 생성해 스프링 빈으로 등록
// 구현 클래스를 직접 만들 필요 없이 어노테이션의 SQL만 작성
@Mapper
public interface QuestionMapper {

    // INSERT: 새 질문 저장
    // #{content}: Question 객체의 content 값을 안전하게 바인딩 (SQL 인젝션 방지)
    @Insert("INSERT INTO questions (content, answer) VALUES (#{content}, #{answer})")
    // useGeneratedKeys = true: DB가 AUTO_INCREMENT로 만든 id를 받아옴
    // keyProperty = "id": 받아온 id를 Question 객체의 id 필드에 세팅
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insert(Question question); // 반환값: 영향받은 행 수

    // SELECT: 전체 질문을 id 순서로 조회
    // 조회 결과의 컬럼(id, content, answer)이 Question 객체의 같은 이름 필드에 자동 매핑
    @Select("SELECT id, content, answer FROM questions ORDER BY id")
    List<Question> findAll();

    // SELECT: id로 질문 하나 조회. 없으면 null 반환
    // @Param("id"): 매개변수를 SQL의 #{id} 이름과 연결
    @Select("SELECT id, content, answer FROM questions WHERE id = #{id}")
    Question findById(@Param("id") long id);

    // UPDATE: id에 해당하는 질문 내용·답변 수정
    @Update("UPDATE questions SET content = #{content}, answer = #{answer} WHERE id = #{id}")
    int update(Question question); // 반환값 0 → 해당 id 없음

    // DELETE: id에 해당하는 질문 삭제
    @Delete("DELETE FROM questions WHERE id = #{id}")
    int delete(@Param("id") long id); // 반환값 0 → 해당 id 없음

    // SQL 직접 작성(MyBatis): 보수적인 은행권 등에서 많이 사용
    // JPA(SQL 자동 생성): 대기업 신규 서비스, 스타트업에서 많이 사용
}
```

### Question.java

- **역할**: `questions` 테이블 한 행(id, 질문 내용, 답변)에 대응되는 데이터 클래스

```java
package com.example.rag.model;

public class Question {
    // MyBatis가 값을 채워 넣어야 하므로 final 없음 (1일차와 차이)
    private long id;         // 질문 번호 (DB의 id 컬럼)
    private String content;  // 질문 내용 (content 컬럼)
    private String answer;   // 답변 내용 (answer 컬럼)

    // 기본 생성자: MyBatis가 조회 결과로 객체를 만들 때 먼저 빈 객체 생성 후 setter로 값 세팅
    public Question() {

    }

    // 전체 값을 한 번에 세팅하는 생성자 (서비스에서 사용)
    public Question(long id, String content, String answer) {
        this.id = id;
        this.content = content;
        this.answer = answer;
    }

    // getter: JSON 변환과 MyBatis #{...} 바인딩에 사용
    // getId() → "id", getContent() → "content", getAnswer() → "answer"
    public long getId() { return id; }
    public String getContent() { return content; }
    public String getAnswer() { return answer; }

    // setter: MyBatis 조회 결과 매핑, INSERT 후 생성된 id 세팅에 사용
    public void setId(long id) { this.id = id; }
    public void setContent(String content) { this.content = content; }
    public void setAnswer(String answer) { this.answer = answer; }
}
```

### QuestionRequest.java

- **역할**: 질문 등록·수정 요청 body 형식과 검증 규칙을 정의하는 DTO

```java
package com.example.rag.dto;

// spring-boot-starter-validation 의존성으로 사용 가능한 검증 어노테이션
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

// record: 필드(private final), 생성자, getter(content()), equals(), toString() 자동 생성
// 요청 JSON의 "content" 키가 같은 이름의 content 필드에 들어감 (Jackson 역직렬화)
public record QuestionRequest(
        // 아래 규칙은 Controller 매개변수에 @Valid가 있을 때 검사
        // null, "", "   " 모두 거절
        @NotBlank(message = "질문을 입력해 주세요.")
        // 200자 초과 거절 (DB 컬럼 VARCHAR(200)과 길이 일치)
        @Size(max = 200, message = "질문은 200자 이내로 입력해 주세요.")
        String content) {
}
```

### ApiErrorHandler.java

- **역할**: 컨트롤러에서 발생한 예외를 한곳에서 받아 `{"message": "..."}` 형태로 응답

```java
package com.example.rag.exception;

import java.util.Map;

import org.springframework.dao.DataAccessException;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.server.ResponseStatusException;

// 모든 Controller에서 발생한 예외를 가로채는 전역 예외 처리 클래스
// 반환값은 Controller처럼 JSON으로 응답
@RestControllerAdvice
public class ApiErrorHandler {

    // ResponseStatusException 처리 (서비스의 notFound() → 404)
    @ExceptionHandler(ResponseStatusException.class)
    public ResponseEntity<Map<String, String>> status(ResponseStatusException error) {
        // getReason(): 예외에 담긴 이유 문구. 없으면 기본 문구 사용
        String message = error.getReason() == null ? "요청을 처리하지 못했습니다." : error.getReason();
        // getStatusCode(): 예외에 담긴 상태 코드(404 등)를 그대로 응답 코드로 사용
        return ResponseEntity.status(error.getStatusCode()).body(Map.of("message", message));
    }

    // @Valid 검사 실패 시 스프링이 던지는 예외 처리 → 400
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> validation(MethodArgumentNotValidException error) {
        String message = error.getBindingResult()   // 검증 결과
                .getFieldErrors()                    // 실패한 필드 에러 목록
                .stream()
                .map(e -> e.getDefaultMessage())     // 각 에러의 message 값만 추출
                .findFirst()                         // 첫 번째 메시지만 사용
                .orElse("입력값을 확인해 주세요.");     // 메시지가 없을 때 기본 문구
        // badRequest(): 400 Bad Request
        return ResponseEntity.badRequest().body(Map.of("message", message));
    }

    // DB 작업 중 발생한 예외 처리 (DB 서버 꺼짐, SQL 오류 등) → 503
    // DataAccessException: 스프링이 DB 관련 예외를 한 종류로 묶어 주는 상위 예외
    @ExceptionHandler(DataAccessException.class)
    public ResponseEntity<Map<String, String>> database(DataAccessException error) {
        // 503 Service Unavailable: 서버가 일시적으로 요청을 처리할 수 없는 상태
        // 상세 오류 대신 간단한 문구만 응답 → 내부 DB 정보 노출 방지
        return ResponseEntity.status(503).body(Map.of("message", "DB 작업 실패"));
    }
}
```

### ObjectPractice.java

- **역할**: `Question` 객체 생성과 getter 사용 연습용 코드 (서버와 별개로 실행)

```java
package com.example.rag.practice;

import com.example.rag.model.Question;

public class ObjectPractice {
    // 스프링과 상관없이 단독 실행하는 main 메서드
    public static void main(String[] args) {
        // new로 Question 객체 2개 생성 (1L, 2L: long 타입 숫자 표기)
        Question first = new Question(1L, "도서관은 언제 열어요?", "안내문을 준비 중입니다.");
        Question second = new Question(2L, "동아리는 언제 신청해요?", "안내문을 준비 중입니다.");

        // getter로 값을 꺼내 콘솔에 출력
        // 같은 클래스로 만들어도 객체마다 각자 다른 값을 가짐
        System.out.println(first.getId() + " " + first.getContent() + " " + first.getAnswer());
        System.out.println(second.getId() + " " + second.getContent() + " " + second.getAnswer());
    }
}
```
## 6. 개발 지식

### 트랜잭션

- 이름 그대로 번역하면 거래.
- 데이터베이스에서 트랜잭션은 하나의 거래를 안전하게 처리하도록 보장해주는 것을 뜻함.
- 하나의 트랜잭션이 모두 성공해서 데이터베이스에 반영되는 것을 커밋이라하고, 작업 중 하나라도 실패해서 거래 이전으로 되돌리는 것을 롤백이라고 한다.
- 트랜잭션 원칙
	- **원자성**: 트랜잭션 내에서 실행한 작업들은 **마치 하나의 작업인 것처럼 모두 성공하거나 실패**해야 함
	- **일관성**: 모든 트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야 함. 
	- **격리성**: 동시에 실행되는 트랜잭션들이 서로에게 영향을 미치지 못하도록 격리한다. **즉, 동시에 같은 데이터를 수정하지 못하도록 해야함**
	- **지속성**: 트랜잭션을 성공적으로 끝내면 그 결과가 항상 기록되어야 한다. **중간에 시스템에 문제가 발생해도 데이터베이스 로그 등을 사용하여 성공한 트랜잭션 내용을 복구할 수 있어야 함**

### 랭체인
RAG 시스템을 구성하는 다양한 컴포넌트를 쉽게 연결하고 관리할 수 있도록 돕는 파이프라인 프레임워크

### 랭그래프
LLM을 사용하여 상태를 유지하고 여러 에이전트를 포함한 애플리케이션을 쉽게 만들 수 있도록 도와주는 프레임워크

### 그 외

**DB 연결법** (cmd)
1. mysql -u root -p
2. create database school_ai;
3. use school_ai;
4. CREATE TABLE IF NOT EXISTS questions (
    id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    content VARCHAR(200) NOT NULL,
    answer TEXT NOT NULL
  ) DEFAULT CHARSET=utf8mb4
5. show tables; :: 현재 테이블 조회 명령어
	desc questions (DB 이름) :: 테이블 컬럼 조회 명령어