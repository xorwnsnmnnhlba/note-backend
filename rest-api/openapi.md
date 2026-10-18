# API 문서화(OpenAPI)

### API 문서가 필요한 이유
- API를 사용하는 쪽(프론트엔드, 다른 서비스, CI 스크립트)은 경로, 요청 형식, 응답 형식, 오류 응답을 알아야 함
- README나 위키에 직접 적은 문서는 코드가 바뀌어도 함께 고쳐지지 않아, 시간이 지나면 실제 API와 어긋남
- 그래서 코드에서 문서를 자동으로 만들거나, 문서와 코드가 일치하는지 테스트로 확인하는 방식을 사용함

<br>

### OpenAPI Specification
- HTTP API를 기술하기 위한 표준 명세로써, 이전 이름은 Swagger Specification임
  - JSON 또는 YAML로 작성하며, 현재 3.x 버전이 사용됨
- 주요 구성
  - info: 제목, 버전, 설명
  - paths: 경로와 메서드별 파라미터, 요청 본문, 응답
  - components: 여러 곳에서 재사용하는 스키마(schemas), 인증 방식(securitySchemes) 등
  - tags: API 묶음
  - security: 요청에 필요한 인증
```
openapi: 3.1.0
info:
  title: Sentinel API
  version: v1
paths:
  /api/runs/{runId}:
    get:
      tags: [실행]
      summary: 실행 결과 조회
      parameters:
        - name: runId
          in: path
          required: true
          schema: { type: integer, format: int64 }
      responses:
        "200":
          content:
            application/json:
              schema: { $ref: "#/components/schemas/TestRunDetail" }
```
- 표준 형식이므로 하나의 문서로 여러 도구를 활용할 수 있음
  - Swagger UI, Redoc: 문서를 웹 화면으로 보여줌
  - OpenAPI Generator: Client, Server 코드를 생성함
  - Postman 등: 문서를 가져와 요청 목록을 만듦
- 문서를 먼저 작성하고 코드를 맞추는 방식(Design-First)과, 코드에서 문서를 생성하는 방식(Code-First)이 있음

<br>

### springdoc-openapi
- Spring MVC의 Controller, 요청·응답 타입, Bean Validation 애노테이션을 분석하여 OpenAPI 문서를 자동으로 생성해주는 라이브러리
  - Spring Boot 4.x(Spring Framework 7)에는 springdoc-openapi 3.x를 사용함
  - Spring Boot가 버전을 관리하지 않으므로 직접 명시해야 함
- 스타터 종류
  - springdoc-openapi-starter-webmvc-ui: 문서(JSON/YAML)와 Swagger UI 화면을 함께 제공함
  - springdoc-openapi-starter-webmvc-api: 문서만 제공함. 화면을 직접 만들거나 다른 도구로 볼 때 사용함
```
build.gradle

dependencies {
    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:3.1.1'
}
```
- 기본 경로
  - /v3/api-docs: OpenAPI 문서(JSON), /v3/api-docs.yaml: YAML
  - /swagger-ui.html: Swagger UI
- 주요 설정
```
springdoc:
  api-docs:
    path: /api/openapi
  writer-with-order-by-keys: true
  default-produces-media-type: application/json
```
  - api-docs.path: 문서 경로 변경
  - writer-with-order-by-keys: 문서의 키를 정렬하여, 같은 코드면 항상 같은 문서가 나오도록 함
  - default-produces-media-type: 응답 형식을 지정하지 않은 API의 응답 형식을 */* 대신 표시할 형식

<br>

### 문서 내용 보강하기
- 자동 생성만으로는 경로와 타입만 나오므로, 설명과 묶음은 애노테이션(io.swagger.v3.oas.annotations)으로 보강함
  - @Tag: Controller의 API 묶음 이름과 설명
  - @Operation: API별 요약(summary)과 설명(description)
  - @Parameter, @Schema: 파라미터와 필드 설명, 예시 값
  - @ApiResponse: 상태 코드별 응답 설명
  - @SecurityRequirements: 인증이 필요 없는 API 표시(빈 값으로 지정)
- 전체 문서의 정보와 인증 방식은 OpenAPI Bean으로 지정함
```
@Bean
OpenAPI sentinelOpenApi() {
    return new OpenAPI()
            .info(new Info().title("Sentinel API").version("v1").description("..."))
            .components(new Components().addSecuritySchemes("bearer",
                    new SecurityScheme().type(SecurityScheme.Type.HTTP).scheme("bearer")))
            .addSecurityItem(new SecurityRequirement().addList("bearer"));
}


@Tag(name = "실행")
@RestController
@RequestMapping("/api/runs")
class RunController {

}
```
- Javadoc을 그대로 문서 설명으로 사용하기
  - 애노테이션으로 설명을 따로 적으면 Javadoc과 내용이 이중으로 관리되어 서로 어긋나기 쉬움
  - therapi-runtime-javadoc을 추가하면 컴파일 시점에 Javadoc을 저장해두고, springdoc이 이를 읽어 Controller 메서드와 record 필드의 설명으로 사용함
```
dependencies {
    implementation 'com.github.therapi:therapi-runtime-javadoc:0.15.0'
    annotationProcessor 'com.github.therapi:therapi-runtime-javadoc-scribe:0.15.0'
}
```

<br>

### 문서 노출 범위
- API 문서는 내부 경로, 요청 형식, 인증 방식 등 공격에 도움이 되는 정보를 담고 있음
  - springdoc은 문서 경로가 켜져 있으면 기동 시 운영 환경에서는 끄라는 경고를 남김
- 선택지
  - 운영에서 끔: springdoc.api-docs.enabled=false, springdoc.swagger-ui.enabled=false
  - 다른 API와 같이 인증된 사용자에게만 제공함
- Swagger UI는 인라인 스크립트·스타일을 사용하므로, 엄격한 CSP를 적용한 서비스에서는 예외를 두어야 함. 관련 내용은 [웹 취약점과 방어](/spring-security/web-vulnerability.md) 참고
  - 예외를 두고 싶지 않다면 문서(JSON)만 제공하고, 화면은 서비스의 디자인으로 직접 그리는 방법도 있음
  - 이때 설명 문자열을 HTML로 삽입하지 않고 텍스트로만 출력해야 XSS를 피할 수 있음

<br>

### 테스트 기반 문서화(Spring REST Docs)와 비교
- springdoc-openapi
  - Controller 코드만으로 문서가 만들어져 도입이 쉬움
  - 실제 동작이 아니라 선언(타입, 애노테이션)을 기반으로 하므로, 설명과 실제 응답이 다를 수 있음
- Spring REST Docs
  - 테스트(MockMvc 등)를 실행하며 실제 요청·응답을 기록하여 문서를 만듦
  - 테스트가 통과해야 문서가 만들어지므로 문서의 정확성이 보장되지만, 테스트마다 문서화 코드를 작성해야 함
  - restdocs-api-spec 같은 확장으로 OpenAPI 형식으로 내보낼 수도 있음

<br>

#### 참고
- OpenAPI Specification - https://spec.openapis.org/oas/latest.html
- springdoc-openapi - https://springdoc.org/
- therapi-runtime-javadoc - https://github.com/dnault/therapi-runtime-javadoc
- Spring REST Docs - https://docs.spring.io/spring-restdocs/docs/current/reference/htmlsingle/

#### 배워가는 것들
- OpenAPI라는 표준 형식 하나로 문서 화면, 코드 생성, 요청 도구까지 연결된다는 것을 알게 되었다.
- 설명을 애노테이션에 따로 적지 않고 Javadoc을 그대로 문서로 쓰는 방법이 인상적이었다. 같은 내용을 두 곳에 관리하지 않아야 문서가 코드와 어긋나지 않는다.
- API 문서도 공격에 도움이 되는 정보라 노출 범위를 정해야 한다는 점을 배웠다. 편의를 위해 열어두는 기능일수록 운영 환경에서 어떻게 할지 먼저 정해야겠다.
