# 트랜잭션(Transaction) 활용

### 트랜잭션과 ACID
- 트랜잭션은 더 이상 나눌 수 없는 하나의 작업 단위로써, 아래 네 가지 성질(ACID)을 보장해야 함
  - 원자성(Atomicity): 트랜잭션 안의 작업은 모두 반영되거나 모두 반영되지 않아야 함
  - 일관성(Consistency): 트랜잭션이 끝난 뒤에도 데이터는 정의된 규칙(제약조건 등)을 만족해야 함
  - 격리성(Isolation): 동시에 수행되는 트랜잭션이 서로의 중간 상태를 볼 수 없어야 함
  - 지속성(Durability): 커밋된 결과는 장애가 발생해도 유지되어야 함
- @Transactional의 기본 옵션은 [Spring Data JPA](/jpa/spring-data-jpa.md) 참고

<br>

### @Transactional의 동작 원리
- Spring은 @Transactional이 선언된 Bean을 Proxy로 감싸서, 메서드 호출 전에 트랜잭션을 시작하고 호출이 끝나면 커밋 또는 롤백함
  - Spring AOP를 이용한 방식이며, 관련 내용은 [Dependency Injection](/di-spring-test/dependency-injection.md) 참고
- 롤백 규칙
  - 기본적으로 RuntimeException(Unchecked Exception)과 Error가 발생하면 롤백함
  - Checked Exception이 발생하면 롤백하지 않고 커밋함
  - rollbackFor, noRollbackFor 속성으로 규칙을 바꿀 수 있음
- Proxy 방식이기 때문에 주의해야 하는 점
  - 같은 클래스 안에서 다른 메서드를 호출하면(Self-Invocation) Proxy를 거치지 않으므로, 호출된 메서드의 @Transactional이 적용되지 않음
  - private 메서드에는 적용되지 않음

<br>

### 전파 속성(Propagation)
- 이미 트랜잭션이 진행 중일 때, 트랜잭션이 필요한 메서드가 호출되면 어떻게 할지 정하는 속성
  - REQUIRED(기본값): 진행 중인 트랜잭션이 있으면 참여하고, 없으면 새로 시작함
  - REQUIRES_NEW: 진행 중인 트랜잭션을 잠시 멈추고, 항상 새로운 트랜잭션을 시작함
  - MANDATORY: 진행 중인 트랜잭션에 참여하며, 없으면 예외가 발생함
  - SUPPORTS: 진행 중인 트랜잭션이 있으면 참여하고, 없으면 트랜잭션 없이 수행함
  - NOT_SUPPORTED: 진행 중인 트랜잭션을 잠시 멈추고, 트랜잭션 없이 수행함
  - NEVER: 트랜잭션 없이 수행하며, 진행 중인 트랜잭션이 있으면 예외가 발생함
  - NESTED: 진행 중인 트랜잭션 안에 Savepoint를 만들어, 내부 작업만 부분 롤백할 수 있도록 함
- MANDATORY 활용 예
  - 변경 이력처럼 "반드시 본래 변경과 같은 트랜잭션에서 기록되어야 하는" 작업에 지정하면, 트랜잭션 밖에서 실수로 호출하는 것을 막을 수 있음
  - 본래 변경이 롤백되면 이력도 함께 롤백되므로, 거부된 변경의 이력이 남지 않음
```
@Transactional(propagation = Propagation.MANDATORY)
public void record(ChangeLog log) {
    changeLogs.save(log);
}
```
- REQUIRES_NEW 활용 예
  - 알림 발송 이력처럼 부가적인 기록의 실패가 본래 작업을 되돌리면 안 될 때 사용함
  - 바깥 트랜잭션이 커넥션을 가진 채로 새 커넥션을 하나 더 사용하므로, 커넥션 풀이 작은 환경에서 동시에 많이 호출되면 커넥션 고갈(Deadlock)이 발생할 수 있음

<br>

### 읽기 전용 트랜잭션
- @Transactional(readOnly = true)는 단순히 표시만 하는 것이 아니라 실제로 동작에 영향을 줌
  - Hibernate는 읽기 전용 세션에서 엔티티의 스냅샷을 보관하지 않고 변경 감지(Dirty Checking)와 Flush를 생략하므로, 메모리와 CPU 사용이 줄어듦
  - JDBC 커넥션에도 읽기 전용 힌트가 전달되며, 읽기 전용 복제본(Replica)으로 요청을 보내는 구성의 기준으로도 사용됨
- 조회만 수행하는 서비스는 클래스 레벨에 readOnly = true를 선언하고, 쓰기 메서드에만 @Transactional을 따로 붙이는 방식을 많이 사용함

<br>

### 프로그래밍 방식 트랜잭션(TransactionTemplate)
- @Transactional은 메서드 단위로 트랜잭션을 지정하므로, 한 메서드 안에서 트랜잭션을 여러 번 나눠야 하는 경우에는 사용하기 어려움
  - 예: 스위트 실행 준비 → 케이스마다 외부 API 호출 및 결과 저장 → 완료 기록
  - 메서드 전체를 하나의 트랜잭션으로 묶으면, 외부 API 응답을 기다리는 동안 DB 커넥션을 계속 붙잡고 있게 됨
- TransactionTemplate을 사용하면 원하는 코드 블록만 트랜잭션으로 묶을 수 있음
  - execute(): 결과를 반환하는 작업
  - executeWithoutResult(): 결과가 없는 작업
  - 블록 안에서 예외가 발생하면 롤백하며, status.setRollbackOnly()로 직접 롤백을 지정할 수도 있음
```
@Component
public class Transactions {

    private final TransactionTemplate write;

    private final TransactionTemplate readOnly;

    Transactions(PlatformTransactionManager transactionManager) {
        this.write = new TransactionTemplate(transactionManager);
        this.readOnly = new TransactionTemplate(transactionManager);
        this.readOnly.setReadOnly(true);
    }

    public <T> T write(TransactionCallback<T> action) {
        return write.execute(action);
    }

    public <T> T readOnly(TransactionCallback<T> action) {
        return readOnly.execute(action);
    }

}


long runId = transactions.write(status -> runs.save(new TestRun(...)).getId());

for (TestCase testCase : cases) {
    HttpExchange exchange = executor.execute(request);    // 트랜잭션 밖에서 외부 호출
    transactions.writeWithoutResult(status -> results.save(new TestResult(runId, exchange)));
}
```
- 외부 API 호출, 파일 처리, 대기(sleep)처럼 오래 걸리는 작업은 트랜잭션 밖에서 수행하고, DB 작업만 짧은 트랜잭션으로 묶는 것이 좋음

<br>

### OSIV(Open Session In View)
- 영속성 컨텍스트(EntityManager)를 HTTP 요청이 끝날 때까지 열어두는 방식
  - Controller나 View에서도 지연 로딩(Lazy Loading)을 사용할 수 있다는 장점이 있음
  - 대신 요청이 끝날 때까지 DB 커넥션을 붙잡고 있으므로, 트래픽이 많거나 응답이 느린 요청이 있으면 커넥션 풀이 고갈될 수 있음
- Spring Boot는 spring.jpa.open-in-view의 기본값을 true로 두며, 따로 지정하지 않으면 기동 시 경고 로그를 출력함
- false로 지정하면 트랜잭션이 끝날 때 영속성 컨텍스트도 닫히므로, 필요한 데이터는 서비스 계층에서 DTO로 변환하여 반환해야 함
  - 트랜잭션 밖에서 지연 로딩을 시도하면 LazyInitializationException이 발생함
```
spring:
  jpa:
    open-in-view: false
```

<br>

### 동시 변경과 갱신 손실(Lost Update)
- 두 트랜잭션이 같은 행을 읽은 뒤 각자 값을 바꿔 저장하면, 나중에 저장한 쪽이 먼저 저장한 쪽의 변경을 덮어씀
  - 예: 실행 Thread는 통과 건수를 올리고, 사용자는 같은 실행을 중지 상태로 바꾸는 경우
- 해결 방법
  - 낙관적 락(Optimistic Lock): @Version 컬럼으로 충돌을 감지하여 나중에 저장한 쪽을 실패시킴. 관련 내용은 [Tactical Design](/domain-driven-design/tactical-design.md) 참고
  - 비관적 락(Pessimistic Lock): SELECT ... FOR UPDATE로 행을 잠가 다른 트랜잭션이 기다리게 함
  - 조건부 UPDATE: 필요한 컬럼만, 현재 상태를 조건으로 걸어서 변경함
```
UPDATE test_run SET passed_count = passed_count + 1 WHERE id = ?;

UPDATE test_run SET status = 'CANCELED', finished_at = ?
WHERE id = ? AND status = 'RUNNING';
```
- 조건부 UPDATE는 변경된 행 수를 반환하므로, 0이면 "이미 다른 쪽이 상태를 바꿨다"는 것을 알 수 있음
  - 관련 구현 방법은 [QueryDSL](/jpa/querydsl.md)의 일괄 변경 참고

<br>

#### 참고
- Spring Framework Reference Documentation <Transaction Management> - https://docs.spring.io/spring-framework/reference/data-access/transaction.html
- Spring Framework Reference Documentation <Transaction Propagation> - https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html
- Spring Boot Reference Documentation <Data Access> - https://docs.spring.io/spring-boot/how-to/data-access.html

#### 배워가는 것들
- @Transactional이 Proxy 기반이라 같은 클래스 안의 호출에서는 동작하지 않는다는 점을 다시 확인할 수 있었다. 트랜잭션 경계가 중요한 코드라면 애노테이션이 실제로 적용되는지 의심해봐야 한다.
- 외부 API 호출을 트랜잭션 안에 두면 응답을 기다리는 동안 커넥션을 붙잡게 된다는 것을 알게 되었다. TransactionTemplate으로 DB 작업만 짧게 묶는 방식이 커넥션 풀이 작은 환경에서 특히 중요하다.
- 엔티티 전체를 저장하는 방식이 동시 변경 상황에서는 다른 쪽의 변경을 덮어쓸 수 있다는 점이 인상적이었다. 필요한 컬럼만 조건을 걸어 바꾸는 UPDATE도 동시성 문제를 푸는 하나의 방법이다.
