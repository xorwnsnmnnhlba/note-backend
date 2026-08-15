# Actuator

### Spring Boot Actuator
- 운영 중인 애플리케이션의 상태를 확인하고 관리할 수 있는 기능을 제공해주는 모듈
  - 애플리케이션이 정상적으로 동작하고 있는지, 어떤 Bean이 등록되어 있는지, 어떤 설정값으로 구동되었는지 등을 HTTP 요청을 통해 확인 가능함
  - 별도의 코드 구현 없이 의존성 추가만으로 사용할 수 있으며, 자동 설정(Auto-configuration)을 통해 관련 Bean이 등록됨
- 개발 단계보다는 운영 단계에서 필요한 기능들을 제공하므로, Production-ready Feature라 부름
  - 모니터링 도구(Prometheus, Grafana 등)나 컨테이너 오케스트레이션 도구(Kubernetes)와 연동하여 사용하는 경우가 많음

<br>

### 의존성 추가
```
build.gradle

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```

<br>

### Endpoint
- Actuator가 제공하는 각각의 기능을 Endpoint라 하며, 기본 경로(Base Path)는 `/actuator`임
  - 예를 들어 health Endpoint는 `/actuator/health` 경로로 매핑됨
  - `management.endpoints.web.base-path` 속성을 통해 기본 경로를 변경할 수 있음
- 주요 Endpoint
  - health: 애플리케이션의 상태 정보를 제공함
  - info: 애플리케이션에 대한 임의의 정보를 제공함
  - metrics: 애플리케이션의 각종 메트릭 정보를 제공함
  - beans: 등록된 모든 Spring Bean의 목록을 제공함
  - env: ConfigurableEnvironment에 등록된 속성들을 제공함
  - configprops: @ConfigurationProperties가 선언된 설정 목록을 제공함
  - conditions: 자동 설정의 조건 평가 결과를 제공함. 어떤 자동 설정이 적용되었고 어떤 것이 적용되지 않았는지 그 이유와 함께 확인 가능함
  - mappings: @RequestMapping으로 매핑된 모든 경로의 목록을 제공함
  - loggers: Logger 설정을 조회하고 변경할 수 있음. 애플리케이션 재시작 없이 로그 레벨 변경이 가능함
  - httpexchanges: 최근에 주고받은 HTTP 요청/응답 정보를 제공함(기본 100건)
  - threaddump: Thread Dump를 수행함
  - heapdump: Heap Dump 파일을 반환함
  - scheduledtasks: 등록된 스케줄링 작업의 목록을 제공함
  - caches: 사용 가능한 캐시 정보를 제공함
  - shutdown: 애플리케이션을 정상적으로 종료시킴
  - flyway, liquibase: 적용된 DB Migration 이력을 제공함

<br>

### 노출(Exposure)과 접근(Access)
- Actuator는 노출과 접근이라는 서로 다른 두 개념으로 Endpoint를 제어하므로, 구분하여 이해해야 함
  - 노출(Exposure): 해당 Endpoint를 HTTP 또는 JMX를 통해 외부에서 호출할 수 있도록 열어줄 것인지에 대한 설정
  - 접근(Access): 노출된 Endpoint에 대해 어느 수준까지 허용할 것인지에 대한 설정
- 노출 설정
  - 기본적으로 HTTP와 JMX 모두 health Endpoint만 노출됨
  - `management.endpoints.web.exposure.include` / `exclude` 속성으로 조정 가능하며, `*`를 사용하여 전체 노출도 가능함(YAML에서는 따옴표로 감싸줘야 함)
```
application.yml

management:
  endpoints:
    web:
      exposure:
        include: "health,info,metrics"
        exclude: "env,beans"
```
- 접근 설정
  - `management.endpoint.{Endpoint_ID}.access` 속성으로 개별 지정하며, `management.endpoints.access.default` 속성으로 기본값을 지정할 수 있음
  - 지정 가능한 값
    - unrestricted: 모든 접근 허용
    - read-only: 읽기 전용으로만 허용(웹 Endpoint의 경우 GET/HEAD 요청만 허용)
    - none: 모든 접근 차단
  - 기본적으로 shutdown, heapdump을 제외한 나머지 Endpoint는 접근이 제한되지 않음
    - shutdown은 애플리케이션을 종료시키는 기능이므로, 사용하려면 명시적으로 허용해줘야 함
  - `management.endpoints.access.max-permitted` 속성을 통해 전체 Endpoint의 접근 수준 상한선을 지정할 수 있음
```
application.yml

management:
  endpoints:
    access:
      default: none
  endpoint:
    loggers:
      access: read-only
    shutdown:
      access: unrestricted
```

<br>

### health Endpoint
- 애플리케이션의 상태를 확인할 수 있는 Endpoint로써, 기본적으로 아래와 같이 전체 상태만 응답함
```
GET /actuator/health

{"status":"UP"}
```
- 상태값으로 UP, DOWN, OUT_OF_SERVICE, UNKNOWN을 가짐
- HealthIndicator
  - 개별 구성요소의 상태를 확인하여 제공하는 인터페이스
  - classpath에 존재하는 라이브러리에 따라 자동으로 등록됨
    - db: DataSource를 통한 DB 연결 상태 확인
    - diskSpace: 디스크 여유 공간 확인
    - ping: 항상 UP을 반환
    - redis, mongo, elasticsearch, rabbit, mail 등 연동한 외부 시스템에 대한 상태 확인
  - HealthIndicator 인터페이스를 직접 구현하여 애플리케이션에 맞는 상태 확인 로직을 추가할 수 있음
```
@Component
public class MyHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check();

        if (errorCode != 0) {
            return Health.down()
                    .withDetail("Error Code", errorCode)
                    .build();
        }

        return Health.up().build();
    }

}
```
- show-details
  - 개별 구성요소의 상세 상태를 응답에 포함시킬 것인지 결정하는 `management.endpoint.health.show-details` 속성
    - never(기본값): 상세 정보를 노출하지 않음
    - when-authorized: 인가된 사용자에게만 노출함. `management.endpoint.health.roles` 속성으로 대상 권한 지정 가능
    - always: 모든 사용자에게 노출함
  - 상세 정보에는 DB 접속 정보와 같이 민감한 내용이 포함될 수 있으므로, 운영 환경에서는 never 또는 when-authorized를 사용해야 함
- Health Group
  - 여러 HealthIndicator를 목적에 따라 묶어서 별도의 경로로 제공할 수 있는 기능
  - `/actuator/health/{그룹명}` 경로로 조회 가능함
- Liveness/Readiness Probe
  - Kubernetes 환경에서 사용하는 상태 확인용 Health Group
    - `/actuator/health/liveness`: 애플리케이션이 살아있는지 확인. 실패 시 컨테이너가 재시작됨
    - `/actuator/health/readiness`: 애플리케이션이 트래픽을 받을 준비가 되었는지 확인. 실패 시 트래픽 전달에서 제외됨
  - `management.endpoint.health.probes.enabled` 속성으로 활성화 여부를 지정할 수 있음

<br>

### info Endpoint
- 애플리케이션에 대한 정보를 제공하는 Endpoint
- InfoContributor 구현체를 통해 정보를 수집하며, `management.info.{기여자명}.enabled` 속성으로 활성화 여부를 지정함
  - build: Build 정보. Build Tool Plugin을 통해 생성한 메타데이터 파일이 존재할 때 제공됨
  - git: Git Commit 정보. 관련 Plugin을 통해 생성한 메타데이터 파일이 존재할 때 제공됨
  - env, java, os: 각각 환경 속성, Java 정보, OS 정보를 제공하며 기본적으로 비활성 상태이므로 명시적으로 활성화해줘야 함

<br>

### 보안 관련 유의사항
- Actuator의 Endpoint는 애플리케이션 내부 정보를 그대로 노출하므로, 운영 환경에서는 반드시 접근 통제가 이루어져야 함
  - env, configprops, beans, heapdump 등은 설정값이나 메모리 상태를 그대로 드러내므로 외부에 노출되어서는 안 됨
  - 필요한 Endpoint만 선별하여 노출하고, 나머지는 노출하지 않는 것이 기본 원칙임
- Spring Security를 함께 사용하는 경우, Actuator 경로에 대한 인가 설정을 별도로 구성해줘야 함
  - EndpointRequest를 이용하여 Actuator 경로를 지정할 수 있음
- `management.server.port` 속성을 이용하여 서비스 Port와 분리된 별도의 관리용 Port로 Actuator를 구동시킬 수 있음
  - 외부에 공개되는 Port와 분리하여 내부망에서만 접근하도록 구성 가능함

<br>

#### 참고
- Spring 공식문서 <Building an Application with Spring Boot> - https://spring.io/guides/gs/spring-boot
- Spring Boot Reference Documentation <Endpoints> - https://docs.spring.io/spring-boot/reference/actuator/endpoints.html

#### 배워가는 것들
- 의존성 하나만 추가해도 애플리케이션의 상태를 확인할 수 있는 여러 Endpoint가 제공된다는 점을 익힐 수 있었다. 자동 설정이 주는 편의성을 체감할 수 있는 부분이었다
- 노출과 접근이 서로 다른 개념이라는 것을 정리할 수 있었다. 노출되지 않은 Endpoint는 접근 설정과 무관하게 호출할 수 없으므로, 두 설정을 함께 확인해야 한다
- 편리한 만큼 위험할 수 있다는 것을 알게 되었다. 내부 설정값이 그대로 드러나는 Endpoint들이 있기 때문에, 운영 환경에서는 무엇을 열어둘 것인지 반드시 따져보고 사용해야 할 것이다
