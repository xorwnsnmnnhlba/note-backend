# Relationship Mapping

* N+1 problem
	* JPA를 통한 데이터 조회 시, 조회한 데이터만큼의 SQL 쿼리가 추가적으로 발생하는 문제.
	* 데이터 수만큼 쿼리를 추가로 발생시키기 때문에, DB와 구현한 모듈에 부하를 야기할 수 있음.
	* Fetch Join, @EntityGraph, @NamedEntityGraph 애노테이션을 적절하게 활용하여 현상을 해소할 수 있음.

<br>

* 도메인 중심 설계(DDD, Domain Driven Design)에서의 Aggregate
	* 단일 책임을 수행할 수 있는 엔티티의 집합 단위.
	* 임의의 CRUD 작업이 이루어질 때 하나의 Aggregate에 속한 여러 엔티티들이 동시에 영향을 받게 됨.
	* 설계 단계에서 여러 개의 엔티티를 가지고 Aggregate를 올바르게 잘 구성해야 하며, 이것이 바로 도메인 중심 설계의 핵심이라 볼 수 있음.
	* Aggregate 단위로 관리를 수행하게 되면, 많은 엔티티를 가진 복잡한 도메인 이해에 대한 부담을 조금이나마 덜어낼 수 있음. 또한, 도메인 변경/확장에 대한 부담도 덜어낼 수 있음.

<br>

* Event Sourcing

<br>

* Relationship Mapping 시 사용하는 JPA 애노테이션
	* @OneToMany
		* 
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
  * @JoinColumn

<br>

* 참고
  * 인프런 <스프링 Data JPA> - 백기선
