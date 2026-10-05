# 캐싱

### 캐시(Cache)
- 한 번 계산하거나 조회한 결과를 가까운 저장소에 보관해두고, 같은 요청이 오면 다시 계산하지 않고 재사용하는 것
  - DB 조회, 외부 API 호출, 무거운 계산처럼 비용이 큰 작업의 응답 시간과 부하를 줄여줌
- 캐시가 효과적인 데이터
  - 자주 읽고 드물게 바뀌는 데이터(상품 카테고리, 공통 코드, 설정 값)
  - 약간 오래된 값을 보여줘도 문제가 없는 데이터(조회수, 인기 순위)
- 캐시에 맞지 않는 데이터
  - 항상 최신이어야 하는 데이터(재고 수량, 잔액)
  - 사용자마다 다르고 재사용될 일이 적은 데이터
- 브라우저와 CDN이 응답을 재사용하는 HTTP 캐시는 [HTTP 캐시와 조건부 요청](/http/http-cache.md) 참고

<br>

### 로컬 캐시와 분산 캐시
- 로컬 캐시(Local Cache)
  - 애플리케이션 메모리에 저장함. 네트워크를 거치지 않아 가장 빠름
  - 서버가 여러 대이면 서버마다 캐시 내용이 달라질 수 있고, 서버를 재시작하면 사라짐
  - Caffeine이 대표적인 Java 로컬 캐시 라이브러리임
- 분산 캐시(Distributed Cache)
  - Redis 같은 별도의 저장소에 저장하여 모든 서버가 같은 캐시를 공유함
  - 네트워크 왕복이 필요하고, 객체를 직렬화해서 저장해야 함
  - Redis에 대한 내용은 [Redis](/cqrs/redis.md) 참고
- 자주 쓰이고 바뀔 일이 거의 없는 작은 데이터는 로컬 캐시에, 여러 서버가 일관되게 공유해야 하는 데이터는 분산 캐시에 둠

<br>

### Spring Cache Abstraction
- Spring은 캐시 저장소와 무관하게 애노테이션으로 캐시를 적용하는 추상화를 제공함
  - 저장소는 의존성과 설정으로 바꾸며, 코드는 그대로 유지됨
- @EnableCaching을 선언하여 활성화하며, spring-boot-starter-cache를 추가하면 CacheManager가 자동 설정됨
- 주요 애노테이션
  - @Cacheable: 캐시에 값이 있으면 메서드를 실행하지 않고 캐시의 값을 반환하며, 없으면 실행한 결과를 저장함
  - @CachePut: 항상 메서드를 실행하고 결과로 캐시를 갱신함
  - @CacheEvict: 캐시에서 값을 삭제함. allEntries = true로 캐시 전체를 비울 수 있음
  - @Caching: 여러 캐시 애노테이션을 함께 적용함
- 주요 속성
  - cacheNames: 캐시 이름. 이름별로 만료 시간 등을 다르게 설정할 수 있음
  - key: SpEL로 캐시 키를 지정함. 생략하면 메서드 인수로 키를 만듦
  - condition, unless: 캐시할 조건과 결과에 따라 캐시하지 않을 조건
  - sync: 같은 키에 대한 동시 요청 중 하나만 메서드를 실행하게 함
```
@Configuration
@EnableCaching
public class CacheConfig {
}


@Service
public class ProductService {

    @Cacheable(cacheNames = "products", key = "#id", unless = "#result == null")
    public ProductResponse getProduct(Long id) {
        return ProductResponse.from(productRepository.findById(id).orElseThrow());
    }

    @CacheEvict(cacheNames = "products", key = "#id")
    @Transactional
    public void updateProduct(Long id, ProductUpdateRequest request) {
        ...
    }

}
```
- 캐시 애노테이션은 @Transactional과 마찬가지로 Proxy 기반이므로, 같은 클래스 안에서 호출하면 적용되지 않음

<br>

### Caffeine으로 로컬 캐시 사용하기
- caffeine 의존성을 추가하면 CaffeineCacheManager가 자동 설정됨
- spring.cache.caffeine.spec으로 최대 개수와 만료 시간을 지정함
  - maximumSize: 최대 항목 수. 넘으면 덜 사용된 항목부터 제거함
  - expireAfterWrite: 저장한 뒤 지정한 시간이 지나면 만료됨
```
build.gradle

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-cache'
    implementation 'com.github.ben-manes.caffeine:caffeine'
}
```
```
application.yml

spring:
  cache:
    cache-names: products, categories
    caffeine:
      spec: maximumSize=1000,expireAfterWrite=10m
```

<br>

### Redis로 분산 캐시 사용하기
- spring-boot-starter-data-redis를 추가하면 RedisCacheManager가 자동 설정됨
- spring.cache.redis.time-to-live로 만료 시간을 지정함
- 저장하는 값은 직렬화되므로, JPA Entity 대신 필요한 필드만 담은 DTO(Record)를 캐시해야 함
  - Entity를 캐시하면 지연 로딩 Proxy가 직렬화되지 않거나, 영속성 컨텍스트 밖에서 사용되는 문제가 생김
- 클래스 구조를 바꾸면 이미 저장된 값을 읽지 못할 수 있으므로, 배포할 때 캐시 키에 버전을 붙이거나 캐시를 비우는 방법을 마련해둠
```
application.yml

spring:
  cache:
    type: redis
    redis:
      time-to-live: 10m
      key-prefix: "app:v1:"
```

<br>

### 캐시 무효화
- 원본 데이터가 바뀌었는데 캐시에 이전 값이 남아 있으면 사용자에게 오래된 데이터가 보임
- 무효화 전략
  - 만료 시간(TTL): 일정 시간이 지나면 자동으로 사라지게 함. 가장 단순하며, 모든 캐시에 기본으로 지정하는 것이 좋음
  - 변경 시 삭제(Evict): 데이터를 변경하는 메서드에서 관련 캐시를 삭제함
  - 변경 시 갱신(Put): 데이터를 변경하면서 캐시도 새 값으로 바꿈
- 트랜잭션 안에서 캐시를 삭제하면, 커밋 전에 다른 요청이 이전 값을 다시 캐시에 넣을 수 있음
  - 트랜잭션이 커밋된 뒤에 삭제하거나, 짧은 TTL로 피해를 줄임
- 캐시를 지우는 곳을 빠뜨리기 쉬우므로, 데이터를 변경하는 경로를 한곳으로 모아두는 것이 좋음

<br>

### 캐시 사용 시 주의할 점
- 캐시 쇄도(Cache Stampede)
  - 인기 있는 키가 만료되는 순간 많은 요청이 동시에 DB로 몰리는 현상
  - sync = true로 같은 서버 안의 동시 계산을 하나로 줄이거나, 만료 시간에 무작위 값을 더해 동시에 만료되지 않게 함
- 캐시 관통(Cache Penetration)
  - 존재하지 않는 데이터를 계속 요청하면 캐시에 남지 않아 매번 DB를 조회하게 됨
  - 없다는 결과도 짧은 시간 캐시하는 방법이 있음
- 캐시 키에 사용자별 정보가 빠지면 다른 사용자의 데이터가 보이는 사고로 이어지므로, 사용자마다 다른 데이터는 키에 사용자 정보를 포함하거나 캐시하지 않음
- 캐시 적중률(Hit Ratio)을 지표로 확인하여, 효과가 없는 캐시는 제거함. Spring Boot는 캐시 지표를 Micrometer로 수집함

<br>

#### 참고
- Spring Framework Reference Documentation <Cache Abstraction> - https://docs.spring.io/spring-framework/reference/integration/cache.html
- Spring Boot Reference Documentation <Caching> - https://docs.spring.io/spring-boot/reference/io/caching.html
- GitHub <ben-manes/caffeine> - https://github.com/ben-manes/caffeine

#### 배워가는 것들
- 캐시는 빠르게 만드는 도구이면서 동시에 오래된 데이터를 보여줄 수 있는 위험이라는 점에서, 무엇을 캐시할지보다 언제 지울지를 먼저 정해야 한다는 것을 알게 되었다.
- 로컬 캐시와 분산 캐시의 차이가 결국 서버가 여러 대일 때 일관성을 지킬 수 있느냐의 차이라는 것을 정리할 수 있었다.
- Entity를 그대로 캐시하면 지연 로딩 Proxy 때문에 문제가 생긴다는 점이 인상 깊었다. 캐시에는 필요한 필드만 담은 DTO를 넣어야 한다.
