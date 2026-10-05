# Validation과 예외처리

### Bean Validation
- 애노테이션으로 제약조건을 선언해두면 그에 대한 검증을 수행해주는 Java 표준 명세(Jakarta Bean Validation)
- 검증 로직을 별도로 구현하지 않아도 되며, 어떤 제약조건을 가지는지 선언만으로 드러나므로 가독성이 높아짐
- Spring Boot에서 사용하려면 spring-boot-starter-validation 의존성을 추가해야 함
  - Web 관련 Starter에 포함되어 있지 않으므로, 별도로 추가해줘야 함
```
build.gradle

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-validation'
}
```
- 자주 사용하는 제약조건 애노테이션
  - @NotNull: null이 아니어야 함
  - @NotEmpty: null이 아니어야 하며, 길이나 크기가 0이 아니어야 함
  - @NotBlank: null이 아니어야 하며, 공백 문자를 제외한 문자가 하나 이상 존재해야 함
  - @Size(min, max): 문자열의 길이 또는 컬렉션의 크기 범위를 지정함
  - @Min / @Max: 숫자의 최솟값과 최댓값을 지정함
  - @Positive / @Negative: 양수 또는 음수여야 함
  - @Email: 이메일 형식이어야 함
  - @Pattern(regexp): 지정한 정규표현식과 일치해야 함
  - @Past / @Future: 과거 또는 미래의 일시여야 함

<br>

### @Valid와 @Validated
- @Valid
  - Jakarta Bean Validation에서 제공하는 표준 애노테이션
  - 검증 대상 객체를 지정할 때 사용하며, 중첩된 객체까지 함께 검증하도록 지시함
- @Validated
  - Spring에서 제공하는 애노테이션으로써, 검증 Group을 지정할 수 있음
  - 클래스에 선언하는 경우 AOP 프록시를 통한 메서드 검증이 활성화됨
  - Service 계층처럼 Controller가 아닌 곳에서 메서드 검증을 수행할 때 사용함

<br>

### 검증 방식에 따라 발생하는 예외
- 검증에 실패했을 때 발생하는 예외는 하나가 아니며, 제약조건을 어디에 선언했는지에 따라 달라짐
- @RequestBody 또는 @ModelAttribute 객체에 @Valid를 선언한 경우
  - MethodArgumentNotValidException 발생. 400 Bad Request로 응답됨
  - BindingResult를 통해 어떤 필드가 어떤 이유로 실패했는지 바로 확인할 수 있음
- 메서드 파라미터에 제약조건 애노테이션을 직접 선언한 경우
  - Spring Framework 6.1부터 Spring MVC가 AOP 없이 자체적으로 메서드 검증을 수행함
  - HandlerMethodValidationException 발생. 400 Bad Request로 응답됨
  - 별도 설정이나 @Validated 선언 없이 동작함
- Controller 클래스에 @Validated를 선언한 경우
  - 내장 메서드 검증 대신 AOP 프록시 기반 검증이 수행됨
  - ConstraintViolationException 발생. 상태코드가 매핑되어 있지 않으므로, 별도로 처리하지 않으면 500 Internal Server Error로 응답됨
  - 클라이언트의 입력값 오류인데 서버 오류로 응답되므로, Controller에는 클래스 레벨 @Validated를 선언하지 않아야 함
- 요청 값 자체를 바인딩할 수 없는 경우에 대한 내용은 [File Upload](/file-upload/file-upload.md) 참고
- 반환값 검증에 실패한 경우에는 서버 구현상의 오류이므로 500 Internal Server Error로 응답됨
```
@RestController
public class MemberController {

    // 검증 실패 시 HandlerMethodValidationException -> 400
    @GetMapping("/members")
    public List<MemberResponse> search(@RequestParam @NotBlank String keyword) {
        ...
    }

    // 검증 실패 시 MethodArgumentNotValidException -> 400
    @PostMapping("/members")
    public MemberResponse create(@RequestBody @Valid MemberCreateRequest request) {
        ...
    }

}
```

<br>

### 제약조건을 어디에 선언할 것인가
- 검증 대상이 한두개인 단순 형식 검증이라면, 메서드 파라미터에 직접 선언해도 무방함
- 아래의 경우에는 DTO나 record로 분리하여 필드에 선언하는 것이 좋음
  - 검증 대상이 많아 메서드 시그니처가 길어지는 경우
  - 같은 검증 규칙이 여러 Endpoint에서 반복되는 경우
  - 에러 응답에 어떤 필드가 잘못되었는지 담아야 하는 경우
- 계층별로 어떤 성격의 검증을 수행해야 하는지는 [Domain Model - 유효성 검사 위치](/layered-architecture/domain-model.md#유효성-검사-위치) 참고
  - Controller를 거치지 않는 호출 경로에서는 Bean Validation이 수행되지 않으므로, 도메인 불변식은 반드시 도메인 객체 자체에서도 강제해야 함

<br>

### 예외처리
- @ExceptionHandler
  - Controller 내부에 선언하여, 해당 Controller에서 발생한 예외를 처리함
- @RestControllerAdvice / @ControllerAdvice
  - 여러 Controller에서 발생하는 예외를 한곳에서 처리할 수 있도록 해주는 애노테이션
  - @RestControllerAdvice는 @ControllerAdvice와 @ResponseBody를 합쳐놓은 애노테이션
- 검증 실패 시 발생하는 예외는 Controller 메서드의 시그니처에 따라 달라지므로, MethodArgumentNotValidException과 HandlerMethodValidationException을 모두 처리해줘야 함
```
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handle(MethodArgumentNotValidException e) {
        ...
    }

    @ExceptionHandler(HandlerMethodValidationException.class)
    public ResponseEntity<ErrorResponse> handle(HandlerMethodValidationException e) {
        ...
    }

}
```
- 두 예외에서 오류 내용을 꺼내는 방법이 서로 다름
  - MethodArgumentNotValidException: BindingResult를 통해 필드별 오류를 바로 확인할 수 있음
  - HandlerMethodValidationException: getAllValidationResults() 또는 Visitor를 통해 확인해야 하므로, 동일한 에러 응답 형태로 맞추려면 별도 변환 작업이 필요함

<br>

### ProblemDetail
- HTTP API의 오류 응답 형식을 정한 표준(RFC 9457, 이전 RFC 7807)을 구현한 클래스로써, Spring Framework 6.0부터 제공됨
  - 오류 응답 DTO를 프로젝트마다 따로 만들지 않고, 표준 형식으로 통일할 수 있음
  - Content-Type은 application/problem+json으로 응답됨
- 주요 필드
  - type: 오류 종류를 나타내는 URI. 지정하지 않으면 about:blank
  - title: 오류 종류의 짧은 설명. 기본값은 상태 코드의 이름(Not Found 등)
  - status: HTTP 상태 코드
  - detail: 이번 오류에 대한 구체적인 설명
  - instance: 오류가 발생한 요청 경로
  - 그 외 필요한 값은 setProperty()로 추가할 수 있음
```
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "suite 3 not found",
  "instance": "/api/suites/3"
}
```
- spring.mvc.problemdetails.enabled=true로 지정하면, Spring MVC가 직접 처리하는 예외(검증 실패, 지원하지 않는 메서드 등)도 ProblemDetail 형식으로 응답함
- @RestControllerAdvice의 @ExceptionHandler에서 ProblemDetail을 반환하면, 직접 만든 예외도 같은 형식으로 응답할 수 있음
  - 응답 헤더를 함께 지정해야 할 때는 ResponseEntity<ProblemDetail>로 반환함
```
@RestControllerAdvice
class ApiExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    ProblemDetail handleNotFound(NotFoundException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    }

    @ExceptionHandler(TooManyLoginAttemptsException.class)
    ResponseEntity<ProblemDetail> handleTooManyLoginAttempts(TooManyLoginAttemptsException ex) {
        return ResponseEntity.status(HttpStatus.TOO_MANY_REQUESTS)
                .header(HttpHeaders.RETRY_AFTER, String.valueOf(ex.retryAfter().toSeconds()))
                .body(ProblemDetail.forStatusAndDetail(HttpStatus.TOO_MANY_REQUESTS, ex.getMessage()));
    }

    @ExceptionHandler(DataIntegrityViolationException.class)
    ProblemDetail handleDataIntegrityViolation(DataIntegrityViolationException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, "conflicts with an existing item");
    }

}
```
- 예외와 상태 코드를 짝지을 때 고려할 점
  - 이름 중복을 미리 검사해도, 동시에 등록하는 두 요청은 검사를 함께 통과할 수 있음. DB의 유니크 제약조건으로 막고, 그 위반(DataIntegrityViolationException)을 409 Conflict로 응답함
  - DB 예외 메시지에는 테이블과 제약조건 이름이 들어있으므로, detail에 그대로 담지 않음
  - 인증 실패처럼 사유를 구분하면 안 되는 경우에는 여러 예외를 같은 응답으로 처리함
- 상태 코드의 의미는 [HTTP 상태코드](/http/http-status-code.md) 참고

<br>

#### 참고
- Spring Framework Reference Documentation <Validation> - https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-validation.html
- Spring Framework Reference Documentation <Java Bean Validation> - https://docs.spring.io/spring-framework/reference/core/validation/beanvalidation.html
- Spring Framework Reference Documentation <Error Responses> - https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-ann-rest-exceptions.html
- RFC 9457 <Problem Details for HTTP APIs> - https://www.rfc-editor.org/rfc/rfc9457

#### 배워가는 것들
- 검증에 실패했을 때 발생하는 예외가 하나가 아니라는 것을 알게 되었다. 제약조건을 어디에 선언했는지에 따라 달라지므로, 예외처리를 구성할 때 양쪽을 모두 다뤄야 한다.
- Controller에 클래스 레벨 @Validated를 붙이면 오히려 500이 응답된다는 점이 인상적이었다. 오래된 자료를 그대로 따라 하면 입력값 오류를 서버 오류로 내보내게 되므로, 현재 Version의 동작을 확인하고 적용해야 한다.
- Bean Validation은 Controller를 거치는 경로에서만 동작한다는 한계를 인지할 수 있었다. 반드시 지켜져야 하는 규칙이라면 애노테이션에만 의존하지 말고 도메인 객체에서도 함께 강제해야 할 것이다.
- 오류 응답 형식을 직접 설계하지 않아도 ProblemDetail이라는 표준이 있다는 것을 알게 되었다. 형식이 통일되면 화면이나 다른 서비스가 오류를 한 가지 방식으로 처리할 수 있다.
