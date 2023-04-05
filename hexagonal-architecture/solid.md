# SOLID

* SOLID 원칙
  * 객체지향 프로그래밍 및 설계에 필요한 기본 원칙. 아래 다섯가지로 구분할 수 있음.

<br>

* 단일 책임 원칙(SRP, Single Responsibility Principle)
  * 클래스가 제공하는 서비스는 오직 하나의 책임을 수행하는데 집중되어야 함.
    * 모종의 이유로 하나의 클래스를 변경해야할 때, 그 이유는 오직 하나뿐이어야 함.
  * 책임의 분배를 통한 클래스별 역할구분을 확실하게 함으로써, 코드의 가독성을 올리고 유지보수성을 향상시킬 수 있음.
  * Spring Data JPA를 사용하는 경우, Repository를 따로 놓음으로써 DBMS 변경에 따른 유연한 대처가 가능해짐.

<br>

* 개방-폐쇄 원칙(OCP, Open-Closed Principle)
  * 소프트웨어를 구성하는 요소(모듈, 클래스, 메서드 등등)들은 확장에는 열려있고, 변경에는 닫혀있어야 한다는 원리.
  * Java에서는 인터페이스(Interface)를 이용하여 OCP에 따른 구현을 가능하게 함.
  * 추상화와 다형성을 추구하여 객체지향 프로그래밍이 주는 장점을 누릴 수 있음.
  * 변경될 것과 변경되지 않아야할 것에 대한 구분을 명확하게 하도록 함.

```
@Entity
@Getter
@ToString
@NoArgsConstructor
public class Item {

    @EmbeddedId
    private ItemId id;

    @ManyToOne
    private Product product;

    private String color;

    private String size;

    private Integer stock;

    private Integer originalPrice;

    private Integer onSalePrice;

    private Integer shippingDays;

    public Item(Product product, String color, String size,
                Integer stock, Integer originalPrice, Integer onSalePrice, Integer shippingDays) {
        this.id = ItemId.generate();
        this.product = product;
        this.color = color;
        this.size = size;
        this.stock = stock;
        this.originalPrice = originalPrice;
        this.onSalePrice = onSalePrice;
        this.shippingDays = shippingDays;
    }

    public void update(Integer stock, Integer originalPrice, Integer onSalePrice, Integer shippingDays) {
        this.stock = stock;
        this.originalPrice = originalPrice;
        this.onSalePrice = onSalePrice;
        this.shippingDays = shippingDays;
    }
}


@Embeddable
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
public class ItemId implements Serializable {

    private String id;

    public static ItemId of(String id) {
        return new ItemId(id);
    }

    public static ItemId generate() {
        return new ItemId(TsidCreator.getTsid().toString());
    }

    @Override
    public String toString() {
        return id;
    }

}



```


<br>

* 리스코프 치환 원칙(LSP, Liskov Substitution Principle)


<br>

* 인터페이스 분리 원칙(ISP, Interface Segregation Principle)


<br>

* 의존관계 역전 원칙(DIP, Dependency Inversion Principle)


<br>
