# Value Type

* Aggregate Mapping
  * 엔티티 필드 구성 시, 기본 자료형을 사용하는것보다는 의미가 담긴 VO(Value Object)를 사용하는 것이 좋음.
  * @Embeddable 애노테이션을 이용하여 VO를 구성하며, 구성한 VO를 엔티티의 필드로 넣을 때 @Embedded 애노테이션을 사용함.
  * @Enumerated(EnumType.STRING) 애노테이션을 이용하여 엔티티 필드를 열거형(Enum)으로 구성할 수 있음.
  * 예시
    * 이름 - 성(lastName), 이름(firstName) 구분.
    * 주소 - 도로명(street), 시(city), 도(state), 우편번호(zipCode) 구분.
```
@Embeddable
@EqualsAndHashCode
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class UserName {

    private String firstName;
    
    private String lastName;

}


@Embeddable
@EqualsAndHashCode
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Address { 

    private String street;

    private String city;

    private String state;

    private String zipCode;

}

@Entity 
@Getter 
@Setter 
@NoArgsConstructor 
@AllArgsConstructor
public class Account {

    @Id 
    @GeneratedValue 
    private Long id;

    @Column(nullable = false, unique = true) 
    private String accountId;

    private String password;

    @Embedded
    private UserName userName;

    @Embedded
    private Address address;

}
```

<br>

* JPA에서 자주 사용하는 애노테이션
  * @Entity
    * 도메인 클래스가 엔티티(Entity)라는 것을 명시적으로 알려줄 때 사용.
    * 일반적으로 엔티티 클래스와 매칭되는 테이블이 서로 같은 이름을 사용함.
    * 만약 매칭되는 테이블의 이름이 엔티티 이름과 서로 다른 경우, name 속성에 매칭되는 테이블 이름을 넣어서 지정할 수 있음.
  * @Table
    * 엔티티 클래스에 매칭되는 테이블의 이름을 명시할 때 사용. @Entity의 name이 기본값으로 지정됨.
    * 만약 @Entity와 @Table에서 name을 모두 지정한 경우 @Entity의 name은 JPQL에서 사용되며, @Table의 name은 DB에서 사용됨.
  * @Id
    * 엔티티에서 주 키(PK, Primary Key)를 매핑할 때 사용.
    * Primitive 타입과 그에 해당하는 Wrapper 타입 모두 사용 가능함(Date, BigDecimal, BigInteger도 사용 가능).
  * @GeneratedValue
    * 주 키(PK)의 생성방법을 매핑하는 애노테이션.
    * 생성 전략(strategy)과 생성기(generator)를 설정할 수 있음.
      * 기본 전략은 AUTO이며, 사용하는 DB에 따라 적절한 전략을 선택할 수 있음.
  * @Column
	  * @Entity 필드에 들어가는 기본 속성(생략 가능). 아래 속성들을 부여할 수 있음.
      * unique - 유일한 값
      * nullable - NULL 가능여부
      * length - 길이
      * columnDefinition - 컬럼값 정의
      * (...)
  * @Embeddable
    * Aggregate Mapping을 수행할 때, 수행 대상이 되는 클래스에 사용.
  * @Embedded
    * @Embeddable 애노테이션이 들어간 클래스의 인스턴스를 멤버로 선언할 때 사용.
  * @EmbeddedId
    * @Embeddable 애노테이션이 들어간 클래스의 인스턴스를 주 키로 매핑할 때 사용.
  * @AttributeOverride
    * 기본 자료형이나 Wrapper 클래스가 아닌 클래스에 대한 인스턴스를 멤버로 선언할 때 사용.
    * name 속성을 이용하여 해당 클래스의 멤버로 매핑할 수 있으며, column 속성을 이용하여 DB 컬럼과 매핑 가능함.

```
@Entity(name = "comments")
@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class Comment {

    @EmbeddedId
    private CommentId id;

    @AttributeOverride(name = "id", column = @Column(name = "postid"))
    private PostId postId;

    private String author;

    private String content;

    public Comment(PostId postId, String author, String content) {
        this.id = CommentId.generate();
        this.postId = postId;
        this.author = author;
        this.content = content;
    }

    public void update(String content) {
        this.content = content;
    }

}
```

<br>

* 참고
  * 인프런 <스프링 Data JPA> - 백기선
  * <자바 ORM 표준 JPA 프로그래밍> - 김영한
