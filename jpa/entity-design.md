# 엔티티 설계

### 공통 속성을 상위 클래스로 묶기
- 모든 엔티티가 가지는 ID, 생성 시각, 수정 시각 같은 속성은 @MappedSuperclass로 선언한 상위 클래스에 모아둘 수 있음
  - @MappedSuperclass는 엔티티가 아니므로 테이블로 만들어지지 않고, 상속받은 엔티티의 테이블에 컬럼으로 포함됨
- Hibernate의 @CreationTimestamp, @UpdateTimestamp를 사용하면 저장, 수정 시각을 자동으로 채워줌
```
@MappedSuperclass
@Getter
public abstract class BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreationTimestamp
    @Column(nullable = false, updatable = false)
    private Instant createdAt;

    @UpdateTimestamp
    @Column(nullable = false)
    private Instant updatedAt;

}
```
- 시각은 시간대 정보가 없는 LocalDateTime보다 Instant(UTC 기준 시점)로 저장하고, 화면에 보여줄 때 시간대를 적용하는 것이 안전함
  - 관련 내용은 [로그인 & 로그아웃, 회원가입](/spring-security/login-logout-signup.md)의 날짜·시간 API 참고

<br>

### 엔티티의 동일성 비교
- 엔티티에 equals()와 hashCode()를 모든 필드 기준으로 재정의하면 문제가 생길 수 있음
  - 지연 로딩 Proxy는 실제 엔티티와 클래스가 다르고 필드가 비어 있으므로, 필드 비교 결과가 달라짐
  - 연관 엔티티까지 비교하거나 toString()으로 출력하다가, 지연 로딩이 연쇄적으로 발생하거나 양방향 연관관계에서 무한 순환이 일어날 수 있음
- 저장된 엔티티끼리는 ID로 비교하는 것이 안전함
  - 저장 전에는 ID가 없으므로 인스턴스 자체(==)로 비교해야 함
  - Proxy의 getId()는 지연 로딩을 일으키지 않음
```
public static boolean same(BaseEntity a, BaseEntity b) {
    if (a == b) {
        return true;
    }
    return a != null && b != null && a.getId() != null && a.getId().equals(b.getId());
}
```

<br>

### 엔티티를 안전하게 만드는 방법
- 기본 생성자는 protected로 선언함
  - JPA는 리플렉션으로 엔티티를 만들기 위해 기본 생성자가 필요함
  - public으로 열어두면 검증을 거치지 않은 빈 엔티티를 누구나 만들 수 있음
- 값을 받는 생성자에서 필수 값과 규칙을 검증함
- Setter 대신 의도를 드러내는 도메인 메서드로만 상태를 바꿈
  - 예: setName() 대신 rename(), setArchivedAt() 대신 archive()
  - 메서드 안에서 규칙을 검증하므로, 엔티티가 잘못된 상태가 되는 것을 막을 수 있음
- 컬렉션 필드는 수정할 수 없는 복사본으로 반환함(방어적 복사)
  - 내부 리스트를 그대로 반환하면 외부에서 add(), remove()로 도메인 메서드의 검증을 우회할 수 있고, 변경 감지에 의해 그대로 DB에 반영됨
```
public List<Assertion> getAssertions() {
    return assertions == null ? List.of() : List.copyOf(assertions);
}
```

<br>

### Lombok 사용 범위 제한하기
- Lombok은 편리하지만, 일부 기능은 위에서 정리한 엔티티 설계 원칙을 쉽게 깨뜨림
  - @Setter, @Data: 검증 없이 상태를 바꾸는 메서드가 생김
  - @Builder, @AllArgsConstructor: 생성자의 검증을 우회하는 경로가 생김
  - @EqualsAndHashCode, @ToString: 지연 로딩 Proxy와 충돌함
  - @Getter: 컬렉션 필드에 사용하면 내부 컬렉션을 그대로 노출함
- 프로젝트 루트(또는 모듈)에 lombok.config 파일을 두면, 특정 기능을 사용했을 때 컴파일 오류가 나도록 막을 수 있음
  - config.stopBubbling = true: 상위 디렉터리의 lombok.config를 더 이상 찾지 않음
  - lombok.{기능}.flagUsage = error: 해당 기능 사용 시 컴파일 오류
```
lombok.config

config.stopBubbling = true

lombok.setter.flagUsage = error
lombok.data.flagUsage = error
lombok.builder.flagUsage = error
lombok.allArgsConstructor.flagUsage = error
lombok.toString.flagUsage = error
lombok.equalsAndHashCode.flagUsage = error
```
- 같은 이름의 메서드를 직접 작성해두면 Lombok은 해당 메서드를 생성하지 않으므로, 방어적 getter는 직접 작성하면 됨
- 노출할 필요가 없는 필드에는 @Getter(AccessLevel.NONE)을 지정하여 getter 생성을 막을 수 있음
- lombok.config로 강제할 수 없는 규칙(기본 생성자의 접근 수준 등)은 리플렉션으로 모든 엔티티를 검사하는 테스트로 고정할 수 있음
- DTO와 값 객체는 Lombok의 @Value 대신 Java record를 사용함. 관련 내용은 [DTO](/dto-json-cors/dto.md) 참고

<br>

### 소프트 삭제(Soft Delete)
- 행을 실제로 지우지 않고, 삭제 여부나 삭제 시각을 컬럼에 기록하는 방식
  - 실행 이력처럼 삭제된 대상을 계속 참조하는 데이터가 있을 때, 이력이 깨지지 않도록 하기 위해 사용함
  - 실수로 삭제한 데이터를 되살릴 수 있음
- 반대로 실제로 행을 지우는 방식을 하드 삭제(Hard Delete)라고 함
```
private Instant archivedAt;

public void archive(Instant archivedAt) {
    if (isArchived()) {
        throw new IllegalStateException("already archived");
    }
    this.archivedAt = archivedAt;
}
```
- 조회 시 삭제된 행을 제외하는 방법
  - 조회 조건에 archived_at IS NULL을 직접 명시함(파생 쿼리 메서드 이름에 ...ArchivedAtIsNull)
  - Hibernate의 @SQLRestriction("archived_at IS NULL")을 엔티티에 선언하면 모든 조회에 조건이 자동으로 붙음
  - Hibernate 6.4부터는 @SoftDelete 애노테이션으로 삭제 컬럼 관리까지 맡길 수 있음
- 조건을 자동으로 붙이는 방식은 편리하지만, 이력 화면처럼 삭제된 대상도 보여줘야 하는 조회까지 걸러버리므로 요구사항에 맞는지 확인해야 함
- 삭제할 때 하위 데이터를 어떻게 처리할지(연쇄 보관)는 애플리케이션에서 직접 구현해야 함

<br>

### 소프트 삭제와 유니크 제약조건
- 이름에 유니크 제약조건이 있으면, 삭제(보관)된 행의 이름을 다시 사용할 수 없는 문제가 생김
- 해결 방법
  - 부분 인덱스(Partial Index): CREATE UNIQUE INDEX ... WHERE archived_at IS NULL. 간단하지만 PostgreSQL 등 일부 DBMS만 지원함
  - 보관 키 컬럼: 사용 중이면 0, 보관되면 자기 ID를 넣는 컬럼(archive_key)을 유니크 제약조건에 포함함
- 보관 키 방식은 표준 SQL만으로 동작함
  - 사용 중인 행끼리는 archive_key가 모두 0이므로 이름 중복을 DB가 막아줌
  - 보관된 행은 archive_key가 서로 다르므로 같은 이름이 여러 개 있어도 됨
```
@Table(uniqueConstraints = @UniqueConstraint(columnNames = { "suite_id", "name", "archive_key" }))
```

<br>

### 외래키(Foreign Key) 제약조건을 두지 않는 경우
- 외래키 제약조건은 참조 무결성을 DB가 보장해주지만, 아래와 같은 이유로 일부러 사용하지 않기도 함
  - 소프트 삭제를 사용하면 실제로 행이 지워지지 않으므로, 외래키가 막아주는 상황이 거의 없음
  - 대용량 테이블에서는 INSERT, DELETE 시 참조 확인 비용과 잠금이 부담됨
  - 데이터 이관이나 테이블 분리 시 제약조건이 걸림돌이 됨
- 대신 참조 무결성을 애플리케이션에서 보장해야 하며, 조회 성능을 위한 인덱스는 따로 만들어야 함
- JPA에서는 @ForeignKey(ConstraintMode.NO_CONSTRAINT)를 지정하면 DDL을 생성할 때도 외래키를 만들지 않음
```
@ManyToOne(fetch = FetchType.LAZY, optional = false)
@JoinColumn(name = "suite_id", nullable = false, foreignKey = @ForeignKey(ConstraintMode.NO_CONSTRAINT))
private TestSuite suite;
```

<br>

### JSON 컬럼 매핑
- 구조가 자주 바뀌거나 종류가 다양한 값(판정 규칙 목록, 요청 헤더 등)은 별도 테이블 대신 JSON 컬럼 하나에 저장할 수 있음
  - 규칙 종류가 추가되어도 스키마를 바꿀 필요가 없고, 엔티티를 조회할 때 함께 로딩됨
  - 대신 JSON 안의 값을 조건으로 SQL 조회하기는 불편해짐
- Hibernate 6부터는 @JdbcTypeCode(SqlTypes.JSON)만으로 객체나 컬렉션을 JSON 컬럼에 매핑할 수 있으며, 변환에는 Jackson이 사용됨
```
@JdbcTypeCode(SqlTypes.JSON)
@Column(columnDefinition = "json")
private List<Assertion> assertions = new ArrayList<>();
```
- 하나의 컬럼에 여러 종류의 객체를 담는 다형 JSON은 [직렬화, Marshalling, JSON](/dto-json-cors/serialization-marshalling-JSON.md) 참고

<br>

#### 참고
- Hibernate ORM User Guide <Soft Delete> - https://docs.jboss.org/hibernate/orm/7.0/userguide/html_single/Hibernate_User_Guide.html#soft-delete
- Project Lombok <Configuration System> - https://projectlombok.org/features/configuration
- Vlad Mihalcea <How to implement equals and hashCode using the JPA entity identifier> - https://vladmihalcea.com/how-to-implement-equals-and-hashcode-using-the-jpa-entity-identifier/

#### 배워가는 것들
- Lombok의 편리한 애노테이션들이 엔티티의 검증을 우회하는 통로가 될 수 있다는 것을 알게 되었다. lombok.config로 허용 범위를 정해두면 규칙을 사람의 기억이 아니라 컴파일러가 지켜준다.
- 소프트 삭제가 단순히 컬럼 하나를 추가하는 일이 아니라, 유니크 제약조건과 조회 조건, 하위 데이터 처리까지 함께 설계해야 하는 일이라는 것을 배웠다.
- 외래키를 쓰지 않는 것도 하나의 설계 선택이라는 점이 인상적이었다. 대신 그만큼 애플리케이션이 무결성을 책임져야 하므로, 선택의 이유를 문서로 남겨야겠다.
