# 가상 스레드와 동시성 제어

### 플랫폼 스레드(Platform Thread)
- 기존 Java의 Thread는 OS 스레드와 1:1로 대응하는 플랫폼 스레드임
  - 생성 비용이 크고, 스레드마다 큰 메모리(스택)를 차지하므로 수천 개 이상 만들기 어려움
  - 그래서 Thread Pool을 만들어두고 재사용하는 방식을 사용해 왔음
- 웹 애플리케이션의 요청 처리 시간 대부분은 DB 응답이나 외부 API 응답을 기다리는 시간(Blocking I/O)임
  - 기다리는 동안에도 플랫폼 스레드는 OS 스레드를 점유하므로, 동시에 처리할 수 있는 요청 수가 Thread Pool 크기로 제한됨
- 이를 해결하기 위해 WebFlux 같은 비동기·논블로킹 방식이 등장했으나, 코드가 복잡해지고 디버깅이 어렵다는 단점이 있음

<br>

### 가상 스레드(Virtual Thread)
- JDK 21에서 정식 기능으로 추가된, JVM이 관리하는 가벼운 스레드
  - 적은 수의 플랫폼 스레드(Carrier Thread) 위에서 많은 가상 스레드가 번갈아 실행됨
  - 가상 스레드가 I/O를 기다리면 Carrier Thread에서 내려오고(Unmount), 그 자리에 다른 가상 스레드가 올라가 실행됨
  - 생성 비용이 매우 작아 수십만 개도 만들 수 있음
- 기존의 동기·블로킹 방식 코드를 그대로 작성하면서도, 기다리는 시간이 긴 작업을 많이 동시에 처리할 수 있음
```
Thread.ofVirtual().start(() -> {
    ...
});


ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
executor.execute(() -> {
    ...
});
```
- 사용 시 유의사항
  - 가상 스레드는 Pool로 재사용하지 않고, 작업마다 새로 만들어 쓰고 버리는 것이 원칙임
  - CPU를 계속 사용하는 계산 작업에는 이점이 없으며, I/O 대기가 많은 작업에서 효과가 있음
  - 고정(Pinning): 가상 스레드가 Carrier Thread에서 내려오지 못하고 붙잡혀 있는 상태. JDK 21~23에서는 synchronized 블록 안에서 대기하면 발생했으나, JDK 24부터 해결됨. Native 메서드 호출 중에는 여전히 발생함
  - ThreadLocal에 큰 객체를 담으면 가상 스레드 수만큼 메모리를 차지함
- Spring Boot 3.2부터는 spring.threads.virtual.enabled=true로 지정하면 Tomcat 요청 처리와 @Async 등의 작업 실행에 가상 스레드를 사용함

<br>

### 오래 걸리는 작업을 백그라운드로 실행하기
- 요청을 받은 Thread에서 작업이 끝날 때까지 기다렸다가 응답하면, 작업이 길수록 클라이언트의 응답 대기 시간이 길어지고 타임아웃이 발생할 수 있음
- 작업을 접수만 하고 바로 응답한 뒤, 실제 작업은 다른 Thread에서 진행하는 방식으로 해결할 수 있음
  - 응답 상태 코드는 "요청을 받았으나 처리가 끝나지 않았음"을 의미하는 202 Accepted를 사용함
  - 클라이언트는 응답으로 받은 작업 ID로 상태를 반복 조회(Polling)하여 진행 상황을 확인함
  - 관련 내용은 [Polling과 Webhook](/http/polling-webhook.md) 참고
- 작업 실행을 인터페이스로 분리해두면, 테스트에서는 호출한 Thread에서 바로 실행하는 구현으로 바꿔 끼워 결과를 즉시 검증할 수 있음
```
public interface BackgroundRuns {

    void submit(Runnable run);

}


@Component
class VirtualThreadBackgroundRuns implements BackgroundRuns {

    private final ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

    @Override
    public void submit(Runnable run) {
        executor.execute(() -> {
            try {
                run.run();
            }
            catch (RuntimeException ex) {
                log.error("Background run failed", ex);
            }
        });
    }

    @PreDestroy
    void shutdown() {
        executor.shutdownNow();
    }

}
```
- 다른 Thread에서 발생한 예외는 요청 Thread로 전달되지 않으므로, 작업 안에서 직접 잡아서 기록해야 함
- 애플리케이션이 종료되면 진행 중이던 작업이 중단되므로, 재시작 시 "진행 중" 상태로 남은 작업을 정리하는 처리가 필요함

<br>

### 동시 실행 수 제한
- 가상 스레드는 거의 무제한으로 만들 수 있지만, 그 뒤에 있는 자원(DB 커넥션, 외부 API의 처리량)은 한정되어 있음
  - Thread 수가 아니라 실제로 보호해야 하는 자원을 기준으로 동시 실행 수를 제한해야 함
- Semaphore
  - 정해진 개수의 허가(Permit)를 나눠주어, 동시에 진입할 수 있는 수를 제한하는 동기화 도구
  - acquire(): 허가를 얻을 때까지 기다림
  - tryAcquire(): 허가가 없으면 기다리지 않고 바로 false를 반환함
  - release(): 허가를 반납함. 작업이 실패하더라도 반드시 반납해야 하므로 finally에서 호출함
```
private final Semaphore permits = new Semaphore(4);

public long start(...) {
    if (!permits.tryAcquire()) {
        throw new TooManyRunsException(4);    // 429 Too Many Requests
    }
    try {
        long runId = prepare(...);
        backgroundRuns.submit(() -> {
            try {
                execute(runId);
            }
            finally {
                permits.release();
            }
        });
        return runId;
    }
    catch (RuntimeException ex) {
        permits.release();    // 준비 단계에서 실패하면 바로 반납
        throw ex;
    }
}
```
- 한도를 넘은 요청을 기다리게 하지 않고 바로 거절해야 할 때는 tryAcquire()와 429 Too Many Requests 응답을 함께 사용함
- Spring Framework 7.0부터는 @ConcurrencyLimit으로 메서드 단위의 동시 호출 수를 선언적으로 제한할 수 있으며, 관련 내용은 [타임아웃과 재시도](/spring-framework/resilience.md) 참고

<br>

### 동시성 관련 유틸리티
- volatile
  - 한 Thread가 바꾼 값을 다른 Thread가 바로 볼 수 있도록 가시성(Visibility)을 보장함
  - count++처럼 읽고 쓰는 두 단계가 필요한 연산의 원자성은 보장하지 않음
- ConcurrentHashMap
  - 여러 Thread가 동시에 사용해도 안전한 Map
  - compute(), merge()로 "읽고, 계산하고, 쓰는" 작업을 원자적으로 수행할 수 있음
```
attempts.compute(email, (key, previous) -> previous == null ? 1 : previous + 1);
```
- 이런 메모리 기반 상태는 인스턴스마다 따로 존재하고 재시작하면 사라지므로, 여러 인스턴스로 확장할 때는 Redis 같은 공유 저장소가 필요함

<br>

#### 참고
- JEP 444 <Virtual Threads> - https://openjdk.org/jeps/444
- JEP 491 <Synchronize Virtual Threads without Pinning> - https://openjdk.org/jeps/491
- Oracle Java Documentation <Virtual Threads> - https://docs.oracle.com/en/java/javase/25/core/virtual-threads.html
- Spring Boot Reference Documentation <Virtual Threads> - https://docs.spring.io/spring-boot/reference/features/spring-application.html#features.spring-application.virtual-threads

#### 배워가는 것들
- 가상 스레드 덕분에 익숙한 동기 방식 코드를 유지하면서도 기다리는 작업을 많이 동시에 처리할 수 있다는 것을 알게 되었다. 복잡한 리액티브 코드를 쓰지 않아도 되는 경우가 늘어날 것 같다.
- 스레드를 마음껏 만들 수 있게 되었다고 해서 동시 실행을 제한하지 않아도 되는 것은 아니라는 점이 인상적이었다. 보호해야 하는 것은 스레드가 아니라 DB 커넥션이나 외부 API 같은 실제 자원이다.
- 오래 걸리는 작업은 202로 접수만 하고 상태를 조회하게 하는 패턴을 익힐 수 있었다. 응답 대기 한도를 넘는 작업이 있다면 처음부터 이 구조로 설계하는 것이 좋겠다.
