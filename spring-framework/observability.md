# 관측 가능성

### 관측 가능성(Observability)
- 시스템 밖으로 나오는 신호만 보고도 시스템 내부에서 무슨 일이 일어나는지 알 수 있는 정도
  - 장애가 났을 때 어느 요청이, 어느 구간에서, 왜 느려지거나 실패했는지를 추측이 아닌 데이터로 찾을 수 있어야 함
- 세 가지 신호(Signal)로 구성됨
  - 지표(Metrics): 요청 수, 응답 시간, 오류율, 커넥션 풀 사용량처럼 시간에 따라 집계한 숫자. 경향을 보고 알림을 거는 데 사용함
  - 추적(Traces): 하나의 요청이 여러 서비스와 DB를 거쳐 가는 경로와 구간별 소요 시간. 어디서 느려졌는지 찾는 데 사용함
  - 로그(Logs): 특정 시점에 일어난 일의 상세 기록. 왜 실패했는지 확인하는 데 사용함
- 세 신호가 같은 추적 ID로 연결되어 있어야, 지표에서 이상을 발견하고 해당 요청의 추적과 로그로 이어서 원인을 찾을 수 있음
- Actuator가 상태 확인과 운영 정보를 제공한다면, 관측 가능성은 실제 요청의 흐름과 성능을 수집하는 영역임. 관련 내용은 [Actuator](/spring-framework/actuator.md) 참고

<br>

### Micrometer
- 지표와 추적을 수집하는 Java 라이브러리로, Spring Boot가 내부적으로 사용함
  - 로그에서 SLF4J가 구현체를 감추듯이, Micrometer는 Prometheus, Datadog, OpenTelemetry 같은 수집 도구를 감추는 Facade 역할을 함
  - 코드는 Micrometer API로 한 번만 작성하고, 보낼 곳은 의존성과 설정으로 바꿀 수 있음
- Spring Boot는 HTTP 요청 처리, RestClient 호출, JVM 메모리와 GC, HikariCP 커넥션 풀 등의 지표를 자동으로 수집함
- 지표의 종류
  - Counter: 계속 증가하는 값(주문 건수, 오류 횟수)
  - Timer: 걸린 시간과 횟수(외부 API 응답 시간)
  - Gauge: 현재 값(대기 중인 작업 수, 캐시 크기)
  - DistributionSummary: 시간이 아닌 값의 분포(주문 금액, 요청 크기)
```
@Service
public class OrderService {

    private final Counter orderCounter;

    public OrderService(MeterRegistry meterRegistry) {
        this.orderCounter = Counter.builder("orders.created")
                .tag("channel", "web")
                .register(meterRegistry);
    }

    public void createOrder(OrderRequest request) {
        ...
        orderCounter.increment();
    }

}
```
- 태그의 값 종류가 무한히 늘어나면(High Cardinality) 지표 저장소에 부담이 커지므로, 사용자 ID나 주문 번호처럼 값이 계속 늘어나는 것은 지표 태그로 쓰지 않고 추적이나 로그에 남김

<br>

### Observation API
- 하나의 작업을 한 번 계측하면 지표(Timer)와 추적(Span)을 함께 만들어주는 Micrometer의 API
- Observation.createNotStarted(이름, registry).observe(작업) 형태로 감싸며, 걸린 시간, 성공과 실패 여부가 함께 기록됨
  - lowCardinalityKeyValue: 지표와 추적 모두에 들어가는 값. 종류가 적은 값을 넣음
  - highCardinalityKeyValue: 추적에만 들어가는 값. 주문 번호처럼 종류가 많은 값을 넣음
- 메서드에 @Observed를 선언하는 방식도 있으며, AOP를 사용하므로 같은 클래스 안의 호출에는 적용되지 않음
```
@Component
public class PaymentProcessor {

    private final ObservationRegistry observationRegistry;

    public PaymentProcessor(ObservationRegistry observationRegistry) {
        this.observationRegistry = observationRegistry;
    }

    public void process(Payment payment) {
        Observation.createNotStarted("payment.process", observationRegistry)
                .lowCardinalityKeyValue("method", payment.method())
                .highCardinalityKeyValue("payment.id", payment.id())
                .observe(() -> gateway.charge(payment));
    }

}
```

<br>

### 분산 추적(Distributed Tracing)
- 하나의 요청 전체를 Trace, 그 안의 각 구간(HTTP 처리, DB 쿼리, 외부 API 호출)을 Span이라고 함
  - 모든 Span은 같은 Trace ID를 공유하고, 각자의 Span ID와 부모 Span ID를 가짐
- 서비스 사이에서는 traceparent 헤더(W3C Trace Context)로 Trace ID를 전달하여, 여러 서비스를 거치는 요청을 하나의 흐름으로 이어 붙임
  - Spring Boot가 자동 설정한 RestClient.Builder, WebClient.Builder로 만든 Client는 이 헤더를 자동으로 전달함
  - RestClient.create()처럼 직접 만든 Client는 전달되지 않으므로 주의해야 함
- 모든 요청을 추적하면 비용이 크므로 일부만 수집하는 샘플링(Sampling)을 적용함
  - management.tracing.sampling.probability로 비율을 지정하며, 기본값은 0.1(10%)임

<br>

### OpenTelemetry
- 지표, 추적, 로그를 수집하고 전송하는 방식을 표준화한 CNCF 프로젝트
  - OTLP(OpenTelemetry Protocol)라는 표준 형식으로 보내므로, 수집 도구(Grafana, Jaeger, Datadog 등)를 바꿔도 애플리케이션 코드를 바꾸지 않아도 됨
- Spring Boot 4.x부터 spring-boot-starter-opentelemetry가 추가됨
  - Micrometer로 수집한 지표와 추적을 OTLP 형식으로 내보내는 데 필요한 의존성을 한 번에 추가함
  - Actuator 없이도 사용할 수 있음
```
build.gradle

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-opentelemetry'
}
```
```
application.yml

spring:
  application:
    name: order-service
management:
  tracing:
    sampling:
      probability: 1.0
  opentelemetry:
    tracing:
      export:
        otlp:
          endpoint: http://localhost:4317
```
- 로컬 개발 환경에서는 OTLP 수집기, Prometheus, Tempo, Loki, Grafana를 하나로 묶은 grafana/otel-lgtm 컨테이너를 Docker Compose로 띄워 바로 확인할 수 있음

<br>

### Prometheus로 지표 수집하기
- Prometheus는 애플리케이션의 지표 Endpoint를 주기적으로 가져가서(Pull) 저장하는 시계열 데이터베이스이며, Grafana로 대시보드를 만듦
- micrometer-registry-prometheus를 추가하고 Actuator의 prometheus Endpoint를 노출하면 /actuator/prometheus에서 지표를 제공함
  - 지표 Endpoint는 외부에 공개하지 않고, 별도 포트나 내부망에서만 접근할 수 있게 해야 함
```
build.gradle

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    runtimeOnly 'io.micrometer:micrometer-registry-prometheus'
}
```
```
application.yml

management:
  endpoints:
    web:
      exposure:
        include: health, prometheus
```

<br>

### 로그와 추적 연결하기
- Micrometer Tracing을 사용하면 Spring Boot가 로그에 현재 요청의 Trace ID와 Span ID를 자동으로 넣어줌
  - 기본 형식은 [traceId-spanId]이며, logging.pattern.correlation으로 바꿀 수 있음
- 오류 로그에서 Trace ID를 찾으면 그 요청의 전체 추적을 바로 열어볼 수 있음
- 로그는 JSON 같은 구조화된 형식으로 남기면 검색과 집계가 쉬워지며, Spring Boot는 logging.structured.format.console 속성으로 구조화된 로그를 지원함

<br>

### 무엇을 관측할 것인가
- 서비스 단위로는 요청량(Rate), 오류율(Errors), 응답 시간(Duration)을 기본으로 봄(RED 방식)
- 자원 단위로는 사용률(Utilization), 포화도(Saturation), 오류(Errors)를 봄(USE 방식)
  - 커넥션 풀 대기 시간, 스레드 사용량, 큐 길이 등
- 응답 시간은 평균보다 p95, p99 같은 백분위수로 봐야, 일부 사용자가 겪는 느린 응답을 놓치지 않음
- 알림은 원인(CPU 사용률)보다 사용자가 느끼는 증상(오류율, 응답 시간)을 기준으로 거는 것이 좋음

<br>

#### 참고
- Spring Boot Reference Documentation <Observability> - https://docs.spring.io/spring-boot/reference/actuator/observability.html
- Spring Boot Reference Documentation <Tracing> - https://docs.spring.io/spring-boot/reference/actuator/tracing.html
- Spring 공식블로그 <OpenTelemetry with Spring Boot> - https://spring.io/blog/2025/11/18/opentelemetry-with-spring-boot/
- Micrometer 공식문서 <Observation API> - https://docs.micrometer.io/micrometer/reference/observation.html
- OpenTelemetry 공식문서 - https://opentelemetry.io/docs/

#### 배워가는 것들
- 지표, 추적, 로그가 따로 있으면 각각은 유용해도 원인을 찾기 어렵고, 같은 Trace ID로 연결되어 있어야 한다는 점이 핵심이라는 것을 알게 되었다.
- Micrometer가 로그의 SLF4J처럼 수집 도구를 감추는 Facade라는 설명으로, 코드를 바꾸지 않고 보낼 곳만 바꿀 수 있는 이유를 이해할 수 있었다.
- 사용자 ID를 지표 태그로 넣으면 안 되는 이유가 Cardinality 때문이라는 것을 알게 되었다. 값의 종류가 많은 정보는 추적과 로그에 남겨야 한다.
