# Domain Model

### 도메인 모델(Domain Model)
* 특정 비즈니스에 대한 모든 지식을 의미하는 도메인을 표현하는 개념적 모델.
* 도메인끼리 행위(behavior)를 명확하게 구분하여 각자 맡은 역할에 맞춰 올바르게 위임해야 함.
* Spring Data JPA에서는 Repository를 이용하여 도메인 모델을 관리하고 있음.

<br>

### Repository
* 사전적 의미로는 여러 데이터들을 보관하기 위한 저장소를 의미함.
* JPA를 이용하여 데이터베이스 접근을 수행하기 위한 인터페이스.

<br>

### VO(Value Object)
* 하나의 도메인 모델에 여러 값을 가지기 위한 목적으로 만들어진 객체.
* 엔티티와 다르게 고유 식별자를 가지지 않기 때문에, 가지고 있는 여러 속성값에 따른 동등성 확인을 위해 사용함.
* 같은 값들을 가진 VO의 논리적 동등성을 비교하기 위해 equals 메서드와 hashcode 메서드를 이용하여 비교할 수 있음.
* equals 메서드와 hashcode 메서드를 오버라이딩하여 직접 구현해도 되지만, Lombok에 있는 @EqualsAndHashCode를 이용하여 간략하게 비교 가능.
* 보통 불변값을 가지는 필드에 대해 @EqualsAndHashCode(of = "(불변값)")으로 선언하여 사용.

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

#### 참고
* 인프런 <스프링 Data JPA> - 백기선

#### 배워가는 것들
* 도메인 모델에 대한 기초적인 개념을 익힐 수 있었다.
* 도메인 모델을 관리하는 Repository와 동등성 확인을 위해 사용하는 객체인 VO에 대해 익힐 수 있었다.
* 도메인 관련 학습은 상당히 많은 노력을 필요로 하는 것을 알게 되었다. 처음부터 모든걸 다 흡수하기는 쉽지 않을테니, 관련 개념에 대한 지속적인 반복과 실무 경험을 통해 하나하나 흡수해나간다는 느낌으로 학습해나가려 한다.
