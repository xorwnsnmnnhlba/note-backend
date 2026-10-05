# 스케줄링(Scheduling)

### 스케줄링
- 정해진 시각이나 일정한 간격으로 작업을 자동 실행하는 것
  - 예: 매일 새벽 오래된 데이터 정리, 매주 월요일 요약 알림 발송, 1분마다 처리할 작업 확인
- Spring은 별도 라이브러리 없이 @Scheduled로 간단한 스케줄링을 지원함
- 스케줄 자체를 사용자가 등록하고 바꿀 수 있어야 하거나, 재시작 후 놓친 실행을 복구해야 한다면 Quartz 같은 스케줄러 라이브러리나 DB 기반 스케줄러를 고려함

<br>

### @EnableScheduling과 @Scheduled
- @Configuration 클래스에 @EnableScheduling을 선언하면, Bean의 @Scheduled 메서드가 스케줄에 등록됨
- @Scheduled의 주요 속성
  - fixedRate: 이전 실행의 시작 시점부터 일정 간격마다 실행함
  - fixedDelay: 이전 실행이 끝난 시점부터 일정 간격 뒤에 실행함. 작업이 오래 걸려도 실행이 겹치지 않음
  - initialDelay: 첫 실행 전 대기 시간
  - cron: cron 표현식에 맞는 시각에 실행함
  - zone: cron 표현식을 해석할 시간대
  - fixedRateString, fixedDelayString처럼 String이 붙은 속성에는 "${...}" Placeholder로 설정 값을 사용할 수 있음
```
@Configuration
@EnableScheduling
public class SchedulingConfig {

}


@Component
public class RawResultRetention {

    @Scheduled(cron = "0 30 3 * * *", zone = "Asia/Seoul")
    public void purgeExpired() {
        ...
    }

}
```
- 스케줄 메서드는 파라미터가 없고 반환값을 사용하지 않음

<br>

### Cron 표현식
- 실행 시각을 필드별로 표현하는 문자열
- Spring의 cron은 초 필드가 포함된 6자리 형식임
  - 초 분 시 일 월 요일
  - 0 0 9 * * MON: 매주 월요일 09:00:00
  - 0 */10 * * * *: 10분마다
  - 0 30 9 * * 1-5: 평일 09:30
- Linux crontab이나 대부분의 웹 화면 입력기는 초 필드가 없는 5자리 형식(분 시 일 월 요일)을 사용함
  - 두 형식을 함께 다룬다면, 5자리 값 앞에 "0 "을 붙여서 Spring 형식으로 해석하는 식으로 맞춰줘야 함
- 특수 문자
  - *: 모든 값, ?: 값 없음(일과 요일 중 하나에 사용)
  - -: 범위, ,: 목록, /: 간격
  - L: 마지막(L은 그 달의 마지막 날), W: 가장 가까운 평일, #: n번째 요일(MON#1은 첫째 주 월요일)
- @daily, @weekly, @monthly 같은 매크로도 사용할 수 있음
- CronExpression 클래스로 직접 해석하여 다음 실행 시각을 계산할 수 있음
```
CronExpression cron = CronExpression.parse("0 0 9 * * MON");
ZonedDateTime next = cron.next(ZonedDateTime.now(ZoneId.of("Asia/Seoul")));
```

<br>

### 시간대(Time Zone) 주의사항
- zone을 지정하지 않으면 cron은 JVM의 기본 시간대로 해석됨
- Docker 컨테이너의 기본 시간대는 대부분 UTC이므로, 로컬에서 09:00에 실행되던 작업이 컨테이너에서는 한국 시각 18:00에 실행되는 문제가 생김
- 따라서 cron의 시간대는 설정으로 명시하고, 컨테이너 환경변수(TZ)와 무관하게 동작하도록 하는 것이 안전함

<br>

### 설정 값으로 스케줄 등록하기
- 실행 간격이나 사용 여부를 설정으로 바꾸고 싶다면, SchedulingConfigurer를 구현하여 코드로 스케줄을 등록할 수 있음
- 설정 값에 따라 등록 자체를 하지 않을 수 있으며, cron의 시간대도 설정 값으로 지정할 수 있음
```
@Configuration
@EnableScheduling
@RequiredArgsConstructor
class SchedulingConfig implements SchedulingConfigurer {

    private final ScheduleProperties properties;

    private final SchedulePoller poller;

    private final RawResultRetention retention;

    @Override
    public void configureTasks(ScheduledTaskRegistrar registrar) {
        if (properties.enabled()) {
            registrar.addFixedDelayTask(poller::poll, properties.pollInterval());
        }
        registrar.addCronTask(new CronTask(retention::purgeExpired,
                new CronTrigger("0 30 3 * * *", properties.zone())));
    }

}
```

<br>

### 스케줄러 Thread
- 별도 설정이 없으면 Spring Boot는 Thread가 하나뿐인 TaskScheduler를 사용함
  - 한 작업이 오래 걸리면 다른 작업들이 그만큼 늦게 실행됨
  - spring.task.scheduling.pool.size로 Thread 수를 늘릴 수 있음
- 스케줄 작업에서 예외가 발생하면 로그만 남기고 다음 주기에 다시 실행되지만, 작업 안에서 여러 대상을 처리한다면 대상별로 예외를 잡아서 하나의 실패가 나머지를 막지 않도록 해야 함

<br>

### 놓친 실행(Misfire)과 재시작 복구
- @Scheduled는 실행 시각을 메모리에서만 관리하므로, 그 시각에 애플리케이션이 꺼져 있었다면 해당 실행은 그냥 건너뛰어짐
  - 노트북이 절전 상태였거나 재배포 중이었다면, 매주 한 번 보내는 알림이 한 주 통째로 누락될 수 있음
- 놓친 실행을 처리하는 정책
  - 놓친 횟수만큼 몰아서 실행함
  - 놓친 실행은 한 번만 수행하고, 다음 시각은 현재 시각을 기준으로 다시 계산함(Quartz의 FireAndProceed)
  - 놓친 실행은 건너뜀
- 정기 작업을 cron 작업으로 등록하지 않고, 짧은 주기로 "예정 시각이 지났는데 아직 처리하지 않았는가"를 확인하는 방식으로 바꾸면 켜진 직후에 놓친 작업을 처리할 수 있음
  - 예정 시각마다 처리 기록을 DB에 남기고 유니크 제약조건을 걸면, 재시작해도 같은 작업이 두 번 실행되지 않음

<br>

### DB 폴링 스케줄러
- 사용자가 화면에서 등록하는 스케줄처럼 실행 시각이 데이터로 관리되어야 할 때 사용할 수 있는 방식
  - 스케줄마다 다음 실행 시각(next_run_at)을 컬럼으로 저장함
  - 짧은 주기(예: 60초)로 enabled = true AND next_run_at <= 현재 시각인 스케줄을 조회하여 실행함
  - 재시작해도 next_run_at이 DB에 남아 있으므로, 켜진 직후 첫 폴링에서 놓친 실행을 처리함
- 실행하기 전에 next_run_at을 먼저 다음 시각으로 갱신해야 함
  - 실행 도중 장애가 발생해도 같은 스케줄이 계속 반복 실행되지 않음
- Quartz도 JDBC JobStore로 같은 기능을 제공하지만, 전용 테이블 여러 개와 스키마 관리가 필요하므로 요구사항이 단순하다면 직접 구현하는 편이 가벼울 수 있음
- 여러 인스턴스에서 실행하는 경우의 주의사항
  - 모든 인스턴스가 같은 스케줄을 찾아 동시에 실행할 수 있음
  - SELECT ... FOR UPDATE SKIP LOCKED로 하나의 인스턴스만 행을 가져가도록 하거나, ShedLock 같은 라이브러리로 작업 단위 잠금을 걸어야 함

<br>

### 시간에 의존하는 코드 테스트하기
- 코드 안에서 Instant.now()를 직접 호출하면, 테스트에서 "1시간 뒤", "다음 주 월요일" 같은 상황을 만들기 어려움
- java.time.Clock을 Bean으로 등록하고 주입받아 사용하면, 테스트에서는 원하는 시각을 돌려주는 Clock으로 바꿔 끼울 수 있음
```
@Bean
Clock clock() {
    return Clock.systemUTC();
}


Instant now = clock.instant();
```
- 관련 테스트 방법은 [외부 의존성을 격리한 테스트](/di-spring-test/test-isolation.md) 참고

<br>

#### 참고
- Spring Framework Reference Documentation <Task Execution and Scheduling> - https://docs.spring.io/spring-framework/reference/integration/scheduling.html
- Spring Boot Reference Documentation <Task Execution and Scheduling> - https://docs.spring.io/spring-boot/reference/features/task-execution-and-scheduling.html
- Quartz Scheduler Documentation <Misfire Instructions> - https://www.quartz-scheduler.org/documentation/

#### 배워가는 것들
- Spring의 cron은 초 필드가 있는 6자리라서 흔히 보던 5자리 cron과 다르다는 것을 알게 되었다. 두 형식이 섞이는 곳에서는 변환 규칙을 한곳에 모아둬야 한다.
- 컨테이너의 기본 시간대가 UTC라서 cron이 9시간 어긋날 수 있다는 점이 인상적이었다. 시간대는 환경에 맡기지 말고 설정으로 명시해야겠다.
- @Scheduled는 꺼져 있던 동안의 실행을 복구해주지 않는다는 것을 배웠다. 놓치면 안 되는 작업이라면 실행 시각과 처리 기록을 DB에 두고, 켜진 뒤에 확인하는 구조가 필요하다.
