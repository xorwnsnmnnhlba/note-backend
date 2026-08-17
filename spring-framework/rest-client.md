# RestClient

### Spring의 HTTP Client
- 요청을 받아서 처리하는 서버 관점이 아니라, 다른 서버의 API를 호출하는 클라이언트 관점에서 사용하는 기능
  - 외부 서비스 연동, 마이크로서비스 간 통신 등을 수행할 때 사용함
- Spring Framework는 REST 호출을 위해 아래 네 가지 선택지를 제공함
  - RestClient: 동기 방식이며, 메서드 체인 형태의 Fluent API를 제공함
  - WebClient: 논블로킹(Non-Blocking) 리액티브 방식이며, Fluent API를 제공함
  - RestTemplate: 동기 방식이며, Template Method 형태의 API를 제공함
  - HTTP Service Client: 인터페이스를 선언해두면 그에 대한 Proxy를 만들어주는 선언형 방식
- 새로 구현하는 경우, 동기 방식은 RestClient를 사용하는 것이 권장됨

<br>

### 의존성 추가
- spring-boot-starter-restclient: RestClient, RestTemplate, HTTP Service Client 등 동기 방식 Client 사용 시 추가
- spring-boot-starter-webclient: WebClient 등 리액티브 방식 Client 사용 시 추가
```
build.gradle

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-restclient'
}
```

<br>

### RestClient
- 동기 방식으로 HTTP 요청을 수행하는 Client로써, Spring Framework 6.1부터 추가됨
- 여러 Thread에서 동시에 사용해도 안전함
- Spring Boot가 RestClient.Builder를 Prototype Bean으로 자동 설정해주므로, 이를 주입받아 생성하는 것이 권장됨
  - HttpMessageConverters와 ClientHttpRequestFactory가 미리 구성된 상태로 제공됨
  - baseUrl을 지정해두면 이후 호출에서는 나머지 경로만 지정하여 사용할 수 있음
- 요청 구성부터 응답 변환까지를 메서드 체인으로 이어서 작성함
  - get(), post(), put(), delete(): 사용할 HTTP 메서드 지정
  - uri(): 호출할 경로 지정
  - retrieve(): 요청 수행 후 응답을 받아옴
  - body(변환할_타입): 응답 본문을 지정한 타입의 객체로 변환
```
@Bean
public ApplicationRunner run(RestClient.Builder builder) {
    RestClient restClient = builder.baseUrl("http://localhost:8080").build();

    return args -> {
        Quote quote = restClient.get()
                .uri("/api/random")
                .retrieve()
                .body(Quote.class);

        log.info(quote.toString());
    };
}
```
- 애플리케이션 구동 직후 호출을 수행하기 위해 ApplicationRunner를 사용할 수 있으며, 관련 내용은 [Spring Boot](/spring-framework/spring-boot.md) 참고

<br>

### 응답 데이터 매핑
- 응답으로 받은 JSON은 HttpMessageConverter를 통해 지정한 타입의 객체로 변환됨
  - 관련 내용은 [직렬화, Marshalling, JSON](/dto-json-cors/serialization-marshalling-JSON.md) 참고
- 응답을 담기 위한 객체는 값이 변하지 않으므로, record를 사용하면 간결하게 구현할 수 있음
- 외부 API의 응답에는 사용하지 않는 필드가 함께 담겨오는 경우가 많으므로, @JsonIgnoreProperties(ignoreUnknown = true)를 선언하여 매핑 대상이 아닌 필드를 무시하도록 해야 함
```
@JsonIgnoreProperties(ignoreUnknown = true)
public record Quote(String type, Value value) {

}


@JsonIgnoreProperties(ignoreUnknown = true)
public record Value(Long id, String quote) {

}
```

<br>

### RestClient, RestTemplate, WebClient 비교
- RestClient
  - 동기 방식이며, Fluent API를 제공함
  - 신규 구현 시 권장되는 기본 선택지
  - spring-boot-starter-restclient를 추가하면 RestClient.Builder가 자동 설정됨
- RestTemplate
  - 동기 방식이며, Template Method 형태의 API를 제공함
  - Spring Framework 7.0부터 RestClient를 대체재로 하여 Deprecated 처리되었으며, 향후 제거될 예정임
  - Spring Boot는 RestTemplate 자체를 Bean으로 등록해주지는 않고, RestTemplateBuilder만 자동 설정해줌
  - 기존 코드 유지보수 목적이 아니라면 사용하지 않는 것이 좋음
- WebClient
  - 논블로킹 리액티브 방식이며, 동기/비동기/스트리밍 방식 모두 수행 가능함
  - 적은 리소스로 높은 동시성을 처리해야 하거나, 응답을 스트리밍으로 다뤄야 할 때 사용함
  - spring-boot-starter-webclient를 추가하면 WebClient.Builder가 자동 설정됨
- 이름이 유사한 RestTestClient는 테스트 수행 시 사용하는 별개의 Client이며, 관련 내용은 [Spring Test](/di-spring-test/spring-test.md) 참고

<br>

### 하위 HTTP Client 라이브러리
- 실제 통신은 ClientHttpRequestFactory 구현체를 통해 이루어지며, classpath에 존재하는 라이브러리를 감지하여 자동 설정됨
- RestClient와 RestTemplate의 경우, 아래 우선순위로 감지함
  1. Apache HttpClient
  2. Jetty HttpClient
  3. Reactor Netty HttpClient
  4. JDK HttpClient(java.net.http.HttpClient)
  5. Simple JDK Client(java.net.HttpURLConnection)
- 자동 감지된 결과를 사용하지 않고 직접 지정하려면 spring.http.clients.imperative.factory 속성을 이용함

<br>

#### 참고
- Spring 공식문서 <Consuming a RESTful Web Service> - https://spring.io/guides/gs/consuming-rest
- Spring Framework Reference Documentation <REST Clients> - https://docs.spring.io/spring-framework/reference/integration/rest-clients.html

#### 배워가는 것들
- 지금까지 요청을 받는 쪽만 정리해왔는데, 요청을 보내는 쪽에도 별도의 도구가 있다는 것을 익힐 수 있었다. 마이크로서비스 간 통신이나 외부 연동을 구현할 때 반드시 필요한 부분이다.
- 익숙하게 알고 있던 RestTemplate이 Spring Framework 7.0부터 Deprecated 처리되었다는 것을 알게 되었다. 오래된 자료를 보고 그대로 따라 구현하면 사라질 API를 쓰게 되므로, 클래스를 고르기 전에 현재 상태를 확인하는 습관을 들여야겠다.
- @JsonIgnoreProperties가 왜 필요한지 클라이언트 관점에서 이해할 수 있었다. 응답 스펙을 내가 통제할 수 없기 때문에, 필요한 필드만 받고 나머지는 무시하도록 해야 연동이 쉽게 깨지지 않는다.
