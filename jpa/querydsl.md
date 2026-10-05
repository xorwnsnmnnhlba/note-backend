# QueryDSL

### Spring Data JPA로 조회를 작성하는 방식과 한계
- 파생 쿼리 메서드(Derived Query Method)
  - findByEmailAndArchivedAtIsNull처럼 메서드 이름을 분석하여 쿼리를 만들어줌
  - 조건이 한두 개일 때는 간결하지만, 조건이 늘어나면 메서드 이름이 지나치게 길어져 의도를 알아보기 어려움
  - 예: findBySuiteIdAndEnvironmentIdAndTriggerAndStatusAndIdLessThanOrderByIdDesc
- @Query로 JPQL 직접 작성
  - 쿼리를 문자열로 작성하므로, 엔티티의 필드 이름을 바꿔도 컴파일은 성공하고 실행 시점에서야 오류가 드러남
  - 조건이 선택적으로 붙고 빠지는 동적 쿼리를 문자열로 조립하기 번거로움
- 이런 한계를 보완하기 위해, 쿼리를 Java 코드로 작성할 수 있도록 해주는 라이브러리가 QueryDSL임

<br>

### QueryDSL
- 엔티티를 기반으로 생성한 Q-타입(Q-Type) 클래스를 이용하여 타입 안전(Type-Safe)한 쿼리를 작성할 수 있도록 해주는 라이브러리
  - 쿼리를 문자열이 아닌 Java 코드로 작성하므로, 필드 이름이나 타입이 틀리면 컴파일 오류가 발생함
  - IDE의 자동 완성을 사용할 수 있음
  - 조건을 객체로 다루므로, 동적 쿼리를 깔끔하게 조립할 수 있음
- Q-타입은 애노테이션 프로세서(APT, Annotation Processing Tool)가 컴파일 시점에 @Entity를 읽어 생성함
  - TestRun 엔티티라면 QTestRun 클래스가 생성됨
  - Gradle 기준으로 build/generated/sources/annotationProcessor 경로에 생성됨
- 원본 프로젝트(com.querydsl)는 5.1.0 이후 새 릴리스가 없어, 현재는 OpenFeign에서 유지보수 중인 포크(io.github.openfeign.querydsl)를 주로 사용함
  - Hibernate 7(Jakarta Persistence 3.2) 등 최신 버전 대응이 이루어지고 있음
  - Spring Boot가 의존성 버전을 관리해주지 않으므로, 버전을 직접 명시해야 함
```
build.gradle

dependencies {
    implementation 'io.github.openfeign.querydsl:querydsl-jpa:7.7'
    annotationProcessor 'io.github.openfeign.querydsl:querydsl-apt:7.7:jakarta'
    annotationProcessor 'jakarta.persistence:jakarta.persistence-api'
}
```

<br>

### JPAQueryFactory
- QueryDSL로 JPA 쿼리를 작성할 때 시작점이 되는 객체로써, EntityManager를 전달하여 생성함
- Spring이 주입해주는 EntityManager는 현재 트랜잭션의 실제 EntityManager로 위임해주는 Proxy이므로, JPAQueryFactory를 한 번 만들어두고 여러 Thread에서 함께 사용해도 됨
- SQL과 비슷한 순서로 메서드 체인을 작성함
  - selectFrom(), select().from(): 조회 대상
  - join(), fetchJoin(): 조인 및 연관 엔티티 함께 로딩
  - where(): 조건. 쉼표로 여러 조건을 전달하면 AND로 묶이며, null인 조건은 무시됨
  - orderBy(), groupBy(), offset(), limit()
  - fetch(): 목록 조회, fetchOne(): 단건 조회, fetchFirst(): 첫 건만 조회
```
QTestRun run = QTestRun.testRun;

List<TestRun> runs = queries.selectFrom(run)
        .join(run.suite).fetchJoin()
        .where(run.suite.id.eq(suiteId),
               run.status.eq(TestRun.Status.COMPLETED),
               run.id.lt(beforeRunId))
        .orderBy(run.id.desc())
        .limit(20)
        .fetch();
```
- fetchJoin()으로 연관 엔티티를 함께 로딩하면, 목록의 행마다 추가 쿼리가 발생하는 N+1 문제를 막을 수 있음
  - 관련 내용은 [Relationship Mapping](/jpa/relationship-mapping.md) 참고

<br>

### 동적 쿼리
- 검색 화면처럼 사용자가 입력한 조건만 쿼리에 포함해야 할 때는 BooleanBuilder로 조건을 조립함
- 조건을 만드는 부분을 별도의 객체로 분리하면, 조회 조건이 서비스 로직에 흩어지지 않음
```
public record RunFilter(Long suiteId, Long environmentId, TestRun.Status status) {

    Predicate toPredicate() {
        QTestRun run = QTestRun.testRun;
        BooleanBuilder conditions = new BooleanBuilder();
        if (suiteId != null) {
            conditions.and(run.suite.id.eq(suiteId));
        }
        if (environmentId != null) {
            conditions.and(run.environment.id.eq(environmentId));
        }
        if (status != null) {
            conditions.and(run.status.eq(status));
        }
        return conditions;
    }

}
```
- 자주 쓰이는 조건은 BooleanExpression을 반환하는 메서드로 만들어두고 재사용할 수 있음
```
private static BooleanExpression wholeSuite() {
    return run.testCaseId.isNull().and(run.rerunOfRunId.isNull());
}
```

<br>

### Spring Data JPA와 함께 사용하기
- Spring Data JPA는 Repository에 직접 구현한 메서드를 추가할 수 있도록 사용자 정의 조각(Custom Repository Fragment) 기능을 제공함
  1. 직접 구현할 메서드를 선언한 인터페이스를 만듦
  2. 인터페이스 이름 뒤에 Impl을 붙인 클래스에 구현함
  3. Repository가 JpaRepository와 함께 해당 인터페이스를 상속함
- 이렇게 하면 파생 쿼리 메서드와 QueryDSL 메서드를 하나의 Repository에서 함께 사용할 수 있음
```
public interface TestRunQueries {

    Page<TestRun> findNewestFirst(Predicate condition, Pageable pageable);

}


class TestRunQueriesImpl implements TestRunQueries {

    private final JPAQueryFactory queries;

    TestRunQueriesImpl(EntityManager entityManager) {
        this.queries = new JPAQueryFactory(entityManager);
    }

    ...

}


public interface TestRunRepository extends JpaRepository<TestRun, Long>, TestRunQueries {

    List<TestRun> findByStatus(TestRun.Status status);

}
```
- 구현 클래스에서 JPAQueryFactory를 직접 생성하면, 별도의 설정 Bean이 없으므로 @DataJpaTest 같은 슬라이스 테스트에서도 추가 설정 없이 동작함
- Spring Data가 제공하는 QuerydslPredicateExecutor를 상속하는 방법도 있으나, 조인 로딩이나 정렬, 건수 쿼리를 세밀하게 지정하기 어려움

<br>

### 페이지 조회
- 페이지 목록을 만들려면 목록 쿼리와 전체 건수 쿼리가 모두 필요함
- PageableExecutionUtils.getPage()에 건수 쿼리를 함수로 전달하면, 첫 페이지의 결과가 페이지 크기보다 작은 경우처럼 건수를 알 수 있는 상황에서는 건수 쿼리를 생략해줌
```
List<TestRun> content = queries.selectFrom(run)
        .where(condition)
        .orderBy(run.startedAt.desc(), run.id.desc())
        .offset(pageable.getOffset())
        .limit(pageable.getPageSize())
        .fetch();

return PageableExecutionUtils.getPage(content, pageable,
        () -> queries.select(run.count()).from(run).where(condition).fetchOne());
```

<br>

### 일괄 변경(Bulk Update)
- update(), delete()로 여러 행을 한 번에 변경할 수 있으며, execute()는 변경된 행 수를 반환함
- 엔티티를 조회하여 값을 바꾸는 방식(변경 감지)은 엔티티 전체를 다시 저장하므로, 여러 Thread가 같은 행을 바꾸면 한쪽의 변경을 덮어쓸 수 있음
  - 필요한 컬럼만 바꾸는 UPDATE를 사용하면 이러한 문제를 피할 수 있음
```
long updated = queries.update(run)
        .set(run.status, TestRun.Status.CANCELED)
        .set(run.finishedAt, now)
        .where(run.id.eq(runId), run.status.eq(TestRun.Status.RUNNING))
        .execute();
```
- 일괄 변경은 영속성 컨텍스트를 거치지 않고 DB에 바로 반영되므로, 이미 조회해둔 엔티티는 변경 전 값을 가지고 있음
  - 같은 트랜잭션에서 변경 후 값을 다시 사용해야 한다면 EntityManager.clear() 후 다시 조회해야 함

<br>

### 네이티브 쿼리가 필요한 경우
- PERCENTILE_CONT, LATERAL처럼 JPQL과 QueryDSL에 대응하는 문법이 없는 경우에는 @Query(nativeQuery = true)로 SQL을 직접 작성함
- 네이티브 쿼리에는 hibernate.default_schema 설정이 적용되지 않으므로, 테이블 앞에 {h-schema}를 붙이면 Hibernate가 기본 스키마로 치환해줌
```
@Query(nativeQuery = true, value = """
        SELECT ...
        FROM {h-schema}test_result r
        JOIN {h-schema}test_run t ON t.id = r.run_id
        """)
```
- 관련 SQL 문법은 [인덱스와 실행 계획](/database/index-and-query-plan.md) 참고

<br>

#### 참고
- OpenFeign QueryDSL - https://github.com/OpenFeign/querydsl
- Spring Data JPA Reference Documentation <Custom Repository Implementations> - https://docs.spring.io/spring-data/jpa/reference/repositories/custom-implementations.html

#### 배워가는 것들
- 문자열 쿼리는 필드 이름이 바뀌어도 컴파일이 된다는 점이 생각보다 큰 위험이라는 것을 알게 되었다. Q-타입을 쓰면 엔티티를 고치는 순간 깨지는 쿼리가 컴파일 오류로 드러난다.
- 원본 QueryDSL이 더 이상 릴리스되지 않고 포크가 그 역할을 이어받았다는 것을 알게 되었다. 라이브러리를 고를 때는 최근 릴리스와 유지보수 상태도 함께 확인해야겠다.
- 파생 쿼리 메서드는 조건이 적을 때만 쓰고, 조건이 많아지면 의도를 드러내는 이름의 메서드로 옮기는 기준이 유용했다. 이름이 쿼리 그 자체가 되면 읽는 사람이 무엇을 위한 조회인지 알 수 없다.
