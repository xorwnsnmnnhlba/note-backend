# Relationship Mapping

### N+1 problem
* JPA를 통한 데이터 조회 시, 조회한 데이터만큼의 SQL 쿼리가 추가적으로 발생하는 문제.
* 데이터 수만큼 쿼리를 추가로 발생시키기 때문에, DB와 구현한 모듈에 부하를 야기할 수 있음.
* Fetch Join, @EntityGraph, @NamedEntityGraph 애노테이션을 적절하게 활용하여 현상을 해소할 수 있음.

<br>

### 도메인 주도 설계(DDD, Domain Driven Design)에서의 Aggregate
* 단일 책임을 수행할 수 있는 엔티티의 집합 단위.
* 임의의 CRUD 작업이 이루어질 때 하나의 Aggregate에 속한 여러 엔티티들이 동시에 영향을 받게 됨.
* 설계 단계에서 여러 개의 엔티티를 가지고 Aggregate를 올바르게 잘 구성해야 하며, 이것이 바로 도메인 중심 설계의 핵심이라 볼 수 있음.
* Aggregate 단위로 관리를 수행하게 되면, 많은 엔티티를 가진 복잡한 도메인 이해에 대한 부담을 조금이나마 덜어낼 수 있음. 또한, 도메인 변경/확장에 대한 부담도 덜어낼 수 있음.

<br>

### Event Sourcing
* 데이터의 현재 상태만 저장하지 않고, 데이터에 수행된 모든 작업들을 하나하나 기록하는 것을 의미함.

<br>

### Relationship Mapping 시 사용하는 JPA 애노테이션
* @OneToMany
	* 엔티티에서 일대다(1:N) 관계를 표현할 때 사용하는 애노테이션.
	* 엔티티 클래스의 필드에 선언함. 해당 클래스가 One이며, 선언이 이루어진 필드에 대한 클래스가 Many이다.
	* CascadeType
		* @OneToMany에서 엔티티의 상태 변화를 전파시키는 옵션이며, enum 형태로 구성되어 있음. cascade 옵션 값으로 넣을 수 있음.
		* 기본값이 없기 때문에, 값을 넣어주지 않으면 전이시키지 않음.
		* 선택 가능한 옵션은 ALL, PERSIST, MERGE, REMOVE, REFRESH, DETACH.
		* 일반적으로 자식 엔티티로 모든 작업을 전파시키는 CascadeType.ALL 옵션을 가장 많이 사용함.
	* orphanRemoval
		* 자식 엔티티가 부모 엔티티와 연관관계가 끊어졌을 때, 이를 삭제하기 위한 옵션.
		* 기본값은 false이며, true로 구성할 경우 부모 엔티티가 삭제했을 때 엮여있는 자식 엔티티까지 삭제가 이루어짐.
	* @OneToMany 옵션에 cascade=CascadeType.ALL, orphanRemoval=true을 넣게 되면, 부모 엔티티가 자식 엔티티의 생명주기까지 관리하게 됨.
		* 위의 옵션을 적용해서 Aggregate로 구성된 엔티티들을 올바르게 관리하는 것을 권장함.
* @ManyToOne
	* 엔티티에서 다대일(N:1) 관계를 표현할 때 사용하는 애노테이션.
	* @OneToMany와 마찬가지로 엔티티 클래스의 필드에 선언하며, @OneToMany와 정반대 형태로 구성됨.
	* 양방향 관계 구성을 수행하는 경우, @OneToMany 쪽에서 mappedBy를 사용하여 관계를 맺고 있는 필드(@ManyToOne 선언 필드)를 명확하게 설정해야 함.
	* @JoinColumn
		* @ManyToOne에서 외래 키(FK)를 매핑할 때 사용하며, 생략 가능함.
		* 생략이 이루어진 경우, 외래 키가 (선언이 이루어진 필드에 대한 클래스명)_(@Id 애노테이션에 해당하는 필드명)으로 구성됨.

```

@Entity 
@Getter 
@Setter 
@ToString 
@NoArgsConstructor 
@AllArgsConstructor 
public class Study {

    @Id 
    @GeneratedValue 
    private Long id;

    private String name;

    @ManyToOne 
    private Account owner;

}


@Entity 
@Getter 
@Setter 
@ToString(exclude = "studies") 
@NoArgsConstructor 
@AllArgsConstructor 
public class Account {

    @Id 
    @GeneratedValue 
    private Long id;

    @Column(nullable = false, unique = true) 
    private String username;

    private String password;

    @OneToMany(mappedBy = "owner")
    private Set<Study> studies = new HashSet<>();
	
		(하략)
	
}
```

<br>

#### 참고
* 인프런 <스프링 Data JPA> - 백기선
* https://jaegukim.github.io/posts/joincolumn-vs-mappedby-orm%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%A0%EB%95%8C-%EC%A3%BC%EC%9D%98%ED%95%A0%EC%A0%90/
* https://learn.microsoft.com/ko-kr/azure/architecture/patterns/event-sourcing
