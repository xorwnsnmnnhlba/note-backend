# 외부 의존성을 격리한 테스트

### 테스트하기 어려운 코드
- 아래와 같은 대상에 직접 의존하는 코드는 테스트 결과가 실행할 때마다 달라지거나, 원하는 상황을 만들기 어려움
  - 외부 API: 네트워크 상태에 따라 결과가 달라지고, 오류나 지연 상황을 마음대로 만들 수 없음
  - 현재 시각: "1시간 뒤", "다음 주 월요일 9시" 같은 상황을 만들려면 실제로 기다려야 함
  - 비동기 실행: 결과가 언제 나올지 알 수 없어 검증 시점을 정하기 어려움
  - 데이터베이스: 운영과 다른 DB(H2 등)로 테스트하면, 실제 DB에서만 드러나는 문제를 놓침
- 해결 방법은 의존 대상을 인터페이스나 Bean으로 분리해두고, 테스트에서는 원하는 동작을 하는 대체물로 바꿔 끼우는 것임
  - Test Double의 종류는 [Unit Test](/di-spring-test/unit-test.md) 참고

<br>

### Fake 객체
- 실제 구현 대신 동작하는 간단한 구현체로써, 미리 정한 응답을 돌려주고 받은 요청을 기록함
- Mock 라이브러리로 메서드 호출마다 동작을 지정하는 방식에 비해
  - 여러 테스트에서 재사용하기 쉽고, 테스트 코드가 "어떤 응답이 왔을 때"라는 의도를 드러냄
  - 내부 호출 방식이 바뀌어도 테스트가 덜 깨짐
```
public interface RequestExecutor {

    HttpExchange execute(ResolvedRequest request);

}


public class FakeRequestExecutor implements RequestExecutor {

    private final List<ResolvedRequest> requests = new CopyOnWriteArrayList<>();

    private volatile Function<ResolvedRequest, HttpExchange> behavior;

    @Override
    public HttpExchange execute(ResolvedRequest request) {
        requests.add(request);
        return behavior.apply(request);
    }

    public void respond(int statusCode, String body) {
        behavior = request -> new HttpExchange(request, statusCode, Map.of(), body, Duration.ofMillis(12));
    }

    public List<ResolvedRequest> requests() {
        return List.copyOf(requests);
    }

}
```
- 외부 호출을 인터페이스로 분리해두면, 서비스 로직 테스트는 네트워크 없이 Fake로, 실제 HTTP 통신은 해당 구현체의 통합 테스트로 나눠서 검증할 수 있음

<br>

### 테스트에서 Bean 바꿔 끼우기
- @TestConfiguration으로 테스트 전용 설정 클래스를 만들고, 필요한 테스트에서만 @Import로 가져옴
- 같은 타입의 Bean이 이미 있으므로, @Primary를 붙여 테스트용 Bean이 우선 주입되도록 함
- 예: 백그라운드 Thread에서 실행하는 구현을, 호출한 Thread에서 바로 실행하는 구현으로 바꾸면 API 응답을 받은 시점에 결과를 바로 검증할 수 있음
  - 백그라운드 동작 자체는 별도의 테스트에서 검증함
```
@TestConfiguration(proxyBeanMethods = false)
public class InlineBackgroundRunsConfiguration {

    @Bean
    @Primary
    BackgroundRuns inlineBackgroundRuns() {
        return Runnable::run;
    }

}


@SpringBootTest
@Import({ TestcontainersConfiguration.class, InlineBackgroundRunsConfiguration.class })
class RunControllerTest {

}
```
- 특정 Bean 하나를 Mockito Mock으로 바꿀 때는 @MockitoBean을 사용함(Spring Framework 6.2부터 @MockBean을 대체함)

<br>

### 시각 제어하기
- 코드에서 Instant.now() 대신 주입받은 Clock의 instant()를 사용하도록 하면, 테스트에서 시각을 원하는 대로 바꿀 수 있음
  - 고정된 시각만 필요하다면 Clock.fixed()를 사용함
  - 테스트 도중 시간을 흐르게 해야 한다면, 값을 바꿀 수 있는 Clock을 직접 만들어 사용함
```
public class MutableClock extends Clock {

    private volatile Instant now;

    public MutableClock(Instant now) {
        this.now = now;
    }

    public void advance(Duration duration) {
        this.now = now.plus(duration);
    }

    @Override
    public Instant instant() {
        return now;
    }

    @Override
    public ZoneId getZone() {
        return ZoneOffset.UTC;
    }

    @Override
    public Clock withZone(ZoneId zone) {
        throw new UnsupportedOperationException();
    }

}


clock.advance(Duration.ofHours(1));
poller.poll();
```
- Thread.sleep()처럼 실제로 기다리는 동작도 함수로 주입받으면, 테스트에서는 기다리지 않고 "몇 초를 기다리려 했는지"만 기록하여 검증할 수 있음

<br>

### WireMock
- 실제 HTTP 서버처럼 동작하는 가짜 API 서버(Mock Server)
  - 요청 조건(메서드, 경로, 헤더, 본문)에 맞는 응답을 미리 정의해두는 방식(Stubbing)
  - 응답 지연(fixedDelayMilliseconds), 연결 끊김, 특정 상태 코드 등 실제 서버로는 만들기 어려운 상황을 재현할 수 있음
- HTTP Client를 사용하는 구현체의 타임아웃, 리다이렉트, 4xx/5xx 처리를 실제 통신으로 검증할 때 사용함
- 응답 정의는 JSON 파일(mappings)로 작성할 수 있음
```
src/test/resources/wiremock/mappings/stubs.json

{
  "mappings": [
    {
      "request": { "method": "GET", "url": "/users/1" },
      "response": {
        "status": 200,
        "headers": { "Content-Type": "application/json" },
        "body": "{\"id\":1,\"name\":\"kim\"}"
      }
    },
    {
      "request": { "method": "GET", "url": "/slow" },
      "response": { "status": 200, "fixedDelayMilliseconds": 3000 }
    }
  ]
}
```
- Testcontainers로 WireMock 컨테이너를 띄우고, 정의 파일을 컨테이너 안으로 복사하여 사용할 수 있음
```
@Testcontainers
class RestClientRequestExecutorTest {

    @Container
    static final GenericContainer<?> wiremock = new GenericContainer<>(DockerImageName.parse("wiremock/wiremock:3.13.2"))
            .withExposedPorts(8080)
            .withCopyFileToContainer(MountableFile.forClasspathResource("wiremock/mappings"), "/home/wiremock/mappings")
            .waitingFor(Wait.forHttp("/__admin/health").forStatusCode(200));

    private String url(String path) {
        return "http://" + wiremock.getHost() + ":" + wiremock.getMappedPort(8080) + path;
    }

}
```
- 컨테이너 없이 JUnit 확장(WireMockExtension)으로 같은 JVM 안에서 실행하는 방법도 있음

<br>

### Testcontainers와 Spring Boot 연동
- Testcontainers의 기본 개념은 [Spring Test](/di-spring-test/spring-test.md) 참고
- Spring Boot 3.1부터는 @ServiceConnection을 사용하면, 컨테이너의 접속 정보(URL, 계정)를 DataSource 설정에 자동으로 연결해줌
  - application-test.yml에 접속 정보를 따로 적지 않아도 됨
- Testcontainers 2.0부터는 모듈 이름에 testcontainers- 접두어가 붙고, PostgreSQLContainer처럼 제네릭 타입 파라미터가 없어지는 등 변경 사항이 있음
```
build.gradle

dependencies {
    testImplementation 'org.springframework.boot:spring-boot-testcontainers'
    testImplementation 'org.testcontainers:testcontainers-junit-jupiter'
    testImplementation 'org.testcontainers:testcontainers-postgresql'
}


@TestConfiguration(proxyBeanMethods = false)
public class TestcontainersConfiguration {

    @Bean
    @ServiceConnection
    PostgreSQLContainer postgresContainer() {
        return new PostgreSQLContainer(DockerImageName.parse("postgres:18-alpine"));
    }

}
```
- @DataJpaTest는 기본적으로 내장 DB로 바꿔서 실행하므로, 컨테이너의 실제 DB를 사용하려면 @AutoConfigureTestDatabase(replace = Replace.NONE)를 함께 지정함
```
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Import(TestcontainersConfiguration.class)
class TestRunQueriesTest {

}
```
- 개발 중 로컬에 DB를 설치하지 않고 애플리케이션을 실행하고 싶다면, 테스트 소스에 실행용 클래스를 만들고 ./gradlew bootTestRun으로 실행함
  - 컨테이너 DB에 연결된 상태로 애플리케이션이 실행되며, 종료하면 데이터는 사라짐
```
public class TestSentinelApplication {

    public static void main(String[] args) {
        SpringApplication.from(SentinelApplication::main).with(TestcontainersConfiguration.class).run(args);
    }

}
```

<br>

#### 참고
- Spring Boot Reference Documentation <Testcontainers> - https://docs.spring.io/spring-boot/reference/testing/testcontainers.html
- WireMock Documentation <Stubbing> - https://wiremock.org/docs/stubbing/
- Testcontainers for Java - https://java.testcontainers.org/

#### 배워가는 것들
- 시각과 대기를 주입받도록 만들어두면, 시간이 걸리는 시나리오도 기다리지 않고 테스트할 수 있다는 것을 알게 되었다. 테스트하기 쉬운 구조가 곧 의존성이 잘 드러나는 구조다.
- Mock으로 호출을 하나하나 지정하는 것보다, 재사용할 수 있는 Fake 객체를 두는 편이 테스트의 의도를 더 잘 드러낸다는 점이 인상적이었다.
- 외부 API는 WireMock으로, DB는 Testcontainers로 실제와 가까운 환경을 만들 수 있다는 것을 배웠다. 서비스 로직은 Fake로 빠르게, 통신과 쿼리는 실제에 가깝게 나눠서 검증하는 것이 좋겠다.
