# 최신 Java 문법

### Java의 릴리스 주기
- Java는 6개월마다 새 버전이 출시되며, 2년마다 출시되는 버전이 LTS(Long Term Support)로 지정되어 장기간 지원됨
  - 최근 LTS: Java 17(2021년), Java 21(2023년), Java 25(2025년 9월)
  - 2026년 9월 출시된 Java 27은 LTS가 아닌 단기 지원 버전이며, G1 GC가 모든 환경의 기본값이 되고 TLS 1.3의 양자 내성 하이브리드 키 교환이 추가됨
- 새 기능은 대부분 프리뷰(Preview)로 먼저 공개되어 여러 버전에 걸쳐 다듬어진 뒤 정식 기능이 됨
  - 프리뷰 기능은 --enable-preview 옵션을 켜야 사용할 수 있고, 다음 버전에서 바뀔 수 있으므로 운영 코드에는 정식 기능만 사용하는 것이 좋음
- Spring Boot 4.x는 Java 17 이상을 요구하며, Java 25 사용이 권장됨

<br>

### var와 텍스트 블록
- var(Java 10): 지역 변수의 타입을 초기값으로부터 추론함
  - 타입이 오른쪽에 명확히 드러날 때 사용하고, 반환 타입을 알기 어려운 메서드 호출 결과에는 사용을 피함
- 텍스트 블록(Java 15): """로 감싸 여러 줄 문자열을 그대로 작성함. SQL, JSON을 코드에 넣을 때 유용함
```
var orders = new ArrayList<Order>();

String query = """
        SELECT id, name
        FROM product
        WHERE category = ?
        """;
```

<br>

### Record
- 값을 담는 불변 객체를 간결하게 선언하는 문법으로, Java 16에서 정식 기능이 됨
  - 생성자, 필드 접근 메서드, equals, hashCode, toString을 자동으로 만들어줌
  - 모든 필드는 final이며, 다른 클래스를 상속할 수 없음
- DTO, 값 객체, 메서드의 여러 반환값을 묶을 때 사용함. 관련 내용은 [DTO](/dto-json-cors/dto.md) 참고
- 압축 생성자(Compact Constructor)에서 값을 검증할 수 있음
```
public record Money(BigDecimal amount, String currency) {

    public Money {
        if (amount.signum() < 0) {
            throw new IllegalArgumentException("금액은 음수일 수 없음");
        }
    }

}
```

<br>

### Sealed Class와 Interface
- 상속하거나 구현할 수 있는 타입을 permits로 제한하는 문법으로, Java 17에서 정식 기능이 됨
- 결과나 상태처럼 가능한 경우가 정해진 타입을 표현할 때 사용함
  - 컴파일러가 모든 하위 타입을 알 수 있으므로, switch에서 빠뜨린 경우를 찾아줌
```
public sealed interface PaymentResult permits Approved, Declined, Pending {
}

public record Approved(String transactionId) implements PaymentResult {
}

public record Declined(String reason) implements PaymentResult {
}

public record Pending() implements PaymentResult {
}
```

<br>

### 패턴 매칭
- instanceof 패턴 매칭(Java 16): 타입 검사와 형변환을 한 번에 처리함
- switch 표현식(Java 14): switch가 값을 반환하며, 화살표(->) 문법으로 break 없이 작성함
- switch 패턴 매칭(Java 21): case에 타입 패턴을 사용하며, when으로 조건을 덧붙일 수 있음
  - sealed 타입을 대상으로 하면 모든 하위 타입을 처리했는지 컴파일러가 검사하므로 default가 필요 없음
- Record 패턴(Java 21): Record를 구성 요소 단위로 분해하여 꺼냄
- 이름 없는 변수와 패턴(Java 22): 사용하지 않는 변수나 패턴 요소를 _로 표시함
```
if (obj instanceof String text && !text.isBlank()) {
    System.out.println(text.length());
}

String message = switch (result) {
    case Approved(String transactionId) -> "승인 완료: " + transactionId;
    case Declined(String reason) when reason.contains("한도") -> "한도 초과";
    case Declined(_) -> "결제 거절";
    case Pending() -> "결제 대기";
};
```
- Sealed 타입, Record, 패턴 매칭을 함께 사용하면 타입으로 경우를 나누고 컴파일러가 누락을 검사해주는 데이터 중심 프로그래밍(Data-Oriented Programming)이 가능함

<br>

### Sequenced Collections
- 순서가 있는 컬렉션의 공통 인터페이스로, Java 21에서 추가됨
  - SequencedCollection, SequencedSet, SequencedMap
- List, Deque, LinkedHashSet, LinkedHashMap 등에서 같은 메서드로 처음과 마지막 요소를 다룰 수 있음
  - getFirst(), getLast(), addFirst(), addLast(), removeFirst(), removeLast(), reversed()
  - 이전에는 list.get(list.size() - 1)처럼 컬렉션마다 다른 방식으로 접근해야 했음
```
List<Order> orders = repository.findRecentOrders();

Order latest = orders.getLast();
List<Order> newestFirst = orders.reversed();
```

<br>

### Stream Gatherers
- Stream의 중간 연산을 직접 정의할 수 있는 기능으로, Java 24에서 정식 기능이 됨
- 기본으로 제공되는 Gatherer
  - windowFixed(n): n개씩 묶음
  - windowSliding(n): 한 칸씩 이동하며 n개씩 묶음
  - fold, scan: 누적 계산
  - mapConcurrent: 가상 스레드로 동시에 변환하되 동시 실행 수를 제한함
```
List<List<Integer>> chunks = Stream.of(1, 2, 3, 4, 5)
        .gather(Gatherers.windowFixed(2))
        .toList(); // [[1, 2], [3, 4], [5]]

List<Price> prices = productIds.stream()
        .gather(Gatherers.mapConcurrent(10, priceClient::getPrice))
        .toList();
```

<br>

### Java 25에서 정식 기능이 된 문법
- Scoped Values(JEP 506)
  - 요청 단위의 정보(사용자, 추적 ID 등)를 메서드 매개변수로 넘기지 않고 하위 호출에 전달하는 불변 값
  - ThreadLocal과 달리 값을 바꿀 수 없고 범위가 끝나면 자동으로 사라지며, 가상 스레드를 많이 만들어도 부담이 적음
- Flexible Constructor Bodies(JEP 513)
  - 생성자에서 super()나 this()를 호출하기 전에 인수 검증 같은 코드를 작성할 수 있음
- Compact Source Files와 Instance Main Methods(JEP 512)
  - 클래스 선언 없이 void main()만으로 프로그램을 작성할 수 있어, 간단한 스크립트나 학습용 코드를 짧게 작성할 수 있음
- Module Import Declarations(JEP 511)
  - import module java.base;처럼 모듈이 공개하는 모든 패키지를 한 번에 가져옴
```
private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

void handle(Request request) {
    ScopedValue.where(REQUEST_ID, request.id())
            .run(() -> orderService.process(request));
}

void log(String message) {
    System.out.println("[" + REQUEST_ID.get() + "] " + message);
}
```
```
public class AdminUser extends User {

    public AdminUser(String name) {
        if (name.isBlank()) {
            throw new IllegalArgumentException("이름이 비어 있음");
        }
        super(name);
    }

}
```
- 여러 작업을 하나의 단위로 묶어 실행하고, 하나가 실패하면 나머지를 취소하는 Structured Concurrency는 Java 27에서도 아직 프리뷰 단계임. 가상 스레드와의 관계는 [가상 스레드와 동시성 제어](/java/virtual-thread.md) 참고

<br>

### 시작 시간과 메모리 개선
- Java 24부터 Project Leyden의 AOT(Ahead-of-Time) 캐시가 도입되어, 한 번 실행하며 기록한 클래스 로딩과 메서드 프로파일 정보를 다음 실행에 재사용함
  - Spring Boot처럼 시작할 때 많은 클래스를 불러오는 애플리케이션의 시작 시간을 줄여줌
- Java 25에서 정식 기능이 된 Compact Object Headers는 -XX:+UseCompactObjectHeaders 옵션으로 켜면 객체 헤더 크기를 줄여 메모리 사용량을 낮춰줌
- 시작 시간이 특히 중요하다면 GraalVM Native Image로 미리 컴파일하는 방법도 있지만, 빌드 시간이 길고 리플렉션 사용에 제약이 있음

<br>

#### 참고
- OpenJDK <JDK 25> - https://openjdk.org/projects/jdk/25/
- OpenJDK <JDK 27> - https://openjdk.org/projects/jdk/27/
- OpenJDK <JEP 441: Pattern Matching for switch> - https://openjdk.org/jeps/441
- OpenJDK <JEP 506: Scoped Values> - https://openjdk.org/jeps/506
- InfoQ <Java 25 Released> - https://www.infoq.com/news/2025/09/java25-released

#### 배워가는 것들
- Record, Sealed 타입, 패턴 매칭이 각각 따로 추가된 문법이 아니라, 함께 사용하여 경우를 타입으로 나누고 컴파일러가 누락을 검사하게 만드는 하나의 흐름이라는 것을 알게 되었다.
- Scoped Values가 ThreadLocal과 달리 불변이고 범위가 끝나면 사라진다는 점에서, 가상 스레드를 많이 만드는 환경에 더 잘 맞는다는 것을 이해할 수 있었다.
- 6개월마다 새 버전이 나오지만, 운영 환경에서는 LTS 버전과 정식 기능을 기준으로 삼아야 한다는 기준을 세울 수 있었다.
