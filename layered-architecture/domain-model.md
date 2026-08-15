# Domain Model

### 도메인 모델(Domain Model)
- 특정 비즈니스에 대한 모든 지식을 의미하는 도메인을 표현하는 개념적 모델
- 도메인 계층을 구현할 때 사용하는 객체 모델
- 도메인끼리 행위(behavior)를 명확하게 구분하여 각자 맡은 역할에 맞춰 올바르게 위임해야 함
- Spring Data JPA에서는 Repository를 이용하여 도메인 모델을 관리하고 있음

<br>

### Repository
- 사전적 의미로는 여러 데이터들을 보관하기 위한 저장소를 의미함
- JPA를 이용하여 데이터베이스 접근을 수행하기 위한 인터페이스

<br>

### VO(Value Object)
- 하나의 도메인 모델에 여러 값을 가지기 위한 목적으로 만들어진 객체
- 엔티티와 다르게 고유 식별자를 가지지 않기 때문에, 가지고 있는 여러 속성값에 따른 동등성 확인을 위해 사용함
- 같은 값들을 가진 VO의 논리적 동등성을 비교하기 위해 equals 메서드와 hashcode 메서드를 이용하여 비교할 수 있음
- equals 메서드와 hashcode 메서드를 오버라이딩하여 직접 구현해도 되지만, Lombok에 있는 @EqualsAndHashCode를 이용하여 간략하게 비교 가능
- 보통 불변값을 가지는 필드에 대해 @EqualsAndHashCode(of = "(불변값)")으로 선언하여 사용

```
public class Point {

    private Integer x;
    private Integer y;

    public Point(Integer x, Integer y) {
        this.x = x;
        this.y = y;
    }

    public void setX(Integer x) {
        return this.x = x;
    }

    public void setY(Integer y) {
        return this.y = y;
    }

    public Integer getX() {
        return this.x;
    }

    public Integer getY() {
        return this.y;
    }

    @Override
    public boolean equals(Object obj) {
        if ( obj instanceof Point ) {
            Point point = (Point) obj;
            return ( point.x.equals(this.x) && point.y.equals(this.y) );
        }

        return false;
    }

    @Override
    public int hashCode() {
        return Objects.hash(this.x, this.y);
    }
}

// Lombok 이용
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
public class Point {

    private Integer x;
    private Integer y;

}
```

<br>

### 유효성 검사 위치
- 입력값에 대한 유효성 검사는 그 성격에 따라 검사해야 할 위치가 달라짐
	- 컨트롤러단: 입력 형식 자체의 유효성(null, 길이, 패턴 등)을 검사함. DTO에 @Valid와 Bean Validation을 이용하여 "이 요청이 처리할 가치가 있는 형태인가"를 빠르게 걸러냄
	- 서비스단: 비즈니스 규칙(DB 조회가 필요한 검증, 상태 전이, 권한 등)을 검사함. "의미상 맞는가, 비즈니스적으로 허용되는가"를 판단함
	- DDD를 적용한다면 엔티티/VO의 생성자나 정적 팩토리 메서드에서 불변식(invariant)을 강제하는 것이 이상적임

- 타입 변환과 유효성 검사가 세트로 필요한 경우, 변환의 결과물이 되는 타입(VO 등) 자신에게 그 책임을 두는 것이 좋음
	- 변환과 검증은 그 타입의 불변식이므로, 응집도 관점에서 타입 자신이 책임지는 것이 맞음
	- 어디서 호출하든 동일한 규칙이 적용되어 중복을 방지할 수 있음
	- 서비스는 Money.of(rawAmount) 한 줄로 끝나며, 검증에 대한 지식을 서비스가 가질 필요가 없음
	- Spring 컨텍스트에 의존하지 않는 순수 자바 코드이므로 테스트가 용이함
```
public class Money {

    private final BigDecimal amount;

    private Money(BigDecimal amount) {
        this.amount = amount;
    }

    public static Money of(String raw) {
        if (raw == null || raw.isBlank()) {
            throw new IllegalArgumentException("금액은 필수입니다.");
        }

        BigDecimal parsed = new BigDecimal(raw); // 형변환

        if (parsed.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("금액은 음수일 수 없습니다."); // 검증
        }

        return new Money(parsed);
    }
}
```

- SRP(단일 책임 원칙) 관점에서도 변환+검증을 타입 자신에게 두는 것이 유리함. (SRP 원칙 자체에 대한 설명은 [SOLID - 단일 책임 원칙(SRP)](/hexagonal-architecture/solid.md#단일-책임-원칙srp-single-responsibility-principle) 참고.)
	- 변환과 검증은 별개의 책임이 아니라 "유효한 객체를 만든다"는 하나의 책임의 두 단계이므로, 이를 한 곳에 두는 것은 SRP 위반이 아님
	- Money가 변경되는 이유는 오직 "금액 규칙이 바뀔 때" 하나뿐이므로 SRP에 부합함
	- 반대로 이 로직을 서비스단에 두면, 서비스가 "주문 생성"과 "금액 형식 지식"을 동시에 짊어지게 되어 SRP를 위반함. 금액 규칙이 바뀌면 관련된 모든 서비스(OrderService, PaymentService, RefundService 등)를 전부 수정해야 함

- VO로 분리하기 어려운 현실적인 상황에서는 아래와 같은 절충안을 사용할 수 있음
	- 서비스 내 private 헬퍼 메서드로 분리: 검증 로직을 한 메서드에 모아두어 가독성과 관리가 용이해지고, 나중에 VO로 뺄 때 extract만 하면 됨. 다만 "서비스가 검증 규칙을 안다"는 책임 자체는 그대로 남으므로 SRP가 해결되는 것은 아님(헬퍼 분리는 가독성/응집도의 축, SRP는 클래스가 지는 책임 개수의 축으로 서로 다름)
	- 별도 Validator/Converter 클래스로 분리: VO까지는 부담스럽지만 검증 로직만이라도 별도 클래스가 책임지게 하여, 서비스는 본연의 책임에만 집중하고 SRP에 더 가까워짐

- 상황별 선택 기준
	- 여러 서비스에서 같은 검증 로직이 반복됨 → 별도 Validator/Converter 클래스 또는 VO로 분리
	- 한 서비스 안에서만 쓰이는 검증 → private 헬퍼 메서드로 충분
	- 설계 여유가 있고 도메인 개념이 명확함 → VO(Value Object)의 정적 팩토리 메서드에 일임

<br>

### AttributeConverter
- Value 타입의 프로퍼티를 DB 컬럼에 매핑할 때 사용하는 인터페이스
	- convertToDatabaseColumn(): Value 타입을 DB 컬럼 값으로 변환함
	- convertToEntityAttribute(): DB 컬럼 값을 Value로 변환함
	- 아래와 같이 AttributeConverter 인터페이스를 구현한 MoneyConverter 클래스를 통해 Money Value 타입을 매핑할 수 있음
		- 클래스의 사용을 위해 @Converter 애노테이션으로 선언하며, autoApply(기본값: false) 속성을 true로 지정함으로써 모든 Money 타입의 속성에 대해 MoneyConverter를 자동으로 적용할 수 있음
		- 만약 autoApply 속성을 따로 지정하지 않는다면, 아래와 같이 변환하려는 속성에 @Convert 애노테이션을 넣어서 적용해줘야 함
```
@Getter
@AllArgsConstructor
@EqualsAndHashCode
public class Money {

    private int value;

    public Money multiply(int multiplier) {
        return new Money(value * multiplier);
    }

    @Override
    public String toString() {
        return Integer.toString(value);
    }

}


public class MoneyConverter implements AttributeConverter<Money, Integer> {

    @Override
    public Integer convertToDatabaseColumn(Money money) {
        return money == null ? null : money.getValue();
    }

    @Override
    public Money convertToEntityAttribute(Integer value) {
        return value == null ? null : new Money(value);
    }
}


public class Order {
    
    @Convert(converter = MoneyConverter.class)
    @Column(name = "total_amounts")
    private Money totalAmounts;

}
```

<br>

#### 참고
- 인프런 <스프링 Data JPA> - 백기선
- <도메인 주도 개발 시작하기: DDD 핵심 개념 정리부터 구현까지> - 최범균

#### 배워가는 것들
- 도메인 모델에 대한 기초적인 개념을 익힐 수 있었다
- 도메인 모델을 관리하는 Repository와 동등성 확인을 위해 사용하는 객체인 VO에 대해 익힐 수 있었다
- 도메인 관련 학습은 상당히 많은 노력을 필요로 하는 것을 알게 되었다. 처음부터 모든걸 다 흡수하기는 쉽지 않을테니, 관련 개념에 대한 지속적인 반복과 실무 경험을 통해 하나하나 흡수해나간다는 느낌으로 학습해나가려 한다
