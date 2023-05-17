# Hibernate

### Hibernate
* 가장 많이 사용하는 JPA 구현체.
* JDBC를 경유하여 DB와의 통신을 수행함.
* SQL 쿼리를 가지고 직접 통신을 수행하는 MyBatis 사용이 줄어들고, Hibernate와 같은 JPA 구현체 사용이 늘어나고 있음.

<br>

### EntityManager
* 엔티티(Entity)를 관리하는 역할을 수행하는 인터페이스.
* 엔티티의 생명주기와 트랜잭션을 관리함.
* Thread-Safe하지 않으므로, 동시성 이슈를 방지하기 위해 여러 쓰레드가 공유하지 않도록 해야 함.

<br>

### EntityManagerFactory
* EntityManager를 생성하는 주체.
* Thread-Safe하므로, 하나의 App에서 한 개만 생성하여 사용함.

<br>

### 트랜잭션(Transaction)
* 데이터베이스 상태 변화를 수행하기 위한 최소 단위를 의미함.
* Spring에서는 @Transactional 애노테이션을 이용히여 선언 가능함.

<br>

### JPQL(Java Persistence Query Language)
* 테이블이 아닌 Entity를 대상으로 한 쿼리.
* JPA 구현체(Hibernate)가 JPQL로 작성한 쿼리를 SQL로 변환해서 실행함.
* 아래와 같이 EntityManager의 createQuery 메서드 첫번째 변수에 쿼리를 넣고, 두번째 변수에 해당 타입을 넣어서 TypedQuery 형태로 반환하여 사용함.
```
TypedQuery<Post> query = entityManager.createQuery("SELECT p FROM Post AS p", Post.class); 
List<Post> posts = query.getResultList();
```

<br>

#### 참고
* 인프런 <스프링 Data JPA> - 백기선

#### 배워가는 것들
* JPA 구현체인 Hibernate에 대해 익힐 수 있었다.
* EntityManager, EntityManagerFactory에 대해 익힐 수 있었다.
* Spring Data JPA를 사용하게 된다면, 위의 것들을 있는그대로 직접 사용하는 일은 드물 것이다. 그렇지만, 어떠한 원리로 돌아가는지 잘 살펴볼 필요가 있다.
