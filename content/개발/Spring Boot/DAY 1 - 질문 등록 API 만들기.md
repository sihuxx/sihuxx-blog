## 1. 프로젝트 개요

- 질문을 등록하고 목록을 조회하는 REST API 서버
- 데이터는 DB 없이 메모리(TreeMap)에 저장
- 답변은 고정 문구("질문을 받았습니다. 답변은 준비 중입니다.")

|항목|내용|
|---|---|
|언어|Java 25|
|프레임워크|Spring Boot 4.2.0|
|빌드 도구|Gradle|
|의존성|`spring-boot-starter-webmvc`, `spring-boot-starter-validation`|
|포트|8081|

## 2. 폴더 구성

```
rag/
├── build.gradle
├── settings.gradle
└── src/
    ├── main/
    │   ├── java/com/example/rag/
    │   │   ├── RagApplication.java
    │   │   ├── QuestionController.java
    │   │   ├── QuestionService.java
    │   │   ├── Question.java
    │   │   ├── QuestionRequest.java
    │   │   └── ApiErrorHandler.java
    │   └── resources/
    │       └── application.yaml
    └── test/java/com/example/rag/
        └── RagApplicationTests.java
```

## 3. 요청 흐름

```
클라이언트 → QuestionController → QuestionService → 응답(JSON)
                   │
                   └─ 검증 실패 시 → ApiErrorHandler → 400 응답
```

1. 클라이언트가 `/api/questions`로 요청 전송
2. `QuestionController`가 요청 수신, `@Valid`로 입력값 검사
3. 검사 통과 시 `QuestionService`가 질문 저장 또는 목록 조회
4. 결과를 JSON으로 응답
5. 검사 실패 시 `ApiErrorHandler`가 에러 메시지 응답

## 4. API

|메서드|주소|설명|응답|
|---|---|---|---|
|POST|`/api/questions`|질문 등록|201 + 등록된 질문|
|GET|`/api/questions`|질문 전체 조회|200 + 질문 목록|

요청 예시

```json
{ "content": "도서관은 언제 여나요?" }
```

응답 예시

```json
{ "id": 1, "content": "도서관은 언제 여나요?", "answer": "질문을 받았습니다. 답변은 준비 중입니다." }
```

## 5. Java 파일 설명

### RagApplication.java

- **역할**: 스프링 부트 애플리케이션 실행 시작점

```java
package com.example.rag; // 이 패키지와 하위 패키지가 스프링의 스캔 대상

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// @SpringBootApplication = 아래 3개를 합친 어노테이션
// - @Configuration: 이 클래스를 스프링 설정 클래스로 등록
// - @EnableAutoConfiguration: build.gradle 의존성을 보고 필요한 설정 자동 적용 (내장 톰캣, JSON 변환기 등)
// - @ComponentScan: 같은 패키지의 @RestController, @Service 등을 찾아 빈으로 등록
@SpringBootApplication
public class RagApplication {

    // 자바 프로그램 실행 시 가장 먼저 호출되는 메서드
    public static void main(String[] args) {
        // 스프링 컨테이너 생성 → 빈 등록·주입 → 내장 서버 실행(application.yaml의 8081 포트)
        SpringApplication.run(RagApplication.class, args);
    }

}
```

### QuestionController.java

- **역할**: HTTP 요청을 받아 서비스에 전달하고 결과를 응답으로 반환

```java
package com.example.rag;

import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

import java.util.List;

// @RestController = @Controller + @ResponseBody
// 메서드 반환값을 HTML 화면이 아닌 JSON 데이터로 변환해 응답
@RestController
// 이 클래스의 모든 메서드 주소 앞에 /api/questions 공통으로 붙음
@RequestMapping("/api/questions")
public class QuestionController {

    // 컨트롤러가 사용할 서비스 객체
    // final: 생성 후 다른 객체로 교체 불가
    private final QuestionService service;

    // 생성자 주입(DI): 스프링이 미리 만들어 둔 QuestionService 빈을 넣어 줌
    // 생성자가 하나뿐이라 @Autowired 생략 가능
    public QuestionController(QuestionService service) {
        this.service = service;
    }

    // POST /api/questions → 질문 등록
    @PostMapping
    // 성공 시 기본 200 대신 201 Created(새 데이터 생성됨) 응답
    @ResponseStatus(HttpStatus.CREATED)
    // @RequestBody: 요청 body JSON {"content": "..."} → QuestionRequest 객체로 변환
    // @Valid: QuestionRequest의 @NotBlank, @Size 검사 실행
    //         실패 시 이 메서드는 실행되지 않고 MethodArgumentNotValidException 발생 → ApiErrorHandler로 이동
    public Question create(@Valid @RequestBody QuestionRequest request) {
        // request.content(): record의 자동 생성 getter로 질문 내용만 꺼내 서비스에 전달
        // 서비스가 만든 Question을 반환 → 스프링이 JSON으로 변환해 응답
        return service.create(request.content());
    }

    // GET /api/questions → 질문 전체 조회
    @GetMapping
    public List<Question> findAll() {
        // List<Question> 반환 → JSON 배열로 변환되어 200 응답
        return service.findAll();
    }
}
```

### QuestionService.java

- **역할**: 질문 생성·저장·조회 로직 처리 (현재는 메모리 저장소 역할도 함께 수행)

```java
package com.example.rag;

import org.springframework.stereotype.Service;

import java.util.List;
import java.util.TreeMap;

// 스프링 빈으로 등록 → 컨트롤러 생성자에 자동 주입
@Service
public class QuestionService {

    // 질문 저장소 (DB 대신 메모리 사용 → 서버 재시작 시 초기화)
    // key: 질문 id, value: 질문 객체
    // TreeMap: key 기준 자동 정렬 → 조회 시 항상 id 오름차순
    // 저장(put) 시간 복잡도 O(log n)
    private final TreeMap<Long, Question> questions = new TreeMap<>();

    // 다음 질문에 부여할 id 번호 (1부터 시작)
    private long nextId = 1;

    // synchronized: 한 번에 하나의 요청만 이 메서드 실행 가능
    // 없으면 두 요청이 동시에 nextId = 1을 읽고 둘 다 id 1로 저장 → 하나가 덮어써지는 동시성 문제 발생
    public synchronized Question create(String content) {
        // strip(): 질문 앞뒤 공백 제거 ("  안녕 " → "안녕")
        // 현재 nextId로 질문 객체 생성
        Question question = makeQuestion(nextId, content.strip());
        // 저장소에 추가
        questions.put(nextId, question);
        // 다음 질문을 위해 id 1 증가
        nextId++;
        // 생성한 질문 반환 → 컨트롤러가 응답으로 사용
        return question;
    }

    // 조회에도 synchronized → 저장 도중의 불완전한 상태 조회 방지
    public synchronized List<Question> findAll() {
        // values(): 저장된 질문 객체만 꺼냄 (id 순서)
        // List.copyOf(): 수정 불가능한 복사본 반환 → 외부에서 원본 저장소 변경 불가
        return List.copyOf(questions.values());
    }

    // 클래스 내부에서만 쓰는 질문 생성 메서드
    // 지금은 고정 답변 문구를 넣어 생성
    private Question makeQuestion(long id, String content) {
        return new Question(id, content, "질문을 받았습니다. 답변은 준비 중입니다.");
    }
}
```

### Question.java

- **역할**: 질문 하나의 데이터(id, 질문 내용, 답변)를 담는 클래스. 응답 JSON 형태 결정

```java
package com.example.rag;

public class Question {
    // private: 클래스 밖에서 필드 직접 접근 불가
    // final: 생성자에서 한 번 값을 넣으면 이후 변경 불가 → 불변 객체
    private final long id;         // 질문 번호
    private final String content;  // 질문 내용
    private final String answer;   // 답변 내용

    // 생성자: new Question(...)으로 객체를 만들 때 세 필드 값 세팅
    // this.id는 필드, id는 매개변수
    public Question(long id, String content, String answer) {
        this.id = id;
        this.content = content;
        this.answer = answer;
    }

    // getter: 필드 값을 꺼내는 메서드 (setter는 없음 → 값 변경 불가)
    // 스프링(Jackson)이 getter 이름 기준으로 JSON 변환
    // getId() → "id", getContent() → "content", getAnswer() → "answer"
    // getter가 없으면 해당 값은 JSON에 포함되지 않음
    public long getId() {
        return id;
    }
    public String getContent() {
        return content;
    }
    public String getAnswer() {
        return answer;
    }
}
```

### QuestionRequest.java

- **역할**: 질문 등록 요청 body 형식과 검증 규칙을 정의하는 DTO(Data Transfer Object)

```java
package com.example.rag;

// spring-boot-starter-validation 의존성으로 사용 가능한 검증 어노테이션
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

// record: 필드(private final), 생성자, getter, equals(), hashCode(), toString() 자동 생성
// getter 이름은 getContent()가 아닌 content() 형태
// 요청 JSON {"content": "..."}의 content 값이 아래 content 필드에 매핑
public record QuestionRequest(
    // null, 빈 문자열 "", 공백만 있는 "   " 모두 거절
    // 실패 시 message 문구가 에러 메시지로 사용
    @NotBlank(message = "질문을 입력해주세요.")
    // 200자 초과 시 거절
    @Size(max = 200, message = "질문은 200자 이내로 입력해주세요.")
    String content
) {
    // 컨트롤러의 @Valid가 위 규칙 검사
    // 통과 → 컨트롤러 메서드 실행 / 실패 → MethodArgumentNotValidException → ApiErrorHandler
}
```

### ApiErrorHandler.java

- **역할**: 컨트롤러에서 발생한 예외를 한곳에서 받아 {"message": "..."} 형태로 응답

```java
package com.example.rag;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.server.ResponseStatusException;

import java.util.Map;

// 모든 @RestController에서 발생한 예외를 가로채는 전역 예외 처리 클래스
// 반환값은 JSON으로 응답
// 이 클래스가 없으면 스프링 기본 에러 JSON(timestamp, status, error, path 등)이 그대로 응답
@RestControllerAdvice
public class ApiErrorHandler {

    // ResponseStatusException 발생 시 실행
    // ResponseStatusException: throw new ResponseStatusException(HttpStatus.NOT_FOUND, "이유")처럼
    //                          상태 코드와 이유를 함께 던지는 예외
    @ExceptionHandler(ResponseStatusException.class)
    public ResponseEntity<Map<String, String>> status(ResponseStatusException error) {
        // 주의: null 체크는 getMessage()로, 반환은 getReason()으로 하는 중
        //       reason이 null이면 아래 Map.of()에서 NullPointerException 발생 가능
        String message = error.getMessage() == null ? "요청을 처리하지 못했습니다." : error.getReason();
        // getStatusCode(): 예외에 담긴 상태 코드(404 등)를 그대로 응답 코드로 사용
        // 주의: key가 "msessage"로 오타 → validation()의 "message"와 형식 불일치
        return ResponseEntity.status(error.getStatusCode()).body(Map.of("msessage", message));
    }

    // @Valid 검사 실패 시 스프링이 던지는 예외 처리
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> validation(MethodArgumentNotValidException error) {
        String message = error.getBindingResult()   // 검증 결과
                .getFieldErrors()                    // 검사에 실패한 필드 에러 목록
                .stream()
                .map(e -> e.getDefaultMessage())     // 각 에러의 message 값(@NotBlank의 message 등)만 추출
                .findFirst()                         // 첫 번째 메시지 하나만 사용
                .orElse("입력 값을 입력해주세요.");     // 메시지가 없을 때 기본 문구
        // badRequest(): 400 Bad Request
        // Map.of(): {"message": "질문을 입력해주세요."} 형태의 JSON body
        return ResponseEntity.badRequest().body(Map.of("message", message));
    }

}
```

### RagApplicationTests.java

- **역할**: 스프링 애플리케이션이 정상 실행되는지 확인하는 기본 테스트

```java
package com.example.rag;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

// 실제 실행과 동일하게 스프링 컨테이너 전체를 띄우는 통합 테스트
// JUnit 5는 테스트 클래스가 public이 아니어도 실행 가능
@SpringBootTest
class RagApplicationTests {

    // JUnit 5 테스트 메서드
    @Test
    void contextLoads() {
        // 내용은 비어 있음 → 스프링 컨테이너가 뜨는 과정 자체가 검사 대상
        // 모든 빈(Controller, Service, ErrorHandler) 생성·주입 성공 시 통과, 실패 시 테스트 실패
    }

}
```

## 6. 개발 지식
### MVC 패턴
![[Pasted image 20261124190838.png]]
- Model, View, Controller의 약자.
	-  Model: 핵심 데이터, 비즈니스 로직 담당. 데이터베이스 연동이나 규칙 처리 맡으며 뷰나 컨트롤러 의존 X -> 독립적 동작
	- View: 사용자에게 보이는 UI 구성, 모델의 데이터를 바탕으로 화면을 표시하고, 사용자의 입력 이벤트를 감지.
	- Controller: 사용자 요청을 받아 흐름을 제어하고 모델과 뷰 연결. 입력에 따라 모델 변경, 처리 결과 뷰에 전달
- M-V-C 상호작용
	- 1. 사용자가 애플리케이션에서 작업 수행 시 View가 사용자의 입력 감지하고 Controller에 전달
	- 2. Controller는 사용자 입력을 처리하고 적절한 Model 기능을 호출하여 데이터를 검색, 수정, 저장
	- 3. Model은 데이터와 관련된 비즈니스 로직을 수행하고, 필요 경우 데이터베이스와 상호작용 Model은 작업이 완료되면 결과를 Controller에 반환
	- 4. Controller는 Model의 결과를 받아 View에 전달. View는 이 데이터를 사용하여 사용자에게 보여지는 화면 업데이트
- MVC 패턴 설계 원칙
	- 1. Model은 Controller와 View에 의존하지 않아야 한다.
	- 2. View는 Model에만 의존해야 하고, Controller에는 의존하면 안된다.
	- 3. View가 Model로부터 데이터를 받을 때는, 사용자마다 다르게 보여주어야 하는 데이터에 대해서만 받는다.

### REST API

HTTP 프로토콜 기반으로 URL로 자원을 지정하고 HTTP 메소드로 행위를 정의해 데이터를 주고받는 아키텍처 스타일의 API
- GET: 조회
- POST: 등록
- PUT: 전체 수정
- PATCH: 일부 수정
- DELETE: 삭제
- QUERY: Body 포함 안전 조회

### 동시성

- 하나의 프로세스가 여러 태스크를 동시에 처리하는 것처럼 보이게 하는 것
- 하나의 코어에선 스레드를 통해 2개 이상의 작업을 아주 짧은 시간으로 나누고 번갈아 빠르게 처리함으로써, 사용자 눈엔 동시에 수행되는 것처럼 느끼게 되니까 동시에 처리하는 것처럼 보인다.

### 트래픽

- 서버를 통해 전송되는 데이터의 양이나 특정 웹사이트 / 앱에 방문하는 이용자의 수와 활동량

### 그 외

1. 브라우저에서 주소 값을 입력하면 JSON 응답이 오는 이유
   -> 서버가 화면 대신 데이터를 JSON으로 변환해 반환하기 때문.
	사용자가 브라우저 주소 값을 입력하면 브라우저가 자동으로 이 주소에 있는 데이터를 조회해서 달라는 HTTP GET 요청을 생성하여

2. HTTP 메소드 종류
	-> GET (조회), POST (등록), PUT (전체수정), PATCH (일부수정), DELETE (삭제), QUERY (Body 포함 안전 조회)

3. 스프링에서 컨트롤러의 기능
	-> 클라이언트의 HTTP(URL) 요청을 받아서 서비스에 넘기고 처리 결과를 응답으로 반환하는 역할

4. 포트를 겹쳐서 서버 실행하면 생기는 일
	=> 포트 충돌이 나서 에러가 발생한 후 서버가 켜지지 않는다 (운영체제가 차단)
	포트: 한 컴퓨터 내에 있는 네트워크 통신을 하는 여러 프로그램들의 고유 번호
