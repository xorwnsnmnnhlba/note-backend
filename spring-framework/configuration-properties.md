# 외부 설정(Externalized Configuration)

### 외부 설정
- 코드를 바꾸지 않고 실행 환경(로컬, 개발, 운영)에 따라 동작을 바꿀 수 있도록, 설정 값을 코드 밖에 두는 방식
  - DB 접속 주소, 타임아웃, 기능 사용 여부, 외부 서비스 주소 등이 대상임
- Spring Boot는 여러 위치의 설정을 하나의 Environment로 모아주며, 같은 키가 여러 곳에 있으면 우선순위가 높은 쪽이 적용됨
  - 우선순위가 높은 순서(주요 항목): 명령행 인자 → OS 환경변수 → application-{profile}.yml → application.yml
- 따라서 application.yml에는 기본값을 두고, 환경마다 다른 값은 환경변수로 덮어쓰는 방식을 많이 사용함

<br>

### Placeholder와 환경변수
- ${이름:기본값} 형식으로 다른 설정이나 환경변수 값을 참조할 수 있음
  - 값이 없으면 콜론 뒤의 기본값이 사용되며, 기본값을 빈 값으로 두려면 ${이름:}처럼 작성함
```
spring:
  datasource:
    url: ${DB_URL:jdbc:postgresql://localhost:5432/sentinel}
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:}
```
- 완화된 바인딩(Relaxed Binding)
  - 환경변수는 설정 키를 대문자로 바꾸고, 점(.)은 밑줄(_)로 바꾸고 하이픈(-)은 제거한 이름으로도 인식됨
  - 예: sentinel.schedule.poll-interval → SENTINEL_SCHEDULE_POLLINTERVAL
  - yml 파일에서는 케밥 표기법(poll-interval)을 사용하는 것이 권장됨
- 비밀번호, API 키, Webhook 주소처럼 자격증명에 해당하는 값은 application.yml에 직접 적지 않고, 환경변수로만 주입해야 함
  - 설정 파일은 Git에 올라가므로, 한 번 커밋된 자격증명은 이력에 계속 남음

<br>

### @Value와 @ConfigurationProperties
- @Value("${키}")
  - 필드나 생성자 파라미터에 값 하나를 주입함
  - 간단하지만, 관련된 설정이 여러 클래스에 흩어지고 키 이름을 문자열로 반복해서 적어야 함
- @ConfigurationProperties("접두어")
  - 같은 접두어를 가진 설정들을 하나의 객체로 묶어서 주입받음
  - 타입 변환, 기본값 지정, 검증을 한곳에서 처리할 수 있음
- Spring Boot 3.0부터는 생성자가 하나인 record에 선언하면 생성자 바인딩이 적용되어, 값이 바뀌지 않는 설정 객체를 간결하게 만들 수 있음
  - @DefaultValue로 설정이 없을 때의 기본값을 지정함
```
application.yml

sentinel:
  schedule:
    enabled: true
    zone: Asia/Seoul
    poll-interval: 60s
    gap-threshold: 10m


@ConfigurationProperties("sentinel.schedule")
public record ScheduleProperties(
        @DefaultValue("true") boolean enabled,
        @DefaultValue("Asia/Seoul") ZoneId zone,
        @DefaultValue("60s") Duration pollInterval,
        @DefaultValue("10m") Duration gapThreshold) {
}
```
- 설정 클래스를 Bean으로 등록하는 방법
  - @EnableConfigurationProperties(ScheduleProperties.class): 지정한 클래스만 등록함
  - @ConfigurationPropertiesScan: 패키지를 탐색하여 @ConfigurationProperties 클래스를 모두 등록함
- spring-boot-configuration-processor를 annotationProcessor로 추가하면, IDE에서 yml 작성 시 자동 완성과 설명을 사용할 수 있음

<br>

### 타입 변환
- 문자열로 적은 설정 값을 필드 타입에 맞게 자동으로 변환해줌
  - Duration: 10s, 5m, 2h, 7d처럼 단위를 붙여서 작성함. 단위 없이 숫자만 적으면 밀리초로 해석되며, @DurationUnit으로 기본 단위를 바꿀 수 있음
  - DataSize: 10MB, 512KB
  - ZoneId, enum, List 등도 변환됨
- 시간 값을 int timeoutMillis처럼 숫자로 받으면 단위를 이름으로만 구분해야 하므로, Duration으로 받는 것이 실수를 줄여줌

<br>

### 설정 값 검증
- 설정 값이 잘못되었을 때 실행 중에 오류가 나는 것보다, 기동 시점에 실패하는 것이 원인을 찾기 쉬움(Fail-Fast)
- 방법 1: @Validated와 Bean Validation 애노테이션 사용
```
@Validated
@ConfigurationProperties("sentinel.runner.manual")
public record ManualRunProperties(@Min(1) @DefaultValue("4") int maxConcurrent) {
}
```
- 방법 2: record의 Compact Constructor에서 직접 검증
  - 공백을 null로 바꾸는 등의 정규화도 함께 처리할 수 있음
  - 자격증명에 해당하는 값은 오류 메시지에 값 자체를 넣지 않아야 함
```
@ConfigurationProperties("sentinel.heartbeat")
public record HeartbeatProperties(String url, @DefaultValue("5s") Duration timeout) {

    public HeartbeatProperties {
        url = url == null || url.isBlank() ? null : url.strip();
        if (timeout.isNegative() || timeout.isZero()) {
            throw new IllegalArgumentException("sentinel.heartbeat.timeout must be positive: " + timeout);
        }
    }

    public boolean enabled() {
        return url != null;
    }

}
```
- Bean Validation에 관한 내용은 [Validation과 예외처리](/spring-framework/validation.md) 참고

<br>

### 설정에 따라 다른 Bean 사용하기
- 설정 값이 있을 때와 없을 때 동작이 달라져야 하는 경우, 사용하는 쪽에서 매번 null 검사를 하지 않도록 설정 클래스에서 구현체를 골라 Bean으로 등록할 수 있음
  - 기능이 꺼진 경우에는 아무 일도 하지 않는 구현체를 등록함(Null Object Pattern)
```
@Bean
Heartbeat heartbeat(HeartbeatProperties properties) {
    if (!properties.enabled()) {
        return Heartbeat.NONE;
    }
    return new HttpHeartbeat(properties.url(), properties.timeout());
}
```
- 속성 값만으로 등록 여부를 정할 때는 @ConditionalOnProperty를 사용할 수도 있음
```
@Bean
@ConditionalOnProperty(name = "sentinel.notification.webhook-url")
Notifier webhookNotifier(...) {
    ...
}
```
- 비슷하게 @Profile을 사용하면 활성화된 Profile(spring.profiles.active)에 따라 Bean 등록 여부를 정할 수 있음

<br>

#### 참고
- Spring Boot Reference Documentation <Externalized Configuration> - https://docs.spring.io/spring-boot/reference/features/external-config.html
- Spring Boot Reference Documentation <Conditional Annotations> - https://docs.spring.io/spring-boot/reference/features/developing-auto-configuration.html#features.developing-auto-configuration.condition-annotations

#### 배워가는 것들
- @Value로 흩어져 있던 설정을 @ConfigurationProperties와 record로 묶으면, 설정의 구조와 기본값을 한눈에 볼 수 있다는 것을 알게 되었다.
- 타임아웃 같은 값을 Duration으로 받으면 단위 실수를 줄일 수 있다는 점이 유용했다. 숫자만 보고 초인지 밀리초인지 헷갈리는 일이 없어진다.
- 잘못된 설정은 기동 시점에 실패시키는 것이 운영 중에 오류를 만나는 것보다 낫다는 것을 배웠다. 자격증명이 담긴 설정은 오류 메시지와 로그에 값이 남지 않도록 함께 신경 써야 한다.
