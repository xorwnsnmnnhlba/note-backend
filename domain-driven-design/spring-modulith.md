# Spring Modulith

### 모듈러 모놀리스(Modular Monolith)
- 하나의 애플리케이션으로 배포하되, 내부를 업무 영역별 모듈로 명확하게 나누고 모듈 사이의 의존을 규칙으로 제한하는 구조
- 비교
  - 일반적인 모놀리스: 배포는 단순하지만, 시간이 지나면서 모든 코드가 서로를 참조하여 경계가 사라지기 쉬움
  - 마이크로서비스: 서비스별로 독립 배포할 수 있지만, 네트워크 통신, 분산 트랜잭션, 운영 비용이 크게 늘어남
  - 모듈러 모놀리스: 하나로 배포하면서도 모듈 경계를 지켜, 필요해지면 특정 모듈을 별도 서비스로 분리하기 쉬움
- 모듈은 DDD의 Bounded Context와 대응시키는 경우가 많으며, 관련 내용은 [Strategic Design](/domain-driven-design/strategic-design.md) 참고

<br>

### Spring Modulith
- Spring Boot 애플리케이션을 모듈러 모놀리스로 만들고, 모듈 경계를 검증하는 도구를 제공하는 Spring 프로젝트
  - 2025년 11월 Spring Boot 4.x에 맞춘 2.0이 출시됨
- 애플리케이션 모듈(Application Module)
  - 메인 클래스가 있는 패키지의 바로 아래 패키지 하나하나를 모듈로 간주함
  - 모듈 최상위 패키지의 public 타입이 다른 모듈에 공개하는 API이고, 하위 패키지의 타입은 모듈 내부 구현으로 취급함
```
com.example.shop
├── ShopApplication.java
├── order                  # order 모듈
│   ├── OrderService.java  # 공개 API
│   ├── OrderCompleted.java
│   └── internal           # 내부 구현. 다른 모듈에서 참조하면 안 됨
│       └── OrderRepository.java
├── inventory              # inventory 모듈
└── notification           # notification 모듈
```
```
build.gradle

dependencies {
    implementation 'org.springframework.modulith:spring-modulith-starter-core'
    testImplementation 'org.springframework.modulith:spring-modulith-starter-test'
}
```

<br>

### 모듈 구조 검증하기
- ApplicationModules.of(메인 클래스).verify()로 아래 규칙을 테스트에서 검사함
  - 다른 모듈의 내부 패키지를 참조하지 않는가
  - 모듈 사이에 순환 의존이 없는가
- 규칙을 어기는 코드가 추가되면 테스트가 실패하므로, 시간이 지나도 모듈 경계가 무너지지 않음
- Documenter로 모듈 구조와 의존 관계를 C4, PlantUML 다이어그램으로 생성하여 문서로 남길 수 있음
```
class ModularityTests {

    ApplicationModules modules = ApplicationModules.of(ShopApplication.class);

    @Test
    void verifiesModularStructure() {
        modules.verify();
    }

    @Test
    void writeDocumentation() {
        new Documenter(modules).writeDocumentation();
    }

}
```

<br>

### 모듈 사이의 통신은 이벤트로
- 한 모듈이 다른 모듈의 서비스를 직접 호출하면 모듈끼리 강하게 묶이므로, 이벤트를 발행하고 다른 모듈이 구독하는 방식을 권장함
- @ApplicationModuleListener
  - @Async, @Transactional(propagation = REQUIRES_NEW), @TransactionalEventListener를 합친 애노테이션
  - 이벤트를 발행한 트랜잭션이 커밋된 뒤, 별도 스레드와 새 트랜잭션에서 처리함
  - 리스너가 실패해도 발행한 쪽의 작업에는 영향을 주지 않음
- 애플리케이션 이벤트와 트랜잭션 이벤트의 개념은 [이벤트와 메시징](/cqrs/events-and-messaging.md) 참고
```
package com.example.shop.inventory;

@Component
class InventoryEventHandler {

    @ApplicationModuleListener
    void on(OrderCompleted event) {
        inventoryService.decrease(event.orderId());
    }

}
```

<br>

### 이벤트 발행 기록(Event Publication Registry)
- 이벤트가 발행되면, 그 이벤트를 받을 트랜잭션 리스너마다 발행 기록을 원래의 업무 트랜잭션 안에서 함께 저장함
  - 리스너가 성공하면 기록을 완료로 표시하고, 실패하거나 서버가 종료되면 미완료 기록이 남음
  - 남은 기록을 다시 처리할 수 있으므로 이벤트가 유실되지 않으며, Outbox 패턴과 같은 효과를 냄
- 2.0부터 발행 기록이 처리 대기, 처리 중, 완료, 실패 상태로 구분되어, 실패한 이벤트만 다시 제출하고 처리 중인 것을 실패로 잘못 판단하지 않게 됨
- 저장소는 JDBC, JPA, MongoDB 등을 선택하며, spring-modulith-starter-jdbc 같은 Starter를 추가함
- 완료된 기록은 쌓이지 않도록 일정 기간이 지나면 정리하는 설정을 함께 둠

<br>

### 이벤트 외부로 보내기(Externalization)
- 모듈 사이의 이벤트 중 다른 서비스도 알아야 하는 이벤트는 @Externalized를 선언하여 Kafka, RabbitMQ 같은 메시지 브로커로 보낼 수 있음
  - 내부 이벤트와 같은 코드로 발행하면서, 발행 기록 덕분에 브로커 전송도 유실되지 않음
- 나중에 모듈을 별도 서비스로 분리할 때, 모듈 사이의 통신을 이미 이벤트로 해두었다면 전송 방식만 브로커로 바꾸면 됨

<br>

### 모듈 단위 테스트
- @ApplicationModuleTest를 선언하면 테스트 대상 모듈의 Bean만 띄워 통합 테스트를 실행함
  - 전체 애플리케이션을 띄우는 것보다 빠르고, 다른 모듈에 의존하고 있다면 그 사실이 드러남
- Scenario API로 "이 작업을 실행하면 이런 이벤트가 발행된다", "이 이벤트가 오면 이런 결과가 생긴다"를 테스트할 수 있음
```
@ApplicationModuleTest
class OrderModuleTests {

    @Test
    void publishesOrderCompleted(Scenario scenario) {
        scenario.stimulate(() -> orderService.complete(1L))
                .andWaitForEventOfType(OrderCompleted.class)
                .toArriveAndVerify(event -> assertThat(event.orderId()).isEqualTo(1L));
    }

}
```

<br>

#### 참고
- Spring Modulith Reference Documentation - https://docs.spring.io/spring-modulith/reference/
- Spring Modulith Reference Documentation <Working with Application Events> - https://docs.spring.io/spring-modulith/reference/events.html
- Spring 공식블로그 <Spring Modulith 2.0 GA> - https://spring.io/blog/2025/11/21/spring-modulith-2-0-ga-1-4-5-and-1-3-11-released/

#### 배워가는 것들
- 마이크로서비스로 바로 가지 않고도, 하나의 애플리케이션 안에서 모듈 경계를 테스트로 강제할 수 있다는 점이 인상 깊었다. 경계는 규칙을 문서로 남기는 것만으로는 지켜지지 않는다.
- 모듈 사이를 이벤트로 연결해두면, 나중에 모듈을 서비스로 분리할 때 전송 방식만 바꾸면 된다는 것을 알게 되었다.
- 이벤트 발행 기록이 업무 트랜잭션과 함께 저장되어 Outbox 패턴과 같은 효과를 낸다는 점에서, 앞서 정리한 이벤트 유실 문제를 프레임워크 차원에서 해결해준다는 것을 이해할 수 있었다.
