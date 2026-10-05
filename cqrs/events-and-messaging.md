# 이벤트와 메시징

### 이벤트를 사용하는 이유
- 주문이 완료되면 포인트를 적립하고, 알림을 보내고, 통계를 갱신하는 것처럼 하나의 작업 뒤에 여러 후속 작업이 이어지는 경우가 많음
- 주문 서비스가 이 작업들을 모두 직접 호출하면 문제가 생김
  - 주문 서비스가 포인트, 알림, 통계 서비스를 모두 알아야 하므로 결합도가 높아짐
  - 알림 발송이 실패하면 주문까지 실패하거나, 느린 후속 작업 때문에 주문 응답이 늦어짐
- 이벤트를 사용하면 주문 서비스는 "주문이 완료되었다"는 사실(Event)만 발행하고, 관심 있는 쪽이 각자 구독하여 처리함
  - 발행하는 쪽은 누가 이벤트를 처리하는지 알 필요가 없음
  - 새로운 후속 작업이 생겨도 발행하는 쪽의 코드를 바꾸지 않고 구독하는 쪽만 추가하면 됨
- 이벤트는 이미 일어난 사실이므로 OrderCompleted처럼 과거형으로 이름을 짓고, 처리에 필요한 최소한의 정보를 Record로 담음

<br>

### Spring의 애플리케이션 이벤트
- 하나의 애플리케이션 안에서 이벤트를 발행하고 처리하는 기능
  - ApplicationEventPublisher.publishEvent()로 발행함
  - @EventListener를 선언한 메서드가 이벤트 타입에 맞춰 호출됨
- @EventListener는 기본적으로 발행한 스레드에서 동기적으로 실행되므로, 리스너에서 예외가 나면 발행한 쪽의 트랜잭션도 롤백됨
```
public record OrderCompleted(Long orderId, Long memberId, BigDecimal amount) {
}


@Service
public class OrderService {

    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public void complete(Long orderId) {
        Order order = orderRepository.findById(orderId).orElseThrow();
        order.complete();
        eventPublisher.publishEvent(new OrderCompleted(order.getId(), order.getMemberId(), order.getAmount()));
    }

}
```

<br>

### @TransactionalEventListener
- 트랜잭션의 결과에 맞춰 리스너를 실행함
  - phase = AFTER_COMMIT(기본값): 트랜잭션이 커밋된 뒤에 실행됨. 롤백되면 실행되지 않음
  - AFTER_ROLLBACK, AFTER_COMPLETION, BEFORE_COMMIT도 지정할 수 있음
- 주문이 롤백되었는데 알림이 발송되는 문제를 막을 수 있어, 외부에 영향을 주는 후속 작업에 사용함
- 유의사항
  - 커밋된 뒤에 실행되므로, 리스너가 실패해도 주문은 이미 완료된 상태임. 실패한 후속 작업을 다시 처리할 방법이 필요함
  - AFTER_COMMIT 리스너 안에서 DB를 변경하려면 새로운 트랜잭션(REQUIRES_NEW)이 필요함
  - 응답 시간에 영향을 주지 않으려면 @Async로 별도 스레드에서 실행함
- 서버가 커밋 직후, 리스너가 실행되기 전에 종료되면 이벤트가 사라짐. 이 문제를 해결하는 것이 Outbox 패턴임
- 트랜잭션 전파 속성은 [트랜잭션 활용](/jpa/transaction.md) 참고
```
@Component
public class PointEventHandler {

    @Async
    @TransactionalEventListener
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void on(OrderCompleted event) {
        pointService.earn(event.memberId(), event.amount());
    }

}
```

<br>

### Outbox 패턴
- DB 변경과 이벤트 발행을 하나의 트랜잭션으로 묶을 수 없다는 문제(Dual Write)를 해결하는 패턴
  - 주문을 저장한 뒤 메시지 브로커에 이벤트를 보내다가 실패하면, 주문은 있는데 이벤트는 없는 상태가 됨
  - 반대로 이벤트를 먼저 보내고 주문 저장이 롤백되면, 없는 주문에 대한 이벤트가 나가게 됨
- 동작 방식
  1. 주문을 저장하는 같은 트랜잭션 안에서 이벤트를 outbox 테이블에 함께 저장함
  2. 별도의 프로세스(Relay)가 outbox 테이블에서 아직 발행하지 않은 이벤트를 읽어 메시지 브로커로 보냄
  3. 발행에 성공하면 해당 행을 발행 완료로 표시함
- DB 커밋에 성공했다면 이벤트도 반드시 남으므로 유실되지 않지만, 발행 후 완료 표시 전에 실패하면 같은 이벤트가 다시 발행될 수 있음(At-least-once)
  - 따라서 받는 쪽은 같은 이벤트를 여러 번 받아도 결과가 같도록 멱등하게 처리해야 함
- Spring Modulith의 이벤트 발행 기록(Event Publication Registry)이 이 패턴을 구현해주며, 관련 내용은 [Spring Modulith](/domain-driven-design/spring-modulith.md) 참고

<br>

### 멱등한 소비자(Idempotent Consumer)
- 메시지는 네트워크 오류나 재시도로 중복 전달될 수 있으므로, 같은 메시지를 여러 번 처리해도 결과가 한 번 처리한 것과 같아야 함
- 구현 방법
  - 이벤트마다 고유한 ID를 담고, 처리한 이벤트 ID를 테이블에 기록하여 이미 처리한 이벤트는 건너뜀
  - 처리 기록 저장과 실제 처리를 같은 트랜잭션으로 묶음
  - "포인트 100점 추가"보다 "주문 1번에 대한 포인트 적립"처럼 결과가 정해지는 형태로 처리하면 중복에 강해짐
- 멱등성의 개념은 [HTTP 메서드](/http/http-method.md) 참고

<br>

### 메시지 브로커
- 서비스 사이에서 메시지를 받아 보관하고 전달하는 중간 시스템
  - 보내는 쪽과 받는 쪽이 동시에 실행 중이지 않아도 되고, 받는 쪽이 느려도 메시지가 쌓여 있다가 처리됨
  - 여러 서버나 여러 서비스가 같은 이벤트를 구독할 수 있음
- Kafka
  - 메시지를 디스크의 로그(Log)에 순서대로 기록하고, 소비자가 읽은 위치(Offset)를 스스로 관리하는 방식
  - Topic을 여러 Partition으로 나누어 처리량을 늘리며, 순서는 같은 Partition 안에서만 보장됨. 순서가 중요한 이벤트는 주문 ID 같은 같은 키로 보내 같은 Partition에 들어가게 함
  - 같은 Consumer Group의 소비자들은 Partition을 나누어 처리하고, 서로 다른 Group은 같은 메시지를 각자 모두 받음
  - 메시지를 읽은 뒤에도 보관 기간 동안 남아 있어, 다시 처음부터 읽거나 새 서비스가 과거 이벤트를 처리할 수 있음
  - Kafka 4.0(2025년 3월)부터 ZooKeeper 없이 KRaft 방식으로만 동작함
- RabbitMQ
  - 메시지를 Exchange가 규칙에 따라 Queue로 보내고, 소비자가 가져가면 Queue에서 사라지는 방식
  - 복잡한 라우팅, 작업 분배, 메시지별 확인(Ack)과 재전달에 적합함
- Redis Pub/Sub, Redis Streams는 별도 브로커 없이 가볍게 사용할 수 있으며, Spring Boot 4.1부터 @RedisListener도 자동 설정됨. Redis에 대한 내용은 [Redis](/cqrs/redis.md) 참고

<br>

### Spring for Apache Kafka
- KafkaTemplate으로 메시지를 보내고, @KafkaListener로 메시지를 받음
- 처리에 실패한 메시지는 정해진 횟수만큼 재시도한 뒤, 별도의 Dead Letter Topic으로 보내 따로 확인하고 다시 처리함
  - 실패한 메시지 하나 때문에 뒤의 메시지들이 계속 막히는 것을 막기 위함
```
@Service
public class OrderEventPublisher {

    private final KafkaTemplate<String, OrderCompleted> kafkaTemplate;

    public void publish(OrderCompleted event) {
        kafkaTemplate.send("order-completed", String.valueOf(event.orderId()), event);
    }

}


@Component
public class NotificationConsumer {

    @KafkaListener(topics = "order-completed", groupId = "notification")
    public void on(OrderCompleted event) {
        notificationService.sendOrderCompleted(event.orderId());
    }

}
```
- 로컬 개발과 테스트에서는 Testcontainers로 Kafka 컨테이너를 띄워 실제와 같은 환경에서 확인함. 관련 내용은 [외부 의존성을 격리한 테스트](/di-spring-test/test-isolation.md) 참고

<br>

### 이벤트 기반 설계 시 고려할 것
- 이벤트로 연결하면 결합도는 낮아지지만, 처리 흐름이 코드에 한눈에 드러나지 않아 추적이 어려워짐
  - 이벤트 목록과 발행자, 구독자를 문서로 정리하고, 분산 추적으로 이벤트가 흘러간 경로를 확인함. 관련 내용은 [관측 가능성](/spring-framework/observability.md) 참고
- 처리 결과가 바로 반영되지 않는 최종적 일관성(Eventual Consistency)을 받아들일 수 있는 작업에만 사용함
- 이벤트의 구조를 바꾸면 구독하는 모든 쪽에 영향을 주므로, 필드를 삭제하거나 의미를 바꾸지 않고 추가하는 방향으로 변경함
- 하나의 애플리케이션 안에서 충분하다면 애플리케이션 이벤트와 Outbox로 시작하고, 서비스가 나뉘거나 처리량이 커질 때 메시지 브로커를 도입함

<br>

#### 참고
- Spring Framework Reference Documentation <Transaction-bound Events> - https://docs.spring.io/spring-framework/reference/data-access/transaction/event.html
- Spring for Apache Kafka Reference - https://docs.spring.io/spring-kafka/reference/
- microservices.io <Pattern: Transactional outbox> - https://microservices.io/patterns/data/transactional-outbox.html
- Apache Kafka 공식문서 <Introduction> - https://kafka.apache.org/intro

#### 배워가는 것들
- @TransactionalEventListener로 커밋 이후에 후속 작업을 실행해도, 서버가 그 사이에 종료되면 이벤트가 사라진다는 한계를 알게 되었다. Outbox 패턴이 왜 필요한지 이 지점에서 이해되었다.
- 메시지는 중복 전달될 수 있다는 전제에서 출발해야 하며, 그래서 받는 쪽이 멱등하게 처리해야 한다는 점이 중요했다.
- 이벤트로 결합도를 낮추는 대신 흐름을 추적하기 어려워진다는 대가가 있다는 것을 알게 되었다. 필요한 만큼만 도입하고 추적 수단을 함께 갖춰야 한다.
