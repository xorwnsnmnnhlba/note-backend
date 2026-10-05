# 타임아웃과 재시도

### 장애에 견디는 애플리케이션
- 외부 API, DB, 네트워크는 언제든 느려지거나 끊길 수 있음
- 이러한 일시적인 장애가 애플리케이션 전체의 장애로 번지지 않도록 하는 것을 복원력(Resilience)이라고 함
- 대표적인 수단
  - 타임아웃(Timeout): 응답을 무한정 기다리지 않음
  - 재시도(Retry): 일시적인 실패라면 잠시 뒤 다시 시도함
  - 동시성 제한(Concurrency Limit): 한 번에 보내는 요청 수를 제한하여 상대 시스템과 자신을 보호함
  - 서킷 브레이커(Circuit Breaker): 실패가 계속되면 한동안 호출 자체를 막음

<br>

### 타임아웃
- 타임아웃을 지정하지 않으면, 상대가 응답하지 않을 때 Thread가 무기한 대기하게 됨
  - 대기하는 Thread와 커넥션이 쌓이면 결국 애플리케이션 전체가 멈출 수 있음
- HTTP Client의 타임아웃 종류
  - 연결 타임아웃(Connect Timeout): 상대 서버와 TCP 연결을 맺기까지 기다리는 최대 시간
  - 응답 타임아웃(Read Timeout, Socket Timeout): 연결 후 응답 데이터를 기다리는 최대 시간
  - 커넥션 풀에서 커넥션을 얻기까지 기다리는 시간도 따로 존재함
- 알림 발송이나 생존 신호처럼 부가적인 호출은 타임아웃을 짧게 두어, 본래 작업을 지연시키지 않도록 해야 함
- DB 연결에도 타임아웃이 필요함
  - HikariCP의 connection-timeout: 풀에서 커넥션을 얻기까지 기다리는 시간(기본 30초)
  - PostgreSQL JDBC 드라이버의 socketTimeout: 쿼리 응답을 기다리는 시간. 기본값 0은 무제한이므로, 쿼리 도중 네트워크가 끊기면 Thread가 영원히 대기할 수 있음
  - tcpKeepAlive: 오래 유휴 상태인 연결이 끊겼는지 확인함
```
spring:
  datasource:
    hikari:
      maximum-pool-size: 5
      connection-timeout: 10000
      keepalive-time: 300000
      data-source-properties:
        socketTimeout: 30
        connectTimeout: 10
        tcpKeepAlive: true
```
- DBCP와 HikariCP에 관한 내용은 [JDBC](/database/jdbc.md) 참고

<br>

### 재시도할 것인가, 말 것인가
- 모든 실패를 재시도하면 안 되며, 실패의 종류와 요청의 성격을 함께 따져야 함
- 실패의 종류
  - 연결 실패: 요청이 서버에 닿지 않았으므로, 다시 보내도 중복 처리될 위험이 없음
  - 타임아웃, 응답 도중 연결 끊김: 서버가 이미 요청을 처리했을 수 있음
  - 응답을 받은 경우(4xx, 판정 실패 등): 같은 요청을 다시 보내도 같은 결과일 가능성이 높으므로 재시도 대상이 아님
  - 예외: 503 Service Unavailable, 429 Too Many Requests는 Retry-After 헤더에 안내된 시간 뒤에 다시 시도할 수 있음
- 요청의 성격
  - 안전한(Safe) 메서드(GET, HEAD, OPTIONS)와 멱등한(Idempotent) 메서드는 여러 번 보내도 결과가 같으므로 재시도해도 됨
  - POST처럼 멱등하지 않은 요청을 타임아웃 후 재시도하면, 등록이 두 번 되는 문제가 생길 수 있음
  - 멱등성에 관한 내용은 [HTTP 메서드](/http/http-method.md) 참고
- 재시도 사이에는 대기 시간을 두어야 하며, 대기 시간을 점점 늘리는 방식(Exponential Backoff)과 무작위 값을 더하는 방식(Jitter)을 함께 사용하면 여러 Client가 동시에 재시도하여 상대 서버에 부하가 몰리는 것을 막을 수 있음
- 재시도 횟수에는 상한을 두어야 하며, 몇 번 재시도했는지 기록해두면 재시도가 잦은 것 자체를 불안정 신호로 활용할 수 있음
- 일시적인 장애가 아닌 경우(예: VPN이 끊겨 DB에 접속할 수 없는 경우)는 즉시 재시도해도 소용이 없으므로, 이번 주기를 건너뛰고 다음 주기를 기다리는 편이 나음

<br>

### 재시도 직접 구현하기
- 재시도 정책을 Spring에 의존하지 않는 클래스로 만들면, 단위 테스트로 규칙을 쉽게 검증할 수 있음
- 대기 동작(Thread.sleep)을 주입받도록 하면, 테스트에서는 실제로 기다리지 않고 대기 요청만 기록할 수 있음
```
public final class TransportRetry {

    private static final Set<HttpMethod> SAFE_METHODS = EnumSet.of(HttpMethod.GET, HttpMethod.HEAD, HttpMethod.OPTIONS);

    static boolean retryable(Reason reason, HttpMethod method) {
        return switch (reason) {
            case CONNECTION_FAILED -> true;
            case TIMEOUT, IO_ERROR -> SAFE_METHODS.contains(method);
            case BLOCKED_DESTINATION -> false;
        };
    }

    public Sent send(ResolvedRequest request, RequestExecutor executor) {
        int retries = 0;
        while (true) {
            try {
                return new Sent(executor.execute(request), null, retries);
            }
            catch (RequestExecutionException ex) {
                if (retries >= maxRetries || !retryable(ex.reason(), request.method())) {
                    return new Sent(null, ex, retries);
                }
                try {
                    sleeper.sleep(delay);
                }
                catch (InterruptedException interrupted) {
                    Thread.currentThread().interrupt();
                    return new Sent(null, ex, retries);
                }
                retries++;
            }
        }
    }

}


new TransportRetry(1, Duration.ofSeconds(1), Thread::sleep);
```
- InterruptedException을 잡았을 때는 Thread.currentThread().interrupt()로 인터럽트 상태를 되돌려놓아야, 호출한 쪽에서도 중단 요청을 알 수 있음
- HTTP Client 라이브러리가 제공하는 자동 재시도는 첫 실패가 가려지고 기록이 남지 않으므로, 재시도 여부를 기록해야 한다면 끄고 애플리케이션에서 직접 처리하는 것이 좋음

<br>

### Spring Framework 7의 Resilience 기능
- Spring Framework 7.0부터는 별도 라이브러리(Spring Retry) 없이 재시도와 동시성 제한 기능을 기본으로 제공함
- @Configuration 클래스에 @EnableResilientMethods를 선언하여 활성화함
- @Retryable
  - 메서드에서 예외가 발생하면 정해진 정책에 따라 다시 호출함
  - 기본값은 최대 3회 재시도(첫 호출 포함 최대 4회), 재시도 간격 1초이며 모든 예외가 대상임
  - includes, excludes: 재시도할 예외와 제외할 예외
  - maxRetries, delay, multiplier, maxDelay, jitter: 재시도 횟수와 대기 정책
```
@Retryable(includes = MessageDeliveryException.class, maxRetries = 4, delay = 100, multiplier = 2, maxDelay = 1000, jitter = 10)
public void sendNotification() {
    ...
}
```
- @ConcurrencyLimit
  - 메서드를 동시에 실행할 수 있는 수를 제한함. 한도를 넘은 호출은 자리가 날 때까지 기다림
  - 가상 스레드 환경에서 외부 시스템을 보호할 때 유용함
```
@ConcurrencyLimit(10)
public void callGateway() {
    ...
}
```
- RetryTemplate(org.springframework.core.retry)으로 프로그래밍 방식의 재시도도 사용할 수 있음
```
RetryPolicy retryPolicy = RetryPolicy.builder()
        .includes(MessageDeliveryException.class)
        .maxRetries(4)
        .delay(Duration.ofMillis(100))
        .build();

new RetryTemplate(retryPolicy).invoke(() -> client.send(message));
```
- 애노테이션 방식은 Proxy 기반이므로, @Transactional과 마찬가지로 같은 클래스 안의 호출에는 적용되지 않음

<br>

#### 참고
- Spring Framework Reference Documentation <Resilience Features> - https://docs.spring.io/spring-framework/reference/core/resilience.html
- PostgreSQL JDBC Driver <Connection Parameters> - https://jdbc.postgresql.org/documentation/use/
- AWS Builders' Library <Timeouts, retries, and backoff with jitter> - https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/

#### 배워가는 것들
- DB 드라이버의 기본 응답 타임아웃이 무제한일 수 있다는 것을 알게 되었다. 기본값을 믿지 말고 연결이 끊겼을 때 무엇이 멈추는지 확인해봐야 한다.
- 재시도는 무조건 좋은 것이 아니라, 요청이 서버에 닿았는지와 메서드가 멱등한지에 따라 판단해야 한다는 점이 인상적이었다. 잘못된 재시도는 장애를 숨기거나 데이터를 중복으로 만든다.
- Spring Framework 7부터 @Retryable과 @ConcurrencyLimit이 기본 기능이 되었다는 것을 알게 되었다. 간단한 재시도는 이제 별도 의존성 없이 처리할 수 있다.
