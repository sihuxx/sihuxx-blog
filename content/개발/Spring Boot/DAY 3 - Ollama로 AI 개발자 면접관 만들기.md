## 1. 프로젝트 개요

- Spring AI + Ollama로 만든 **AI 개발자 면접관** 웹앱
- 지원자 정보(이름, 직무, 경력, 기술 스택, 대표 경험)와 면접관 성격(친절·깐깐·압박)을 고르면 5개 질문으로 모의 면접 진행
- 질문 순서: 자기소개 → 프로젝트 경험 → 기술 꼬리질문 → CS 기초 → 협업·인성
    - 자기소개: 성격별 고정 문장
    - 프로젝트·꼬리질문: AI가 생성
    - CS·협업: 미리 만들어 둔 질문 은행에서 랜덤 선택
- 답변마다 AI가 논리성·구체성·기술 이해를 1~10점으로 채점, 마지막에 총평·잘한 점·아쉬운 점·모범 답변 리포트 작성
- 평균·총점·합격 여부는 AI가 아닌 코드가 계산 (평균 6점 이상 합격)
- 채점·리포트는 백그라운드 스레드에서 처리 → 답변 후 바로 다음 질문으로 진행
- 기존 질문 게시판(`/api/questions`)은 고정 문구 대신 AI 답변을 저장하도록 변경

|항목|내용|
|---|---|
|언어|Java 25|
|프레임워크|Spring Boot 4.2.0, Spring AI 2.1.0|
|빌드 도구|Gradle|
|의존성|`spring-boot-starter-webmvc`, `spring-boot-starter-validation`, `mybatis-spring-boot-starter`, `mysql-connector-j`, `spring-ai-starter-model-ollama`|
|DB|MySQL (`school_ai`) — `questions`, `interviews`, `interview_turns` 테이블|
|AI 모델|Ollama `exaone3.5:7.8b` (채팅), `bge-m3` (임베딩)|
|포트|8081|

## 2. 폴더 구성

```
school-spring-ai-day-3/
├── build.gradle
├── settings.gradle
├── README.md / DB-연결.md / 검증.md
├── 퍼블리싱/
└── src/main/
    ├── java/com/example/rag/
    │   ├── RagApplication.java
    │   ├── config/
    │   │   └── AiExecutorConfig.java
    │   ├── controller/
    │   │   ├── InterviewController.java
    │   │   └── QuestionController.java
    │   ├── service/
    │   │   ├── InterviewService.java
    │   │   ├── InterviewerAiService.java
    │   │   ├── QuestionService.java
    │   │   └── AnswerService.java
    │   ├── repository/
    │   │   ├── InterviewMapper.java
    │   │   ├── QuestionBank.java
    │   │   └── QuestionMapper.java
    │   ├── model/
    │   │   ├── Interview.java
    │   │   ├── InterviewTurn.java
    │   │   ├── TurnType.java
    │   │   ├── BankQuestion.java
    │   │   └── Question.java
    │   ├── dto/
    │   │   ├── InterviewStartRequest.java
    │   │   ├── AnswerRequest.java
    │   │   ├── InterviewState.java
    │   │   ├── InterviewDetail.java
    │   │   ├── TurnResult.java
    │   │   ├── TurnEvaluation.java
    │   │   ├── ReportText.java
    │   │   ├── InterviewReport.java
    │   │   └── QuestionRequest.java
    │   ├── exception/
    │   │   └── ApiErrorHandler.java
    │   └── practice/
    │       └── ObjectPractice.java
    └── resources/
        ├── application.yaml       # DB, Ollama 모델, 면접관 성격별 말투 설정
        ├── schema.sql             # 서버 시작 시 테이블 자동 생성
        └── static/
            ├── index.html / app.js / style.css
            └── images/            # 면접관 성격별 이미지
```

|패키지|역할|
|---|---|
|`config`|백그라운드 작업용 스레드 설정|
|`controller`|HTTP 요청 수신, 응답 반환|
|`service`|면접 진행 로직, AI 호출|
|`repository`|DB SQL 실행(MyBatis), 질문 은행|
|`model`|DB 테이블과 대응되는 데이터 클래스, 질문 유형 enum|
|`dto`|요청·응답 형식|
|`exception`|예외 → 에러 응답 변환|
|`practice`|객체 생성 연습용 코드|

## 3. 요청 흐름

```
[면접 시작]
브라우저 → InterviewController.start → InterviewService.start
         → DB에 면접 + 1번 질문(자기소개) 저장 → 1번 질문 응답
         → (뒤에서) InterviewerAiService가 2번 프로젝트 질문 미리 생성

[답변 제출] (5번 반복)
브라우저 → InterviewController.answer → InterviewService.answer
         → 답변 저장 + 다음 질문 결정
              ├─ 프로젝트: 미리 만든 질문 사용
              ├─ 기술 꼬리질문: InterviewerAiService가 직전 답변 보고 생성
              └─ CS·협업: QuestionBank에서 랜덤 선택
         → 다음 질문 바로 응답
         → (뒤에서) aiExecutor가 답변 채점
         → 마지막 답변이면 (뒤에서) 남은 채점 + 총평 작성 시작

[결과 리포트]
브라우저 → 2초마다 InterviewController.report 요청
         → 준비된 점수·총평부터 채워서 응답 (ready=true가 될 때까지 반복)

[에러]
검증 실패(400), 없는 면접(404), 이미 끝난 면접(409), AI 연결 실패(503), DB 오류(503)
→ ApiErrorHandler → {"message": "..."} 응답
```

## 4. API

### 질문 게시판

|메서드|주소|설명|응답|
|---|---|---|---|
|POST|`/api/questions`|질문 등록 (AI 답변 저장)|201 + 등록된 질문|
|GET|`/api/questions`|질문 전체 조회|200 + 질문 목록|
|PUT|`/api/questions/{id}`|질문 수정|200 + 수정된 질문|
|DELETE|`/api/questions/{id}`|질문 삭제|204|

### 면접 AI

|메서드|주소|설명|응답|
|---|---|---|---|
|POST|`/api/interviews`|면접 시작|201 + 첫 질문|
|GET|`/api/interviews/{id}`|면접 정보 (이어하기, 기록)|200 + 면접·질문 목록·현재 질문|
|POST|`/api/interviews/{id}/answers`|답변 제출|200 + 다음 질문|
|GET|`/api/interviews/{id}/report?retry=false`|결과 리포트|200 + 점수·총평 (준비된 것까지)|

면접 시작 요청 예시

```json
{
  "name": "박시후",
  "track": "백엔드",
  "level": "고졸 신입",
  "stack": ["Java", "Spring", "MySQL"],
  "project": "학교 안내 챗봇 개발",
  "persona": "strict"
}
```

답변 제출 요청 예시

```json
{ "content": "저는 백엔드 개발자를 목표로 하는 박시후입니다..." }
```

## 5. Java 파일 설명

### 5-1. 공통

#### RagApplication.java

- **역할**: 스프링 부트 애플리케이션 실행 시작점

```java
package com.example.rag; // 루트 패키지. 하위 패키지 전체가 스캔 대상

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// @Configuration + @EnableAutoConfiguration + @ComponentScan
// 의존성 보고 자동 설정 (내장 톰캣, MySQL DataSource, MyBatis, Spring AI ChatClient 등)
@SpringBootApplication
public class RagApplication {

    public static void main(String[] args) {
        // 스프링 컨테이너 생성 → schema.sql 실행 → 빈 등록·주입 → 8081 포트로 서버 실행
        SpringApplication.run(RagApplication.class, args);
    }

}
```

#### ApiErrorHandler.java

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

// 모든 Controller의 예외를 가로채는 전역 예외 처리 클래스
@RestControllerAdvice
public class ApiErrorHandler {

    // 서비스에서 직접 던진 상태 예외 처리
    // 404(없는 면접·질문), 409(이미 끝난 면접·중복 답변), 503(AI 연결 실패), 502(AI 빈 응답)
    @ExceptionHandler(ResponseStatusException.class)
    public ResponseEntity<Map<String, String>> status(ResponseStatusException error) {
        String message = error.getReason() == null ? "요청을 처리하지 못했습니다." : error.getReason();
        return ResponseEntity.status(error.getStatusCode()).body(Map.of("message", message));
    }

    // @Valid 검증 실패 → 400, 첫 번째 에러 메시지만 응답
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> validation(MethodArgumentNotValidException error) {
        String message = error.getBindingResult().getFieldErrors().stream()
                .map(e -> e.getDefaultMessage()).findFirst().orElse("입력값을 확인해 주세요.");
        return ResponseEntity.badRequest().body(Map.of("message", message));
    }

    // DB 오류 → 503
    @ExceptionHandler(DataAccessException.class)
    public ResponseEntity<Map<String, String>> database(DataAccessException error) {
        return ResponseEntity.status(503).body(Map.of("message",
                "DB 작업에 실패했습니다. MySQL 실행 상태와 접속 설정을 확인해 주세요."));
    }
}
```

### 5-2. 질문 게시판 (Question)

#### QuestionController.java

- **역할**: 질문 게시판 CRUD 요청을 받아 `QuestionService`에 전달 (2일차와 동일, PUT 주소 수정됨)

```java
package com.example.rag.controller;

import com.example.rag.dto.QuestionRequest;
import com.example.rag.model.Question;
import com.example.rag.service.QuestionService;

import java.util.List;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/questions")
public class QuestionController {
    private final QuestionService service;
    public QuestionController(QuestionService service) { this.service = service; }

    // POST /api/questions → 질문 등록 (AI 답변과 함께 저장)
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Question create(@Valid @RequestBody QuestionRequest request) {
        return service.create(request.content());
    }

    // GET /api/questions → 전체 조회
    @GetMapping
    public List<Question> findAll() { return service.findAll(); }

    // PUT /api/questions/{id} → 수정 (2일차에 빠져 있던 "/{id}" 추가됨)
    @PutMapping("/{id}")
    public Question update(@PathVariable long id, @Valid @RequestBody QuestionRequest request) {
        return service.update(id, request.content());
    }

    // DELETE /api/questions/{id} → 삭제, 204 No Content
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void delete(@PathVariable long id) { service.delete(id); }
}
```

#### QuestionService.java

- **역할**: 질문 게시판 CRUD 처리. 질문 저장 시 `AnswerService`로 AI 답변 생성

```java
package com.example.rag.service;

import java.util.List;
import com.example.rag.model.Question;
import com.example.rag.repository.QuestionMapper;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.TransactionTemplate;
import org.springframework.web.server.ResponseStatusException;

@Service
public class QuestionService {
    private final QuestionMapper mapper;
    private final TransactionTemplate transaction;
    private final AnswerService answerService;   // 3일차 추가: AI 답변 생성

    public QuestionService(QuestionMapper mapper, TransactionTemplate transaction, AnswerService answerService) {
        this.mapper = mapper;
        this.transaction = transaction;
        this.answerService = answerService;
    }

    public Question create(String content) {
        // AI 답변을 트랜잭션 밖에서 먼저 생성
        // → AI가 실패하면 DB 저장을 시작조차 하지 않음
        // → AI 응답을 기다리는 동안 DB 연결을 붙잡고 있지 않음
        Question question = makeQuestion(0, content.strip());
        return transaction.execute(status -> {
            mapper.insert(question); // AUTO_INCREMENT 번호가 question.id에 들어옴
            return question;
        });
    }

    @Transactional(readOnly = true)
    public List<Question> findAll() {
        return mapper.findAll();
    }

    public Question update(long id, String content) {
        if (mapper.findById(id) == null) throw notFound();
        // 수정된 질문으로 AI 답변 다시 생성 (쓰기 트랜잭션 전에 끝냄)
        Question updated = makeQuestion(id, content.strip());
        return transaction.execute(status -> {
            if (mapper.update(updated) == 0) throw notFound();
            return updated;
        });
    }

    public void delete(long id) {
        if (mapper.delete(id) == 0) throw notFound();
    }

    private ResponseStatusException notFound() {
        return new ResponseStatusException(HttpStatus.NOT_FOUND, "질문을 찾을 수 없습니다.");
    }

    // 2일차: 고정 문구 → 3일차: AI 답변
    private Question makeQuestion(long id, String content) {
        return new Question(id, content, answerService.answer(content));
    }
}
```

#### AnswerService.java

- **역할**: 질문 게시판용 AI 답변 생성 (면접 기능은 `InterviewerAiService` 사용)

```java
package com.example.rag.service;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ResponseStatusException;

@Service
public class AnswerService {
    private final ChatClient chatClient;
    private final String systemPrompt;

    // @Value("${app.system-prompt:기본값}"): yaml에 app.system-prompt가 없으면 : 뒤의 기본 문장 사용
    public AnswerService(ChatClient.Builder builder,
                         @Value("${app.system-prompt:너는 질문에 한국어 존댓말로 짧게 답하는 AI 도우미야.}") String systemPrompt) {
        this.chatClient = builder.build();
        this.systemPrompt = systemPrompt;
    }

    public String answer(String question) {
        String reply;
        try {
            // 시스템 프롬프트 + 사용자 질문으로 AI 호출 → 응답 글자 받기
            reply = chatClient.prompt().system(systemPrompt).user(question).call().content();
        } catch (RuntimeException error) {
            // Ollama 연결 실패 → 503
            throw new ResponseStatusException(HttpStatus.SERVICE_UNAVAILABLE,
                "AI 서버에 연결하지 못했습니다. 모델 상태를 확인해 주세요.");
        }
        // 빈 응답 → 502
        if (reply == null || reply.isBlank()) {
            throw new ResponseStatusException(HttpStatus.BAD_GATEWAY, "AI 답변이 비어있습니다.");
        }
        return reply.strip();
    }
}
```

#### QuestionMapper.java

- **역할**: MyBatis로 `questions` 테이블에 SQL 실행 (2일차와 동일)

```java
package com.example.rag.repository;

import java.util.List;
import com.example.rag.model.Question;
import org.apache.ibatis.annotations.*;

// MyBatis가 구현 객체를 자동 생성. #{...}는 값을 안전하게 바인딩 (SQL 인젝션 방지)
@Mapper
public interface QuestionMapper {
    // 질문 저장, 생성된 id를 객체에 세팅
    @Insert("INSERT INTO questions (content, answer) VALUES (#{content}, #{answer})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insert(Question question);

    // 전체 조회 (id 순)
    @Select("SELECT id, content, answer FROM questions ORDER BY id")
    List<Question> findAll();

    // id로 하나 조회, 없으면 null
    @Select("SELECT id, content, answer FROM questions WHERE id = #{id}")
    Question findById(@Param("id") long id);

    // 수정, 반환값 0 → 해당 id 없음
    @Update("UPDATE questions SET content = #{content}, answer = #{answer} WHERE id = #{id}")
    int update(Question question);

    // 삭제, 반환값 0 → 해당 id 없음
    @Delete("DELETE FROM questions WHERE id = #{id}")
    int delete(@Param("id") long id);
}
```

#### Question.java

- **역할**: `questions` 테이블 한 행(질문 게시판 글)에 대응되는 클래스 (2일차와 동일)

```java
package com.example.rag.model;

public class Question {
    private long id;          // 질문 번호
    private String content;   // 질문 내용
    private String answer;    // 답변 (3일차부터 AI 답변)

    // MyBatis가 조회 결과를 담을 때 사용하는 기본 생성자와 setter
    public Question() { }
    public Question(long id, String content, String answer) {
        this.id = id;
        this.content = content;
        this.answer = answer;
    }
    public long getId() { return id; }
    public void setId(long id) { this.id = id; }
    public String getContent() { return content; }
    public void setContent(String content) { this.content = content; }
    public String getAnswer() { return answer; }
    public void setAnswer(String answer) { this.answer = answer; }
}
```

#### QuestionRequest.java

- **역할**: 질문 게시판 등록·수정 요청 형식과 검증 규칙 (2일차와 동일)

```java
package com.example.rag.dto;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

// 요청 JSON {"content": "..."} → content 필드
public record QuestionRequest(
        @NotBlank(message = "질문을 입력해 주세요.")                       // 빈 값·공백 거절
        @Size(max = 200, message = "질문은 200자 이내로 입력해 주세요.")   // 200자 초과 거절
        String content) {
}
```

#### ObjectPractice.java

- **역할**: `Question` 객체 생성과 getter 사용 연습용 코드 (서버와 별개로 실행, 2일차와 동일)

```java
package com.example.rag.practice;

import com.example.rag.model.Question;

public class ObjectPractice {
    // 스프링과 상관없이 단독 실행하는 main 메서드
    public static void main(String[] args) {
        // new로 Question 객체 2개 생성
        Question first = new Question(1L, "도서관은 언제 열어요?", "안내문을 준비 중입니다.");
        Question second = new Question(2L, "동아리는 언제 신청해요?", "안내문을 준비 중입니다.");
        // getter로 값을 꺼내 콘솔에 출력
        System.out.println(first.getId() + " " + first.getContent() + " " + first.getAnswer());
        System.out.println(second.getId() + " " + second.getContent() + " " + second.getAnswer());
    }
}
```

### 5-3. 면접 AI (Interview)

#### AiExecutorConfig.java

- **역할**: AI 채점·리포트처럼 오래 걸리는 작업을 뒤에서 처리할 스레드 1개를 빈으로 등록

```java
package com.example.rag.config;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

// @Configuration: 빈을 직접 만들어 등록하는 설정 클래스
@Configuration
public class AiExecutorConfig {

    // @Bean: 메서드 반환값을 스프링 빈으로 등록
    // name = "aiExecutor": 다른 곳에서 @Qualifier("aiExecutor")로 이 빈을 지정해 주입
    // destroyMethod = "shutdown": 서버 종료 시 스레드도 정리
    @Bean(name = "aiExecutor", destroyMethod = "shutdown")
    public ExecutorService aiExecutor() {
        // newSingleThreadExecutor: 작업을 맡기면 스레드 1개가 순서대로 하나씩 처리
        // Ollama도 한 번에 하나씩 처리하므로 스레드 1개로 충분
        return Executors.newSingleThreadExecutor(task -> {
            // 스레드 이름 지정 → 로그에서 구분하기 쉬움
            Thread thread = new Thread(task, "ai-background");
            // 데몬 스레드: 메인 프로그램이 끝나면 같이 종료
            thread.setDaemon(true);
            return thread;
        });
    }
}
```

#### InterviewController.java

- **역할**: 면접 시작·조회·답변 제출·결과 리포트 요청을 받아 `InterviewService`에 전달

```java
package com.example.rag.controller;

import com.example.rag.dto.*;
import com.example.rag.service.InterviewService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.*;

// 반환값을 JSON으로 응답하는 컨트롤러
@RestController
// 모든 주소 앞에 /api/interviews
@RequestMapping("/api/interviews")
public class InterviewController {
    private final InterviewService service;
    // 생성자 주입
    public InterviewController(InterviewService service) { this.service = service; }

    // POST /api/interviews → 면접 시작
    // 지원자 정보 저장 + 첫 질문(자기소개) 응답
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED) // 201
    public InterviewState start(@Valid @RequestBody InterviewStartRequest request) {
        return service.start(request);
    }

    // GET /api/interviews/{id} → 면접 전체 정보
    // 새로고침 후 이어하기, 지난 면접 기록 보기에 사용
    @GetMapping("/{id}")
    public InterviewDetail detail(@PathVariable long id) { return service.detail(id); }

    // POST /api/interviews/{id}/answers → 답변 제출
    // 답변 저장 후 다음 질문 응답 (채점은 뒤에서 따로 진행)
    @PostMapping("/{id}/answers")
    public TurnResult answer(@PathVariable long id, @Valid @RequestBody AnswerRequest request) {
        return service.answer(id, request.content());
    }

    // GET /api/interviews/{id}/report → 결과 리포트
    // 기다리지 않고 지금까지 준비된 것만 바로 응답 → 화면이 2초마다 다시 요청
    // @RequestParam: 주소의 ?retry=true 값을 받음. 없으면 기본값 false
    // retry=true: 실패했던 결과 정리를 다시 시작
    @GetMapping("/{id}/report")
    public InterviewReport report(@PathVariable long id,
                                  @RequestParam(defaultValue = "false") boolean retry) {
        return service.report(id, retry);
    }
}
```

#### InterviewService.java

- **역할**: 면접 진행 담당. 몇 번째 질문인지, 다음 질문을 어디서 가져올지, 언제 채점하고 끝낼지 결정

```java
package com.example.rag.service;

import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;
import com.example.rag.dto.*;
import com.example.rag.model.BankQuestion;
import com.example.rag.model.Interview;
import com.example.rag.model.InterviewTurn;
import com.example.rag.model.TurnType;
import com.example.rag.repository.InterviewMapper;
import com.example.rag.repository.QuestionBank;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.event.EventListener;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.transaction.support.TransactionTemplate;
import org.springframework.web.server.ResponseStatusException;

// [답변이 빨리 넘어가게 하는 방법]
// - 답변을 내면 다음 질문만 바로 응답, 채점은 뒤에서(aiExecutor) 처리
// - AI가 만들 질문이 남아 있는 동안(Q3 꼬리질문까지)은 채점을 미뤄 Ollama를 질문 생성에 양보
//   (Ollama는 한 번에 하나씩 처리 → 채점이 먼저 줄 서 있으면 다음 질문이 늦어짐)
// - 서버 시작 시 모델을 미리 한 번 호출해 깨워 둠
// - CS·협업 질문은 질문 은행에서 바로 꺼냄 (AI 호출 없음)
// - 2번 프로젝트 질문은 면접 시작할 때 미리 생성
// - 마지막 답변 순간 결과 정리(남은 채점 + 총평)를 뒤에서 바로 시작
// - 결과 화면은 바로 열리고 준비된 것부터 채워짐 (2초마다 재요청)
@Service
public class InterviewService {
    // 로그 출력용 객체
    private static final Logger log = LoggerFactory.getLogger(InterviewService.class);
    // 합격 기준: 세 항목 평균 6점 이상
    private static final double PASS_AVERAGE = 6.0;

    private final InterviewMapper mapper;          // 면접·질문 턴 DB 저장
    private final QuestionBank bank;               // CS·협업 질문 은행
    private final InterviewerAiService ai;         // AI 질문 생성·채점·리포트
    private final TransactionTemplate transaction; // 트랜잭션 처리
    private final ExecutorService aiExecutor;      // 백그라운드 작업 스레드

    // ConcurrentHashMap: 여러 스레드(요청 스레드 + ai-background)가 동시에 써도 안전한 Map

    // 미리 만들어 둔 프로젝트 질문 (면접 번호 → 질문 생성 작업)
    // CompletableFuture: 나중에 결과가 채워지는 비동기 작업의 결과 상자
    private final Map<Long, CompletableFuture<String>> prefetched = new ConcurrentHashMap<>();
    // 채점을 이미 맡긴 질문 턴 id 목록 (두 번 채점 방지)
    private final Set<Long> queuedScoring = ConcurrentHashMap.newKeySet();
    // 진행 중인 결과 정리 작업 (면접 번호 → 작업)
    private final Map<Long, CompletableFuture<Void>> reportJobs = new ConcurrentHashMap<>();
    // 결과 정리 실패 메시지 (면접 번호 → 메시지)
    private final Map<Long, String> reportErrors = new ConcurrentHashMap<>();

    // 생성자 주입. ExecutorService 빈이 여러 개일 수 있어 @Qualifier로 "aiExecutor" 지정
    public InterviewService(InterviewMapper mapper, QuestionBank bank, InterviewerAiService ai,
                            TransactionTemplate transaction,
                            @Qualifier("aiExecutor") ExecutorService aiExecutor) {
        this.mapper = mapper;
        this.bank = bank;
        this.ai = ai;
        this.transaction = transaction;
        this.aiExecutor = aiExecutor;
    }

    // ---------- 면접 시작 ----------
    public InterviewState start(InterviewStartRequest request) {
        // 요청 값으로 면접 객체 만들기
        Interview interview = new Interview();
        interview.setName(request.name().strip());
        interview.setTrack(request.track());
        interview.setLevel(request.level());
        interview.setStack(joinStack(request.stack()));   // ["React","Spring"] → "React,Spring"
        interview.setProject(request.project() == null ? "" : request.project().strip());
        interview.setPersona(request.persona());
        interview.setStatus(Interview.ONGOING);            // 진행 중 상태

        // 1번 질문(자기소개)은 성격별 고정 문장 → AI 호출 없이 바로 시작
        String first = firstQuestion(interview);

        // 면접 저장 + 1번 질문 저장을 하나의 트랜잭션으로
        InterviewState state = transaction.execute(status -> {
            mapper.insertInterview(interview);             // 실행 후 interview.id 채워짐
            InterviewTurn turn = new InterviewTurn(interview.getId(), 1, TurnType.INTRO, first, null);
            mapper.insertTurn(turn);
            return InterviewState.of(turn);                // 화면에 보낼 질문 정보
        });

        // 지원자가 자기소개 답변을 쓰는 동안 2번 프로젝트 질문을 뒤에서 미리 생성
        // supplyAsync(작업, 스레드): 작업을 aiExecutor에 맡기고 결과는 나중에 받음
        prefetched.put(interview.getId(), CompletableFuture.supplyAsync(
                () -> ai.askQuestion(interview, TurnType.PROJECT, null), aiExecutor));
        return state;
    }

    // ---------- 면접 정보 (이어하기, 기록) ----------
    public InterviewDetail detail(long id) {
        Interview interview = findInterview(id);           // 없으면 404
        List<InterviewTurn> turns = mapper.findTurns(id);  // 지금까지의 질문 턴 전부
        InterviewTurn current = currentTurn(turns);        // 아직 답변 안 한 질문
        return new InterviewDetail(interview, turns, current == null ? null : InterviewState.of(current));
    }

    // ---------- 답변 제출 → 다음 질문 (채점은 뒤에서) ----------
    public TurnResult answer(long id, String content) {
        Interview interview = findInterview(id);
        // 이미 끝난 면접이면 409 Conflict
        if (!Interview.ONGOING.equals(interview.getStatus())) {
            throw new ResponseStatusException(HttpStatus.CONFLICT, "이미 끝난 면접입니다. 결과를 확인해 주세요.");
        }
        List<InterviewTurn> turns = mapper.findTurns(id);
        InterviewTurn current = currentTurn(turns);        // 지금 답변할 질문
        if (current == null) {
            throw new ResponseStatusException(HttpStatus.CONFLICT, "답변할 질문이 없습니다.");
        }
        current.setAnswer(content.strip());

        // 마지막(5번) 질문인지 확인
        boolean last = current.getTurnNo() >= TurnType.TOTAL;
        // 마지막이 아니면 다음 질문 준비 (AI 호출은 트랜잭션 밖에서 먼저 끝냄)
        InterviewTurn next = last ? null : nextTurn(interview, turns, current);

        // 답변 저장 + 다음 질문 저장(또는 면접 종료)을 하나의 트랜잭션으로
        transaction.executeWithoutResult(status -> {
            // saveAnswer는 answer IS NULL 조건 → 이미 답변한 질문이면 0행 → 409
            if (mapper.saveAnswer(current.getId(), current.getAnswer()) == 0) {
                throw new ResponseStatusException(HttpStatus.CONFLICT, "이미 답변한 질문입니다. 화면을 새로고침해 주세요.");
            }
            if (next != null) mapper.insertTurn(next);
            else mapper.updateStatus(id, Interview.DONE);  // 마지막이면 면접 종료 상태로
        });

        // 앞으로 AI가 만들 질문이 더 없으면 그때 채점 시작 (그 전엔 질문 생성에 양보)
        if (!aiQuestionsLeftAfter(current.getTurnNo() + 1)) {
            // DB에서 읽은 목록 중 current만 방금 답변이 들어간 객체로 교체
            List<InterviewTurn> answered = new java.util.ArrayList<>(turns);
            answered.replaceAll(t -> t.getId() == current.getId() ? current : t);
            for (InterviewTurn t : answered) {
                // 답변 O, 채점 X, 아직 채점을 맡기지 않은 질문만
                // queuedScoring.add()는 처음 넣을 때만 true → 중복 채점 방지
                if (t.isAnswered() && !t.isScored() && queuedScoring.add(t.getId())) {
                    // 채점 작업을 aiExecutor에 맡김 (결과를 기다리지 않음)
                    CompletableFuture.runAsync(() -> score(interview, t), aiExecutor);
                }
            }
        }
        // 마지막 답변이면 결과 정리 시작 (스레드가 1개라 위 채점이 끝난 다음 순서로 실행)
        if (last) startReportJob(id);

        return new TurnResult(current.getTurnNo(), last, next == null ? null : InterviewState.of(next));
    }

    // ---------- 최종 리포트 ----------
    // 기다리지 않고 지금까지 준비된 것만 바로 응답. 안 된 값은 null
    public InterviewReport report(long id, boolean retry) {
        Interview interview = findInterview(id);
        // 면접이 끝나지 않았으면 409
        if (!Interview.DONE.equals(interview.getStatus())) {
            throw new ResponseStatusException(HttpStatus.CONFLICT, "면접을 끝까지 마쳐야 결과를 볼 수 있어요.");
        }
        List<InterviewTurn> turns = mapper.findTurns(id);
        // 채점 끝난 질문 수
        int scoredCount = (int) turns.stream().filter(InterviewTurn::isScored).count();
        boolean allScored = scoredCount == turns.size();
        boolean summaryReady = interview.getSummary() != null;   // 총평 작성 여부
        boolean ready = allScored && summaryReady;               // 전부 준비 완료 여부

        // 덜 됐는데 진행 중인 작업이 없으면(서버 재시작 등) 다시 시작
        // 이전에 실패한 적이 있으면 retry=true일 때만 재시작
        if (!ready && !isRunning(id)) {
            if (retry || !reportErrors.containsKey(id)) startReportJob(id);
        }

        // 숫자는 전부 코드가 계산 (모든 채점이 끝났을 때만)
        Double logic = null, detail = null, tech = null;
        Integer totalScore = null;
        Boolean passed = null;
        String weakestQuestion = null;
        if (allScored) {
            // 항목별 평균 (소수 첫째 자리 반올림)
            logic = round1(turns.stream().mapToInt(InterviewTurn::getLogicScore).average().orElse(0));
            detail = round1(turns.stream().mapToInt(InterviewTurn::getDetailScore).average().orElse(0));
            tech = round1(turns.stream().mapToInt(InterviewTurn::getTechScore).average().orElse(0));
            double overall = (logic + detail + tech) / 3.0;      // 전체 평균 (10점 만점)
            totalScore = (int) Math.round(overall * 10);         // 100점 만점으로 환산
            passed = overall >= PASS_AVERAGE;                    // 합격 여부
            weakestQuestion = weakest(turns).getQuestion();      // 평균이 가장 낮은 질문
        }

        return new InterviewReport(interview.getId(), interview.getName(), interview.getTrack(),
                interview.getLevel(), interview.getPersona(),
                ready, scoredCount, turns.size(), summaryReady, reportErrors.get(id),
                logic, detail, tech, totalScore, passed,
                interview.getSummary(),
                splitLines(interview.getStrengths()), splitLines(interview.getImprovements()),
                weakestQuestion, interview.getBetterAnswer(), turns);
    }

    // 결과 정리 작업이 아직 실행 중인지 확인 (끝난 작업은 목록에서 제거)
    private boolean isRunning(long id) {
        CompletableFuture<Void> job = reportJobs.get(id);
        if (job != null && job.isDone()) {
            reportJobs.remove(id, job);
            return false;
        }
        return job != null;
    }

    // 결과 정리 작업 시작 (이미 실행 중이면 그대로 둠)
    private void startReportJob(long id) {
        if (isRunning(id)) return;
        reportErrors.remove(id);   // 이전 실패 기록 삭제
        // computeIfAbsent: 해당 id 작업이 없을 때만 새로 만들어 넣음
        reportJobs.computeIfAbsent(id, key -> CompletableFuture
                .runAsync(() -> prepareReport(key), aiExecutor)
                // whenComplete: 작업이 끝나면(성공·실패 모두) 실행
                .whenComplete((ok, error) -> {
                    if (error != null) {
                        Throwable cause = error.getCause() != null ? error.getCause() : error;
                        log.warn("{}번 면접 결과 정리 실패: {}", key, cause.getMessage());
                        // 실패 메시지 저장 → report 응답의 error 필드로 화면에 표시
                        reportErrors.put(key, "결과를 정리하지 못했어요. Ollama 상태를 확인하고 다시 시도해 주세요.");
                    }
                }));
    }

    // 뒤에서 실행: 채점 안 된 질문 마저 채점 → 총평 작성
    private void prepareReport(long id) {
        Interview interview = findInterview(id);
        List<InterviewTurn> turns = mapper.findTurns(id);
        // 1) 남은 채점
        for (InterviewTurn turn : turns) {
            if (!turn.isScored()) applyScore(turn, ai.evaluate(interview, turn));
        }
        // 2) 총평이 없으면 AI에게 작성 요청 후 DB 저장
        if (interview.getSummary() == null) {
            ReportText text = ai.writeReport(interview, turns, weakest(turns));
            interview.setSummary(blankTo(text.summary(), "면접을 끝까지 마치셨습니다. 수고하셨어요."));
            interview.setStrengths(joinLines(text.strengths()));       // 리스트 → 줄바꿈 문자열
            interview.setImprovements(joinLines(text.improvements()));
            interview.setBetterAnswer(blankTo(text.betterAnswer(), ""));
            mapper.saveReport(interview);
        }
    }

    // 세 점수 평균이 가장 낮은 질문 찾기
    private static InterviewTurn weakest(List<InterviewTurn> turns) {
        return turns.stream().min(Comparator.comparingDouble(InterviewTurn::average)).orElseThrow();
    }

    // turnNo 이후에 AI가 만들어야 하는 질문(fromBank=false)이 남아 있는지 확인
    private static boolean aiQuestionsLeftAfter(int turnNo) {
        for (int k = turnNo + 1; k <= TurnType.TOTAL; k++) {
            if (!TurnType.of(k).isFromBank()) return true;
        }
        return false;
    }

    // @EventListener(ApplicationReadyEvent.class): 서버 시작이 끝나면 자동 실행
    // 모델을 미리 한 번 호출해 메모리에 올려 둠 → 첫 질문 생성 속도 향상
    @EventListener(ApplicationReadyEvent.class)
    public void warmUpModel() {
        CompletableFuture.runAsync(ai::warmUp, aiExecutor);
    }

    // ---------- 채점 ----------
    // 답변마다 뒤에서 실행. 실패해도 면접은 계속, 결과 정리 때 다시 시도
    private void score(Interview interview, InterviewTurn turn) {
        try {
            applyScore(turn, ai.evaluate(interview, turn));
        } catch (RuntimeException error) {
            log.warn("{}번 면접 Q{} 채점 실패: {}", interview.getId(), turn.getTurnNo(), error.getMessage());
        }
    }

    // AI 채점 결과를 턴 객체에 넣고 DB 저장
    private void applyScore(InterviewTurn turn, TurnEvaluation eval) {
        turn.setLogicScore(eval.logic());
        turn.setDetailScore(eval.detail());
        turn.setTechScore(eval.tech());
        turn.setFeedback(blankTo(eval.feedback(), "답변 잘 들었습니다."));
        mapper.saveScores(turn);
    }

    // ---------- 다음 질문 ----------
    private InterviewTurn nextTurn(Interview interview, List<InterviewTurn> turns, InterviewTurn current) {
        int turnNo = current.getTurnNo() + 1;
        TurnType type = TurnType.of(turnNo);   // 번호로 질문 유형 찾기

        // CS·협업: 질문 은행에서 이미 나온 질문을 제외하고 랜덤 선택
        if (type.isFromBank()) {
            Set<String> asked = turns.stream().map(InterviewTurn::getQuestion).collect(Collectors.toSet());
            BankQuestion picked = bank.pick(type, interview.getTrack(), asked);
            return new InterviewTurn(interview.getId(), turnNo, type, picked.question(), picked.keywords());
        }

        String question = null;
        // 프로젝트 질문: 면접 시작 때 미리 만들어 둔 것 사용 (최대 90초 대기)
        if (type == TurnType.PROJECT) {
            CompletableFuture<String> ready = prefetched.remove(interview.getId());
            if (ready != null) {
                try { question = ready.get(90, TimeUnit.SECONDS); } catch (Exception ignored) { /* 아래에서 새로 */ }
            }
        }
        // 기술 꼬리질문, 또는 미리 만든 질문이 없으면 지금 생성
        // 꼬리질문이면 직전 질문·답변(current)을 함께 넘김
        if (question == null || question.isBlank()) {
            try {
                question = ai.askQuestion(interview, type, type == TurnType.TECH ? current : null);
            } catch (RuntimeException error) {
                log.warn("{}번 면접 Q{} 질문 생성 실패, 기본 질문 사용: {}", interview.getId(), turnNo, error.getMessage());
            }
        }
        // AI 실패·빈 응답 대비 기본 질문 (switch 표현식)
        String fallback = switch (type) {
            case PROJECT -> "지금까지 진행한 프로젝트 중 가장 기억에 남는 것과, 그 안에서 맡은 역할을 말씀해 주세요.";
            case TECH -> "방금 말씀하신 기술을 선택한 이유와, 대신 쓸 수 있었던 다른 방법은 무엇이었나요?";
            default -> "조금 더 자세히 말씀해 주시겠어요?";
        };
        return new InterviewTurn(interview.getId(), turnNo, type, blankTo(question, fallback), null);
    }

    // ---------- 도우미 ----------
    // 1번 자기소개 질문: 면접관 성격별 고정 문장 (%s에 이름·직무 채움)
    private static String firstQuestion(Interview interview) {
        String name = interview.getName();
        String track = interview.getTrack();
        return switch (interview.getPersona()) {
            case "strict" -> "%s님. %s 지원이군요.\n1분 자기소개 시작하세요. 시간은 지켜 주시고요.".formatted(name, track);
            case "pressure" -> "...%s님? 하, %s 지원자 오늘 몇 번째더라.\n자기소개 해보세요. 1분 넘으면 자릅니다. 뻔한 얘기는 빼고요.".formatted(name, track);
            default -> "%s님, 반가워요! 긴장 푸시고 편하게 이야기해요 🙂\n먼저 1분 자기소개 부탁드려요.".formatted(name);
        };
    }

    // 아직 답변하지 않은 첫 번째 질문 = 현재 질문
    private InterviewTurn currentTurn(List<InterviewTurn> turns) {
        return turns.stream().filter(t -> !t.isAnswered()).findFirst().orElse(null);
    }

    // 면접 조회, 없으면 404
    private Interview findInterview(long id) {
        Interview interview = mapper.findInterview(id);
        if (interview == null) throw new ResponseStatusException(HttpStatus.NOT_FOUND, "면접을 찾을 수 없습니다.");
        return interview;
    }

    // 기술 스택 리스트 → "React,Spring" 문자열 (공백 제거, 쉼표 제거, 중복 제거)
    private static String joinStack(List<String> stack) {
        if (stack == null) return "";
        return stack.stream().map(String::strip).filter(s -> !s.isEmpty())
                .map(s -> s.replace(",", "")).distinct().collect(Collectors.joining(","));
    }

    // 문장 리스트 → 줄바꿈으로 이은 문자열 (DB 저장용)
    private static String joinLines(List<String> lines) {
        if (lines == null) return "";
        return lines.stream().filter(s -> s != null && !s.isBlank())
                .map(s -> s.strip().replace("\n", " ")).collect(Collectors.joining("\n"));
    }

    // 줄바꿈 문자열 → 문장 리스트 (응답용)
    private static List<String> splitLines(String text) {
        if (text == null || text.isBlank()) return List.of();
        return Arrays.stream(text.split("\n")).filter(s -> !s.isBlank()).toList();
    }

    // 값이 비어 있으면 기본값 사용
    private static String blankTo(String value, String fallback) {
        return value == null || value.isBlank() ? fallback : value.strip();
    }

    // 소수 첫째 자리 반올림 (6.66 → 6.7)
    private static double round1(double value) { return Math.round(value * 10) / 10.0; }
}
```

#### InterviewerAiService.java

- **역할**: 프롬프트를 조립해 Ollama에 보내는 AI 면접관. 질문 생성, 답변 채점, 최종 총평 작성 담당 (합격 판단·순서 결정은 하지 않음)

```java
package com.example.rag.service;

import java.util.List;
import java.util.Map;
import java.util.function.Supplier;
import com.example.rag.dto.ReportText;
import com.example.rag.dto.TurnEvaluation;
import com.example.rag.model.Interview;
import com.example.rag.model.InterviewTurn;
import com.example.rag.model.TurnType;
import org.springframework.ai.chat.client.ChatClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Service;
import org.springframework.web.server.ResponseStatusException;

@Service
public class InterviewerAiService {
    // ChatClient: Spring AI가 제공하는 AI 모델 호출 도구
    private final ChatClient chatClient;
    // 성격 이름(kind/strict/pressure) → 말투 설명 문장
    private final Map<String, String> personas;

    // ChatClient.Builder: application.yaml의 Ollama 설정이 적용된 빌더를 스프링이 주입
    // @Value("${...}"): application.yaml의 app.interview.persona.* 값을 문자열로 주입
    public InterviewerAiService(ChatClient.Builder builder,
                                @Value("${app.interview.persona.kind}") String kind,
                                @Value("${app.interview.persona.strict}") String strict,
                                @Value("${app.interview.persona.pressure}") String pressure) {
        this.chatClient = builder.build();
        this.personas = Map.of("kind", kind, "strict", strict, "pressure", pressure);
    }

    // ---------- 0) 모델 깨우기 ----------
    // Ollama는 처음 호출 때 모델을 메모리에 올리느라 오래 걸림 → 서버 시작 때 한 번 호출
    public void warmUp() {
        try {
            chatClient.prompt().user("안녕하세요라고만 답해.").call().content();
        } catch (RuntimeException ignored) {
            // Ollama가 꺼져 있어도 서버는 정상 시작
        }
    }

    // ---------- 1) 질문 만들기 ----------
    // 시스템 프롬프트: AI의 역할과 규칙을 정하는 지시문
    // """ """: 여러 줄 문자열(텍스트 블록), %s는 formatted()로 채울 자리
    private static final String QUESTION_SYSTEM = """
            너는 IT 회사의 개발자 기술 면접관이다.
            [면접관 성격] %s

            [지원자 정보]
            %s

            [할 일] 다음 질문 유형은 '%s'이다. %s
            - 면접관 성격의 말투가 확실히 드러나게 질문한다. (친절하면 다정하게, 깐깐하면 건조하고 날카롭게, 압박이면 퉁명스럽고 의심하듯)
            - 한국어 존댓말로 1~2문장만 출력한다. 마지막 문장은 반드시 질문이어야 한다.
            - 번호, 따옴표, "질문:" 같은 머리말 없이 면접관이 실제로 하는 말만 쓴다.
            """;

    // previous: 꼬리질문일 때 직전 질문·답변, 아니면 null
    public String askQuestion(Interview interview, TurnType type, InterviewTurn previous) {
        // %s 자리에 성격, 지원자 정보, 질문 유형, 유형별 안내 채우기
        String system = QUESTION_SYSTEM.formatted(persona(interview), profile(interview),
                type.getLabel(), type.getGuide());
        // 사용자 메시지: 꼬리질문이면 직전 질문·답변 전달
        String user = previous == null
                ? "지원자 정보를 바탕으로 질문을 하나 만들어 주세요."
                : "[직전 질문] %s\n[지원자 답변] %s".formatted(previous.getQuestion(), previous.getAnswer());

        // prompt().system().user().call().content(): 시스템·사용자 메시지로 호출 → 응답 글자 받기
        String reply = call(() -> chatClient.prompt().system(system).user(user).call().content());
        return cleanQuestion(reply);
    }

    // AI 응답 정리: "질문:" 머리말, 따옴표 제거, 여러 줄은 한 줄로, 300자 넘으면 문장 끝에서 자름
    private static String cleanQuestion(String reply) {
        String text = reply.strip().lines().map(String::strip).filter(s -> !s.isEmpty())
                .map(s -> s.replaceFirst("^(질문|Q\\d*|면접관)\\s*[:.)]\\s*", ""))   // 정규식으로 머리말 제거
                .reduce((a, b) -> a + " " + b).orElse("");                          // 줄 이어 붙이기
        text = text.replaceAll("^[\"'“”‘’]+|[\"'“”‘’]+$", "").strip();             // 앞뒤 따옴표 제거
        int limit = 300;
        if (text.length() > limit) {
            // 300자 이내에서 마지막 ?, ., ! 위치 찾기
            int cut = Math.max(text.lastIndexOf('?', limit), Math.max(text.lastIndexOf('.', limit), text.lastIndexOf('!', limit)));
            text = cut > 0 ? text.substring(0, cut + 1) : text.substring(0, limit);
        }
        return text;
    }

    // ---------- 2) 답변 채점 ----------
    // 채점 기준과 점수 가이드를 프롬프트에 명시 → 작은 모델도 일정하게 채점
    private static final String EVALUATE_SYSTEM = """
            너는 IT 회사의 개발자 기술 면접관이다.
            [면접관 성격] %s

            [지원자 정보]
            %s

            [채점 기준] 지원자의 답변을 아래 세 항목으로 각각 1~10점 정수로 평가한다.
            - logic(논리성): 질문에 맞게 순서 있게 답했는가
            - detail(구체성): 실제 경험, 기술 이름, 숫자, 결과가 들어 있는가
            - tech(기술 이해): 개념을 정확히 알고 설명하는가
            점수 가이드: "모르겠다"나 한 문장뿐이면 1~3점, 일반적인 설명만 있으면 4~6점,
            경험·이유·결과가 있으면 7~8점, 정확한 개념과 근거, 예시까지 있으면 9~10점.
            지원자의 경력 수준을 고려하되, 점수를 후하게 몰아주지 않는다.

            점수는 면접관 성격과 상관없이 위 기준대로 공정하게 매긴다.

            [작성 규칙]
            - feedback: 반드시 [면접관 성격]의 말투 그대로 한국어 존댓말 1~2문장으로 쓴다. 예시 표현을 참고한다.
              무엇이 좋았고 무엇을 고쳐야 하는지가 드러나야 한다.
            - 욕설, 외모·출신·성별에 대한 말, 인격 비하는 절대 쓰지 않는다. 비판은 답변 내용에만 한다.
            """;

    public TurnEvaluation evaluate(Interview interview, InterviewTurn turn) {
        String system = EVALUATE_SYSTEM.formatted(persona(interview), profile(interview));
        // 질문 은행 질문이면 채점 키워드 힌트 추가
        String keywords = turn.getKeywords() == null ? "" :
                "\n[좋은 답변에 들어가면 좋은 내용] " + turn.getKeywords();
        String user = """
                [질문 유형] %s
                [질문] %s%s
                [지원자 답변] %s
                """.formatted(turn.getTypeLabel(), turn.getQuestion(), keywords, turn.getAnswer());

        // entity(TurnEvaluation.class): AI에게 JSON 형식으로 답하게 하고 그 JSON을 바로 객체로 변환
        TurnEvaluation result = call(() -> chatClient.prompt().system(system).user(user)
                .call().entity(TurnEvaluation.class));
        return result.clamped();   // 점수를 1~10 범위로 보정
    }

    // ---------- 3) 최종 리포트 글 ----------
    private static final String REPORT_SYSTEM = """
            너는 방금 면접을 본 개발자 면접관이다. 면접 기록을 보고 지원자에게 줄 결과 리포트를 쓴다.
            [면접관 성격] %s

            [지원자 정보]
            %s

            [작성 규칙]
            - summary: 전체 면접에 대한 총평 2~3문장. 반드시 [면접관 성격]의 말투로 쓴다.
            - strengths: 잘한 점 2개 (각각 한 문장, 실제 답변 내용을 근거로)
            - improvements: 아쉬운 점 2개 (각각 한 문장, 어떻게 고치면 좋을지 포함)
            - betterAnswer: [가장 약했던 질문]에 대해 지원자 정보에 맞게 다시 쓴 모범 답변 (4문장 이내, 1인칭)
            - strengths, improvements, betterAnswer는 성격과 상관없이 정확하고 도움이 되게 쓴다.
            - 모두 한국어 존댓말로 쓴다. 지원자 정보에 없는 경험을 지어내지 않는다.
            - 욕설, 외모·출신·성별에 대한 말, 인격 비하는 절대 쓰지 않는다.
            """;

    public ReportText writeReport(Interview interview, List<InterviewTurn> turns, InterviewTurn weakest) {
        // 면접 기록(질문·답변·점수)을 하나의 글로 이어 붙임
        // StringBuilder: 문자열을 여러 번 이어 붙일 때 쓰는 효율적인 도구
        StringBuilder log = new StringBuilder();
        for (InterviewTurn t : turns) {
            log.append("Q%d. (%s) %s\n답변: %s\n점수: 논리 %d, 구체 %d, 기술 %d\n\n".formatted(
                    t.getTurnNo(), t.getTypeLabel(), t.getQuestion(), t.getAnswer(),
                    t.getLogicScore(), t.getDetailScore(), t.getTechScore()));
        }
        String system = REPORT_SYSTEM.formatted(persona(interview), profile(interview));
        String user = """
                [면접 기록]
                %s
                [가장 약했던 질문] %s
                [그 질문에 대한 지원자 답변] %s
                """.formatted(log, weakest.getQuestion(), weakest.getAnswer());

        // AI 응답 JSON → ReportText 객체로 변환
        return call(() -> chatClient.prompt().system(system).user(user)
                .call().entity(ReportText.class));
    }

    // ---------- 공통 ----------
    // 면접의 성격 값에 맞는 말투 설명, 없으면 kind
    private String persona(Interview interview) {
        return personas.getOrDefault(interview.getPersona(), personas.get("kind"));
    }

    // 지원자 정보를 프롬프트용 글로 정리
    private String profile(Interview interview) {
        String stack = interview.stackList().isEmpty() ? "입력 안 함" : String.join(", ", interview.stackList());
        String project = interview.getProject() == null || interview.getProject().isBlank()
                ? "입력 안 함" : interview.getProject();
        return """
                - 이름: %s
                - 지원 직무: %s
                - 경력 수준: %s
                - 기술 스택: %s
                - 대표 경험: %s""".formatted(interview.getName(), interview.getTrack(),
                interview.getLevel(), stack, project);
    }

    // AI 호출 공통 처리
    // <T>: 제네릭. String이든 TurnEvaluation이든 어떤 타입의 결과도 받을 수 있음
    // Supplier<T>: 실행하면 T를 돌려주는 함수 (호출 코드를 넘겨받아 여기서 실행)
    private <T> T call(Supplier<T> request) {
        T result;
        try {
            result = request.get();
        } catch (RuntimeException error) {
            // Ollama 연결 실패 → 503
            throw new ResponseStatusException(HttpStatus.SERVICE_UNAVAILABLE,
                    "AI 면접관 응답을 받지 못했습니다. Ollama 실행 상태를 확인하고 다시 시도해 주세요.");
        }
        if (result == null) {
            // 빈 응답 → 502 Bad Gateway
            throw new ResponseStatusException(HttpStatus.BAD_GATEWAY, "AI 응답이 비어 있습니다. 다시 시도해 주세요.");
        }
        return result;
    }
}
```

#### InterviewMapper.java

- **역할**: MyBatis로 `interviews`, `interview_turns` 테이블에 SQL 실행

```java
package com.example.rag.repository;

import java.util.List;
import com.example.rag.model.Interview;
import com.example.rag.model.InterviewTurn;
import org.apache.ibatis.annotations.*;

// DB 컬럼은 snake_case(better_answer), 자바 필드는 camelCase(betterAnswer)
// 이름이 다른 곳은 SELECT에서 AS로 맞춰 자동 매핑
@Mapper
public interface InterviewMapper {

    // ---------- 면접 ----------
    // """ """ 텍스트 블록으로 긴 SQL을 여러 줄로 작성
    @Insert("""
            INSERT INTO interviews (name, track, level, stack, project, persona, status)
            VALUES (#{name}, #{track}, #{level}, #{stack}, #{project}, #{persona}, #{status})
            """)
    @Options(useGeneratedKeys = true, keyProperty = "id")   // 생성된 id를 객체에 세팅
    int insertInterview(Interview interview);

    @Select("""
            SELECT id, name, track, level, stack, project, persona, status,
                   summary, strengths, improvements, better_answer AS betterAnswer
            FROM interviews WHERE id = #{id}
            """)
    Interview findInterview(@Param("id") long id);

    // 면접 상태 변경 (ONGOING → DONE)
    // 매개변수가 2개 이상이면 @Param으로 SQL의 #{이름}과 연결
    @Update("UPDATE interviews SET status = #{status} WHERE id = #{id}")
    int updateStatus(@Param("id") long id, @Param("status") String status);

    // AI가 쓴 총평·잘한 점·아쉬운 점·모범 답변 저장
    @Update("""
            UPDATE interviews
            SET summary = #{summary}, strengths = #{strengths},
                improvements = #{improvements}, better_answer = #{betterAnswer}
            WHERE id = #{id}
            """)
    int saveReport(Interview interview);

    // ---------- 질문 턴 ----------
    // 여러 SELECT에서 재사용하는 컬럼 목록 상수 (인터페이스 필드는 자동으로 static final)
    String TURN_COLUMNS = """
            id, interview_id AS interviewId, turn_no AS turnNo, type, question, keywords, answer,
            logic_score AS logicScore, detail_score AS detailScore, tech_score AS techScore, feedback
            """;

    // 새 질문 턴 저장 (답변·점수는 아직 비어 있음)
    @Insert("""
            INSERT INTO interview_turns (interview_id, turn_no, type, question, keywords)
            VALUES (#{interviewId}, #{turnNo}, #{type}, #{question}, #{keywords})
            """)
    @Options(useGeneratedKeys = true, keyProperty = "id")
    int insertTurn(InterviewTurn turn);

    // 면접의 질문 턴 전체를 순서대로 조회
    @Select("SELECT " + TURN_COLUMNS + " FROM interview_turns WHERE interview_id = #{interviewId} ORDER BY turn_no")
    List<InterviewTurn> findTurns(@Param("interviewId") long interviewId);

    // 답변 저장. answer IS NULL 조건 → 이미 답한 질문이면 0행 (중복 답변 방지)
    @Update("UPDATE interview_turns SET answer = #{answer} WHERE id = #{id} AND answer IS NULL")
    int saveAnswer(@Param("id") long id, @Param("answer") String answer);

    // 뒤에서 채점이 끝나면 점수와 피드백 저장
    @Update("""
            UPDATE interview_turns
            SET logic_score = #{logicScore}, detail_score = #{detailScore},
                tech_score = #{techScore}, feedback = #{feedback}
            WHERE id = #{id}
            """)
    int saveScores(InterviewTurn turn);
}
```

#### QuestionBank.java

- **역할**: CS·협업 질문을 미리 담아 둔 질문 은행. 직무에 맞고 아직 안 나온 질문을 랜덤으로 선택

```java
package com.example.rag.repository;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Set;
import com.example.rag.model.BankQuestion;
import com.example.rag.model.TurnType;
import org.springframework.stereotype.Repository;

// @Repository: 데이터를 제공하는 빈으로 등록 (DB 대신 코드 안의 목록 사용)
// CS·협업 질문을 AI 대신 여기서 고르는 이유: 질문 품질 일정 유지 + AI 호출 시간 절약
// keywords: 채점 시 AI에게 "이런 내용이 들어가면 좋은 답변"이라고 알려 주는 힌트
@Repository
public class QuestionBank {
    private static final String FRONT = "프론트엔드";
    private static final String BACK = "백엔드";

    // List.of(): 수정 불가능한 리스트
    // BankQuestion(유형, 직무(null이면 공통), 질문, 키워드)
    private final List<BankQuestion> questions = List.of(
        // ---------- CS 기초: 공통 ----------
        new BankQuestion(TurnType.CS, null, "브라우저 주소창에 URL을 입력하면 화면이 보이기까지 어떤 일이 일어나나요?",
                "DNS 조회, TCP 연결, HTTP 요청과 응답, HTML 파싱, 렌더링"),
        new BankQuestion(TurnType.CS, null, "HTTP와 HTTPS의 차이를 설명해 주세요.",
                "암호화, TLS/SSL, 인증서, 보안, 포트 443"),
        new BankQuestion(TurnType.CS, null, "GET과 POST 요청의 차이는 무엇인가요?",
                "조회와 생성, 데이터 위치(URL/본문), 멱등성, 캐시"),
        new BankQuestion(TurnType.CS, null, "프로세스와 스레드의 차이를 설명해 주세요.",
                "독립된 메모리, 메모리 공유, 생성 비용, 동시성 문제"),
        new BankQuestion(TurnType.CS, null, "배열과 연결 리스트의 차이와 각각 언제 쓰면 좋은지 말씀해 주세요.",
                "인덱스 접근 O(1), 삽입·삭제 비용, 연속된 메모리, 포인터"),
        new BankQuestion(TurnType.CS, null, "쿠키와 세션의 차이는 무엇인가요?",
                "저장 위치(클라이언트/서버), 보안, 만료, 상태 유지"),
        new BankQuestion(TurnType.CS, null, "Git에서 merge와 rebase의 차이를 설명해 주세요.",
                "병합 커밋, 히스토리 정리, 커밋 재작성, 협업 시 주의점"),
        // ---------- CS 기초: 프론트엔드 ----------
        new BankQuestion(TurnType.CS, FRONT, "CORS가 무엇이고 왜 발생하는지 설명해 주세요.",
                "동일 출처 정책, 출처(프로토콜·도메인·포트), 서버의 허용 헤더, preflight"),
        new BankQuestion(TurnType.CS, FRONT, "React에서 Virtual DOM을 사용하는 이유는 무엇인가요?",
                "실제 DOM 조작 비용, 변경 비교(diff), 필요한 부분만 업데이트"),
        new BankQuestion(TurnType.CS, FRONT, "웹 페이지 로딩 속도를 개선하는 방법을 아는 대로 말씀해 주세요.",
                "이미지 최적화, 코드 분할, 캐싱, 지연 로딩, 번들 크기 줄이기"),
        new BankQuestion(TurnType.CS, FRONT, "이벤트 버블링과 캡처링의 차이를 설명해 주세요.",
                "이벤트 전파 방향, 부모로 전파, stopPropagation, 이벤트 위임"),
        // ---------- CS 기초: 백엔드 ----------
        new BankQuestion(TurnType.CS, BACK, "데이터베이스 인덱스를 사용하는 이유와 단점을 말씀해 주세요.",
                "조회 속도 향상, B-Tree, 쓰기 성능 저하, 저장 공간"),
        new BankQuestion(TurnType.CS, BACK, "트랜잭션의 ACID 특성을 설명해 주세요.",
                "원자성, 일관성, 격리성, 지속성, 롤백"),
        new BankQuestion(TurnType.CS, BACK, "REST API를 설계할 때 지키려고 하는 원칙이 있나요?",
                "자원 중심 URL, HTTP 메서드 의미, 상태 코드, 무상태성"),
        new BankQuestion(TurnType.CS, BACK, "Spring에서 의존성 주입(DI)을 사용하는 이유는 무엇인가요?",
                "결합도 낮추기, 테스트 쉬움, 스프링 컨테이너, 빈 관리, 생성자 주입"),
        // ---------- 협업·인성 ----------
        new BankQuestion(TurnType.TEAM, null, "팀원과 의견이 크게 달랐던 경험이 있다면, 어떻게 해결했는지 말씀해 주세요.",
                "상황 설명, 상대 의견 경청, 근거로 설득, 결과와 배운 점"),
        new BankQuestion(TurnType.TEAM, null, "마감 직전에 큰 버그를 발견하면 어떻게 대응하시겠어요?",
                "영향 범위 파악, 우선순위, 팀에 빠른 공유, 임시 대응과 근본 해결"),
        new BankQuestion(TurnType.TEAM, null, "모르는 기술을 빠르게 익혀야 했던 경험이 있나요?",
                "학습 방법, 공식 문서, 작은 예제, 적용 결과"),
        new BankQuestion(TurnType.TEAM, null, "코드 리뷰에서 지적을 받았을 때 어떻게 받아들이나요?",
                "열린 태도, 이유 확인, 개선 반영, 배운 점"),
        new BankQuestion(TurnType.TEAM, null, "본인이 생각하는 좋은 개발자는 어떤 사람인가요?",
                "구체적인 기준, 본인의 노력, 예시, 성장 방향")
    );

    // 직무에 맞고 이번 면접에서 아직 안 나온 질문 중 하나를 랜덤 선택
    public BankQuestion pick(TurnType type, String track, Set<String> alreadyAsked) {
        // 조건에 맞는 질문만 걸러 수정 가능한 ArrayList로 복사 (shuffle하려면 수정 가능해야 함)
        List<BankQuestion> candidates = new ArrayList<>(questions.stream()
                .filter(q -> q.fits(type, track))                     // 유형·직무 일치
                .filter(q -> !alreadyAsked.contains(q.question()))    // 이미 나온 질문 제외
                .toList());
        if (candidates.isEmpty()) {
            throw new IllegalStateException(type.getLabel() + " 질문이 질문 은행에 없습니다.");
        }
        Collections.shuffle(candidates);   // 순서 랜덤 섞기
        return candidates.getFirst();      // 첫 번째 질문 선택
    }
}
```

#### Interview.java

- **역할**: `interviews` 테이블 한 행(지원자 정보 + 진행 상태 + 최종 리포트 글)에 대응되는 클래스

```java
package com.example.rag.model;

import java.util.Arrays;
import java.util.List;

public class Interview {
    // 면접 상태 상수 (문자열 오타 방지)
    public static final String ONGOING = "ONGOING";   // 진행 중
    public static final String DONE = "DONE";         // 종료

    private long id;
    private String name;           // 지원자 이름
    private String track;          // 지원 직무
    private String level;          // 경력 수준
    private String stack;          // "React,Spring,MySQL"처럼 쉼표로 이어서 저장
    private String project;        // 대표 경험
    private String persona;        // 면접관 성격 (kind, strict, pressure)
    private String status;         // ONGOING / DONE
    private String summary;        // AI 총평
    private String strengths;      // 잘한 점 (줄바꿈으로 이어서 저장)
    private String improvements;   // 아쉬운 점 (줄바꿈으로 이어서 저장)
    private String betterAnswer;   // 모범 답변

    // MyBatis가 조회 결과를 담을 때 쓰는 기본 생성자
    public Interview() { }

    // 저장된 "React,Spring" 문자열 → ["React", "Spring"] 리스트로 변환
    // String::strip: 각 항목 공백 제거 (메서드 참조 문법)
    public List<String> stackList() {
        if (stack == null || stack.isBlank()) return List.of();
        return Arrays.stream(stack.split(",")).map(String::strip).filter(s -> !s.isEmpty()).toList();
    }

    // getter / setter: MyBatis 매핑, JSON 변환에 사용
    public long getId() { return id; }
    public void setId(long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getTrack() { return track; }
    public void setTrack(String track) { this.track = track; }
    public String getLevel() { return level; }
    public void setLevel(String level) { this.level = level; }
    public String getStack() { return stack; }
    public void setStack(String stack) { this.stack = stack; }
    public String getProject() { return project; }
    public void setProject(String project) { this.project = project; }
    public String getPersona() { return persona; }
    public void setPersona(String persona) { this.persona = persona; }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
    public String getSummary() { return summary; }
    public void setSummary(String summary) { this.summary = summary; }
    public String getStrengths() { return strengths; }
    public void setStrengths(String strengths) { this.strengths = strengths; }
    public String getImprovements() { return improvements; }
    public void setImprovements(String improvements) { this.improvements = improvements; }
    public String getBetterAnswer() { return betterAnswer; }
    public void setBetterAnswer(String betterAnswer) { this.betterAnswer = betterAnswer; }
}
```

#### InterviewTurn.java

- **역할**: `interview_turns` 테이블 한 행(면접 속 질문 하나 + 답변 + 점수 + 피드백)에 대응되는 클래스

```java
package com.example.rag.model;

// 답변 전에는 answer와 점수가 비어 있음(null)
public class InterviewTurn {
    private long id;
    private long interviewId;      // 어느 면접의 질문인지 (FK)
    private int turnNo;            // 몇 번째 질문인지 (1~5)
    private String type;           // 질문 유형 이름 (INTRO, PROJECT, TECH, CS, TEAM)
    private String question;       // 질문 내용
    private String keywords;       // 질문 은행 질문의 채점 키워드
    private String answer;         // 지원자 답변
    // int가 아닌 Integer: 아직 채점 전이면 null이어야 하므로 (int는 null 불가)
    private Integer logicScore;    // 논리성 1~10
    private Integer detailScore;   // 구체성 1~10
    private Integer techScore;     // 기술 이해 1~10
    private String feedback;       // AI 피드백

    public InterviewTurn() { }

    // 새 질문을 만들 때 사용하는 생성자
    public InterviewTurn(long interviewId, int turnNo, TurnType type, String question, String keywords) {
        this.interviewId = interviewId;
        this.turnNo = turnNo;
        this.type = type.name();   // enum → 문자열 ("CS")로 저장
        this.question = question;
        this.keywords = keywords;
    }

    // 문자열 type → enum으로 변환
    public TurnType turnType() { return TurnType.valueOf(type); }
    // 화면 표시용 한글 이름 ("CS 기초"), getter라 JSON에 typeLabel로 포함
    public String getTypeLabel() { return turnType().getLabel(); }
    // 답변 여부
    public boolean isAnswered() { return answer != null; }
    // 채점 완료 여부 (세 점수 모두 있어야 true)
    public boolean isScored() { return logicScore != null && detailScore != null && techScore != null; }

    // 세 점수 평균 (채점 전이면 0)
    public double average() {
        if (!isScored()) return 0;
        return (logicScore + detailScore + techScore) / 3.0;
    }

    // getter / setter
    public long getId() { return id; }
    public void setId(long id) { this.id = id; }
    public long getInterviewId() { return interviewId; }
    public void setInterviewId(long interviewId) { this.interviewId = interviewId; }
    public int getTurnNo() { return turnNo; }
    public void setTurnNo(int turnNo) { this.turnNo = turnNo; }
    public String getType() { return type; }
    public void setType(String type) { this.type = type; }
    public String getQuestion() { return question; }
    public void setQuestion(String question) { this.question = question; }
    public String getKeywords() { return keywords; }
    public void setKeywords(String keywords) { this.keywords = keywords; }
    public String getAnswer() { return answer; }
    public void setAnswer(String answer) { this.answer = answer; }
    public Integer getLogicScore() { return logicScore; }
    public void setLogicScore(Integer logicScore) { this.logicScore = logicScore; }
    public Integer getDetailScore() { return detailScore; }
    public void setDetailScore(Integer detailScore) { this.detailScore = detailScore; }
    public Integer getTechScore() { return techScore; }
    public void setTechScore(Integer techScore) { this.techScore = techScore; }
    public String getFeedback() { return feedback; }
    public void setFeedback(String feedback) { this.feedback = feedback; }
}
```

#### TurnType.java

- **역할**: 면접 질문 순서와 유형을 정의하는 enum. 순서는 코드가 정하고 내용은 AI나 질문 은행이 채움

```java
package com.example.rag.model;

// enum: 정해진 값 몇 개만 가질 수 있는 타입
// 선언 순서 = 질문 순서 (1번 INTRO → 5번 TEAM)
public enum TurnType {
    // (한글 이름, 질문 은행 사용 여부, AI 질문 생성 안내)
    INTRO("자기소개", false,
            "지원자에게 자기소개를 요청한다."),
    PROJECT("프로젝트 경험", false,
            "지원자의 대표 경험이나 기술 스택을 바탕으로, 프로젝트에서 맡은 역할이나 가장 어려웠던 문제를 묻는다."),
    TECH("기술 꼬리질문", false,
            "직전 답변에 나온 기술이나 결정을 하나 골라, 왜 그렇게 했는지 또는 어떻게 동작하는지 파고드는 꼬리질문을 한다."),
    CS("CS 기초", true, ""),      // 질문 은행에서 선택 → 안내 불필요
    TEAM("협업·인성", true, "");

    private final String label;       // 화면 표시용 이름
    private final boolean fromBank;   // true면 질문 은행에서 고름
    private final String guide;       // AI가 질문을 만들 때 쓰는 안내

    // enum 생성자 (위 괄호 값으로 자동 호출)
    TurnType(String label, boolean fromBank, String guide) {
        this.label = label;
        this.fromBank = fromBank;
        this.guide = guide;
    }

    public String getLabel() { return label; }
    public boolean isFromBank() { return fromBank; }
    public String getGuide() { return guide; }

    // 전체 질문 수 (values(): enum 값 배열 → 길이 5)
    public static final int TOTAL = values().length;

    // 1번부터 시작하는 번호로 유형 찾기 (배열은 0부터라 -1)
    public static TurnType of(int turnNo) { return values()[turnNo - 1]; }
}
```

#### BankQuestion.java

- **역할**: 질문 은행에 들어가는 질문 하나 (유형, 직무, 질문, 채점 키워드)

```java
package com.example.rag.model;

// record: 필드·생성자·getter 자동 생성되는 불변 데이터 클래스
// track이 null이면 모든 직무 공통 질문
public record BankQuestion(TurnType type, String track, String question, String keywords) {

    // 원하는 유형·직무에 맞는 질문인지 확인
    public boolean fits(TurnType wantedType, String wantedTrack) {
        if (type != wantedType) return false;   // 유형이 다르면 X
        if (track == null) return true;         // 공통 질문이면 O
        // 직무가 같거나, 지원자가 풀스택이면 프론트·백엔드 질문 모두 O
        return track.equals(wantedTrack) || "풀스택".equals(wantedTrack);
    }
}
```

#### InterviewStartRequest.java

- **역할**: 면접 시작 화면에서 받는 지원자 정보 형식과 검증 규칙

```java
package com.example.rag.dto;

import java.util.List;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Pattern;
import jakarta.validation.constraints.Size;

public record InterviewStartRequest(
        @NotBlank(message = "이름을 입력해 주세요.")
        @Size(max = 20, message = "이름은 20자 이내로 입력해 주세요.")
        String name,

        // @Pattern: 정규식에 맞는 값만 허용 (| = 또는)
        // 화면에서 고른 값 외에 이상한 값이 들어오는 것 차단
        @NotBlank(message = "지원 직무를 골라 주세요.")
        @Pattern(regexp = "프론트엔드|백엔드|풀스택", message = "지원 직무를 다시 골라 주세요.")
        String track,

        @NotBlank(message = "경력 수준을 골라 주세요.")
        @Pattern(regexp = "고졸 신입|대졸 신입|경력 1~3년", message = "경력 수준을 다시 골라 주세요.")
        String level,

        // 리스트 자체는 최대 10개, 리스트 안 각 문자열은 최대 20자
        // List<@Size(...) String>: 리스트 요소 하나하나에 검증 적용
        @Size(max = 10, message = "기술 스택은 10개까지 입력할 수 있어요.")
        List<@Size(max = 20, message = "기술 스택은 하나에 20자 이내로 입력해 주세요.") String> stack,

        // 선택 입력 (@NotBlank 없음)
        @Size(max = 200, message = "대표 경험은 200자 이내로 입력해 주세요.")
        String project,

        @NotBlank(message = "면접관을 골라 주세요.")
        @Pattern(regexp = "kind|strict|pressure", message = "면접관을 다시 골라 주세요.")
        String persona) {
}
```

#### AnswerRequest.java

- **역할**: 면접 답변 제출 요청 형식과 검증 규칙

```java
package com.example.rag.dto;

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;

// 요청 JSON {"content": "..."} → content 필드
public record AnswerRequest(
        @NotBlank(message = "답변을 입력해 주세요.")
        @Size(max = 500, message = "답변은 500자 이내로 입력해 주세요.")   // 질문(200자)보다 길게 허용
        String content) {
}
```

#### InterviewState.java

- **역할**: 지금 화면에 띄울 질문 정보 (면접 번호, 몇 번째, 전체 개수, 유형, 질문)

```java
package com.example.rag.dto;

import com.example.rag.model.InterviewTurn;
import com.example.rag.model.TurnType;

public record InterviewState(long interviewId, int turnNo, int totalTurns, String typeLabel, String question) {

    // 정적 팩토리 메서드: InterviewTurn → InterviewState로 변환
    // new 대신 InterviewState.of(turn)으로 간단히 생성
    public static InterviewState of(InterviewTurn turn) {
        return new InterviewState(turn.getInterviewId(), turn.getTurnNo(), TurnType.TOTAL,
                turn.getTypeLabel(), turn.getQuestion());
    }
}
```

#### InterviewDetail.java

- **역할**: 새로고침 후 이어하기, 기록 보기에 쓰는 면접 전체 정보

```java
package com.example.rag.dto;

import java.util.List;
import com.example.rag.model.Interview;
import com.example.rag.model.InterviewTurn;

public record InterviewDetail(
        Interview interview,          // 면접 정보 (지원자, 상태, 리포트)
        List<InterviewTurn> turns,    // 지금까지의 질문·답변 전체
        InterviewState current) {     // 진행 중인 질문 (끝났으면 null)
}
```

#### TurnResult.java

- **역할**: 답변 제출 후 응답. 다음 질문만 담고, 점수·피드백은 결과 리포트에서 한 번에 확인

```java
package com.example.rag.dto;

public record TurnResult(
        int turnNo,            // 방금 답변한 질문 번호
        boolean finished,      // 면접이 끝났는지
        InterviewState next) { // 다음 질문 (끝났으면 null)
}
```

#### TurnEvaluation.java

- **역할**: AI가 답변 하나를 채점해서 돌려주는 JSON 형식

```java
package com.example.rag.dto;

// Spring AI의 entity(TurnEvaluation.class)가 AI의 JSON 응답을 이 객체로 바로 변환
// 필드 이름이 곧 AI에게 요구하는 JSON 키 이름
public record TurnEvaluation(
        int logic,         // 논리성 1~10
        int detail,        // 구체성 1~10
        int tech,          // 기술 이해 1~10
        String feedback) { // 한두 문장 피드백

    // 작은 모델이 0점이나 15점처럼 범위를 벗어난 점수를 줄 때 대비해 1~10으로 보정
    public TurnEvaluation clamped() {
        return new TurnEvaluation(clamp(logic), clamp(detail), clamp(tech), feedback);
    }

    // 1보다 작으면 1, 10보다 크면 10
    private static int clamp(int score) { return Math.max(1, Math.min(10, score)); }
}
```

#### ReportText.java

- **역할**: AI가 최종 리포트용으로 써 주는 글 (점수 계산·합격 판정은 코드가 따로 처리)

```java
package com.example.rag.dto;

import java.util.List;

// AI 응답 JSON → 이 객체로 변환
public record ReportText(
        String summary,             // 총평
        List<String> strengths,     // 잘한 점
        List<String> improvements,  // 아쉬운 점
        String betterAnswer) {      // 가장 약했던 질문의 모범 답변
}
```

#### InterviewReport.java

- **역할**: 결과 화면에 보내는 최종 리포트. 숫자는 코드가, 글은 AI가 작성. 준비 안 된 값은 null

```java
package com.example.rag.dto;

import java.util.List;
import com.example.rag.model.InterviewTurn;

// 결과 화면을 바로 열고 준비되는 대로 채우기 위해 아직 안 된 값은 null로 전송
// int/boolean 대신 Integer/Double/Boolean: null을 담을 수 있게
public record InterviewReport(
        long interviewId,
        String name,
        String track,
        String level,
        String persona,

        // 진행 상황
        boolean ready,           // 채점 + 총평 모두 끝났는지
        int scoredCount,         // 채점 끝난 질문 수
        int totalTurns,          // 전체 질문 수
        boolean summaryReady,    // 총평 글 준비 여부
        String error,            // 결과 정리 실패 메시지

        // 점수 (모든 채점이 끝나야 채워짐)
        Double logicAvg,
        Double detailAvg,
        Double techAvg,
        Integer totalScore,      // 100점 만점
        Boolean passed,          // 합격 여부

        // 총평 (summaryReady일 때 채워짐)
        String summary,
        List<String> strengths,
        List<String> improvements,
        String weakestQuestion,  // 가장 약했던 질문
        String betterAnswer,     // 그 질문의 모범 답변

        List<InterviewTurn> turns) {   // 질문별 답변·점수·피드백
}
```

## 6. SQL 복습

### 6-1. DB·테이블 생성

```sql
CREATE DATABASE IF NOT EXISTS sql_review;   -- DB가 없을 때만 생성
USE sql_review;                             -- 이후 명령을 이 DB에서 실행

CREATE TABLE users (
    id INT PRIMARY KEY,       -- 기본키: 행을 구분하는 고유 값 (중복·NULL 불가)
    name VARCHAR(20)          -- 최대 20자 문자열
);

CREATE TABLE posts (
    id INT PRIMARY KEY,
    title VARCHAR(100),
    user_id INT,                                  -- 글쓴이 id
    FOREIGN KEY (user_id) REFERENCES users(id)    -- 외래키: users 테이블의 id와 연결
);
```

- **PRIMARY KEY**: 테이블 안에서 각 행을 구분하는 값
- **FOREIGN KEY**: 다른 테이블의 기본키를 참조하는 값. `users`에 없는 id는 `posts.user_id`에 넣을 수 없는 구조
- 테이블 관계: 사용자 1명이 게시글 여러 개를 쓰는 **1:N 관계**

### 6-2. 데이터 추가 (INSERT)

```sql
INSERT INTO users (id, name) VALUES (1, "민수");
INSERT INTO users (id, name) VALUES (2, "지우");

INSERT INTO posts (id, title, user_id) VALUES (1, "첫 인사", 1);
INSERT INTO posts (id, title, user_id) VALUES (2, "급식 질문", 2);
INSERT INTO posts (id, title, user_id) VALUES (3, "동아리 질문", 1);
```

- `INSERT INTO 테이블 (컬럼들) VALUES (값들)`: 컬럼 순서대로 값 추가
- 외래키 때문에 `users`를 먼저 넣어야 `posts`를 넣을 수 있는 순서

|users.id|name|
|---|---|
|1|민수|
|2|지우|

|posts.id|title|user_id|
|---|---|---|
|1|첫 인사|1|
|2|급식 질문|2|
|3|동아리 질문|1|

### 6-3. 데이터 수정 (UPDATE)

```sql
UPDATE posts SET title = "급식 시간 질문" WHERE id = 2;
```

- `UPDATE 테이블 SET 컬럼 = 값 WHERE 조건`: 조건에 맞는 행의 값 변경
- `WHERE`를 빼면 모든 행이 바뀌므로 주의

### 6-4. 데이터 삭제 (DELETE)

```sql
DELETE FROM posts WHERE id = 3;
```

- `DELETE FROM 테이블 WHERE 조건`: 조건에 맞는 행 삭제
- `WHERE`를 빼면 모든 행이 삭제되므로 주의

수정·삭제 후 결과

|id|title|user_id|
|---|---|---|
|1|첫 인사|1|
|2|급식 시간 질문|2|

### 6-5. INNER JOIN

```sql
SELECT p.id, p.title, u.name AS author
FROM posts p
INNER JOIN users u ON p.user_id = u.id;
```

- **INNER JOIN**: 두 테이블에서 조건(`ON`)이 맞는 행끼리만 합쳐서 조회
- `posts p`, `users u`: 테이블 별칭. `p.id`처럼 짧게 사용
- `ON p.user_id = u.id`: 게시글의 글쓴이 id와 사용자 id가 같은 행끼리 연결
- `u.name AS author`: 결과에서 `name` 컬럼 이름을 `author`로 표시 (컬럼 별칭)

결과

|id|title|author|
|---|---|---|
|1|첫 인사|민수|
|2|급식 시간 질문|지우|
