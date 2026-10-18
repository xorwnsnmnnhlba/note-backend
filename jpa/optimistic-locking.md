# 동시 수정과 낙관적 잠금

### 동시 수정 문제
- 여러 요청이 같은 데이터를 동시에 바꾸면, 나중에 저장한 쪽이 먼저 저장한 쪽의 변경을 지워버리는 갱신 손실(Lost Update)이 발생할 수 있음
- JPA에서 특히 주의해야 하는 이유
  - Hibernate는 변경 감지(Dirty Checking)로 UPDATE를 만들 때, 기본적으로 바뀐 필드만이 아니라 엔티티의 모든 컬럼을 SET에 포함함
  - 따라서 A는 enabled만, B는 nextRunAt만 바꿨더라도, 나중에 커밋한 쪽이 자신이 읽어둔 옛 값으로 상대의 컬럼까지 되돌려놓음
- 실제로 발생할 수 있는 예
  - 사용자가 스케줄을 끈 순간, 스케줄러가 같은 스케줄의 다음 실행 시각을 저장하면서 enabled를 다시 true로 되돌림
  - 두 사람이 같은 환경의 변수를 각각 추가하면, 한 사람의 변수가 사라짐
- @DynamicUpdate를 사용하면 바뀐 컬럼만 UPDATE하지만, 같은 컬럼을 동시에 바꾸는 경우나 JSON 컬럼 안의 일부를 바꾸는 경우는 여전히 막지 못함
- 동시 변경을 다루는 기본 방법(조건부 UPDATE 등)은 [트랜잭션 활용](/jpa/transaction.md) 참고

<br>

### 낙관적 잠금(Optimistic Lock)
- "충돌은 드물다"고 보고 잠금 없이 진행하다가, 저장 시점에 충돌을 감지하여 나중 요청을 실패시키는 방식
  - 비관적 잠금(Pessimistic Lock)은 "충돌이 잦다"고 보고 읽을 때부터 행을 잠가(SELECT ... FOR UPDATE) 다른 트랜잭션을 기다리게 함
  - 대부분의 웹 애플리케이션은 같은 행을 동시에 고치는 일이 드물기 때문에, 대기가 없는 낙관적 잠금이 기본 선택지가 됨
- JPA에서는 숫자 필드에 @Version을 선언하면 됨
```
@Version
@Column(nullable = false)
private long version;
```
- 동작 방식
  - 저장할 때마다 version을 1씩 올리고, WHERE 조건에 읽었을 때의 version을 함께 넣음
  - 그사이 다른 트랜잭션이 먼저 저장했다면 version이 달라져 갱신된 행이 0건이 되고, Hibernate가 예외를 발생시킴
```
UPDATE environment
SET name = ?, variables = ?, version = 4
WHERE id = ? AND version = 3;
```
- Spring에서는 ObjectOptimisticLockingFailureException(Spring의 데이터 접근 예외로 변환된 형태)으로 받게 됨
- 기존 테이블에 추가할 때는 마이그레이션으로 NOT NULL DEFAULT 0 컬럼을 추가함. 관련 내용은 [스키마 마이그레이션](/database/schema-migration.md) 참고

<br>

### 충돌을 어떻게 처리할 것인가
- 사용자 요청에서 충돌한 경우
  - 자동으로 재시도하면 상대의 변경을 덮어쓰게 되므로, 409 Conflict로 응답하고 다시 불러온 뒤 수정하도록 안내함
  - 오류 응답 형식은 [Validation과 예외처리](/spring-framework/validation.md)의 ProblemDetail 참고
```
@ExceptionHandler({ ObjectOptimisticLockingFailureException.class, StaleVersionException.class })
ProblemDetail handleOptimisticLock(RuntimeException ex) {
    return ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, "changed elsewhere meanwhile; reload and try again");
}
```
- 백그라운드 작업에서 충돌한 경우
  - 스케줄러처럼 주기적으로 다시 판단하는 작업이라면, 오류로 알리지 않고 다음 주기에 바뀐 내용으로 다시 처리하면 됨
- 충돌이 잦은 데이터를 백그라운드 작업이 매번 갱신한다면(예: 스케줄러가 실행할 때마다 다음 실행 시각을 저장), 사용자의 저장이 계속 실패할 수 있으므로 해당 데이터에는 화면 단위 확인(아래)을 적용하지 않는 등 범위를 조절해야 함

<br>

### 화면 단위의 덮어쓰기
- @Version은 저장 시점에 겹친 트랜잭션만 막아줌
- 아래와 같이 시간 차를 두고 일어나는 덮어쓰기는 막지 못함
  1. A가 수정 화면을 열어둠(version 3을 읽음)
  2. B가 같은 항목을 수정하여 저장함(version 4)
  3. A가 저장하면, 서버는 새로 조회한 version 4의 엔티티에 A의 화면 값을 덮어씀. 트랜잭션은 겹치지 않았으므로 충돌로 감지되지 않음
- 해결 방법: 화면이 읽은 version을 Client가 함께 보내고, 서버가 지금 값과 비교함
  - 조회 응답에 version을 포함함
  - 수정 요청에 version을 받아서 다르면 409로 거부함
  - 기존 Client 호환을 위해 version을 생략하면 확인하지 않도록 할 수도 있음
```
public record UpdateEnvironmentRequest(String name, Map<String, String> variables, Long version) {
}


static void check(String resource, long id, Long expected, long actual) {
    if (expected != null && expected != actual) {
        throw new StaleVersionException(resource, id, expected, actual);
    }
}
```
- Client는 수정 화면을 연 시점의 version을 고정해서 보내야 함
  - 충돌 후 화면을 새로 고치면서 최신 version으로 바꿔버리면, 다시 누른 저장이 옛 입력 내용으로 덮어쓰게 됨
- HTTP 표준 방식으로는 ETag와 If-Match 헤더를 사용하는 조건부 요청이 있으며, 조건이 맞지 않으면 412 Precondition Failed로 응답함. 관련 내용은 [HTTP 캐시와 조건부 요청](/http/http-cache.md) 참고

<br>

### 읽은 뒤 쓰는 코드의 경합 조건(Race Condition)
- "조회 → 확인 → 변경"을 엔티티로 처리하는 코드는, 조회와 변경 사이에 다른 요청이 먼저 바꾸면 잘못 동작할 수 있음
- 일회용 링크를 두 요청이 동시에 사용하는 경우
  - 두 요청 모두 "아직 사용되지 않음"을 읽고 통과한 뒤 각자 사용 기록을 저장하므로, 일회용 링크가 두 번 사용됨
  - 확인과 변경을 하나의 조건부 UPDATE로 합치면, 갱신된 행 수로 하나만 성공시킬 수 있음
```
UPDATE user_token
SET used_at = ?
WHERE token_hash = ? AND used_at IS NULL AND expires_at > ?;
-- 1이면 성공, 0이면 이미 사용되었거나 만료됨
```
- 다른 요청이 먼저 지운 행을 엔티티로 다시 삭제하는 경우
  - 엔티티 삭제는 "1건이 지워져야 한다"고 기대하므로, 0건이면 예외가 발생하여 정상 요청까지 실패함
  - "있으면 지운다"는 의도라면, 삭제된 행 수와 무관한 일괄 삭제 쿼리(DELETE ... WHERE)를 사용함
- 경합 조건 재현 테스트
  - 동시에 실행해보는 방식은 우연에 의존하므로, 특정 순서를 강제하여 재현하는 것이 좋음
  - 예: Repository를 Spy로 감싸 조회 직후에 다른 Thread(다른 트랜잭션)에서 경쟁 작업을 끝내도록 함
```
@MockitoSpyBean
private UserSessionRepository sessions;

doAnswer(invocation -> {
    Object found = invocation.callRealMethod();
    CompletableFuture.runAsync(cleanup::deleteExpired).join();    // 조회 직후 정리 배치가 먼저 삭제
    return found;
}).when(sessions).findByTokenHash(anyString());

assertThat(authService.authenticate(token)).isEmpty();
```

<br>

#### 참고
- Jakarta Persistence Specification <Optimistic Locking and Concurrency> - https://jakarta.ee/specifications/persistence/3.2/jakarta-persistence-spec-3.2#a2079
- Hibernate ORM User Guide <Locking> - https://docs.hibernate.org/orm/7.0/userguide/html_single/#locking
- Vlad Mihalcea <A beginner's guide to database locking and the lost update phenomena> - https://vladmihalcea.com/a-beginners-guide-to-database-locking-and-the-lost-update-phenomena/

#### 배워가는 것들
- Hibernate가 바뀐 필드만이 아니라 모든 컬럼을 UPDATE한다는 점 때문에, 서로 다른 필드를 고쳐도 덮어쓰기가 생길 수 있다는 것을 알게 되었다.
- @Version이 저장 시점에 겹친 요청만 막고, 화면을 열어둔 사이의 변경은 막지 못한다는 점이 인상적이었다. Client가 읽은 version을 함께 보내야 사용자 관점의 덮어쓰기까지 막을 수 있다.
- 경합 조건은 동시에 실행해보는 테스트로는 잘 재현되지 않는다는 것을 배웠다. 순서를 강제하는 테스트를 만들어두면 같은 문제가 다시 생기는 것도 막을 수 있다.
