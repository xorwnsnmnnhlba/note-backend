# Spring Test

* 통합 테스트(Integration Test)
	* 애플리케이션에 있는 각 모듈들이 올바르게 협력하는지 확인하는 테스트.
	* 상당수의 로직들은 단위 모듈별로 수행되는게 아니라 여러 모듈이 협력하여 수행되므로, 모듈간 협력에 이슈가 없는지 반드시 확인해야 함.

<br>

* @Autowired
	* 의존성 주입 시, 필요한 의존 객체의 타입에 해당하는 빈을 찾아 주입해주는 애노테이션.
		* required(기본값: true): 못 찾으면 애플리케이션 구동 실패.
	
<br>

* @SpringBootTest
	* Spring Boot에서 테스트코드 작성을 위해 필요한 애노테이션으로써, @ExtendWith(SpringExtension.class)를 포함하고 있음.
	* @SpringBootApplication이 붙어있는 스프링 메인 애플리케이션을 찾아간 후, 메인에서부터 시작하는 모든 빈들을 스캔함. 그 후 테스트용 애플리케이션 context를 만들면서 모든 빈을 등록해줌.
	* 빈 설정 파일을 알아서 찾아줌(@SpringBootApplication).
	* 테스트코드 작성을 위해 build.gradle에 Test Scope로 spring-boot-starter-test 의존성을 추가해줘야 함.
```
dependencies { 
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

<br>

* Mock
	* 실제 객체와 비슷하게 동작하지만 프로그래머가 직접 그 객체의 행동을 관리하는 객체.
* Mockito
	* Mock 객체를 생성/관리하고 검증할 수 있는 방법을 제공하는 클래스.

<br>

* @SpyBean
	* Spring Boot에서 테스트코드 작성 시, Repository Bean을 가져올 때 주로 사용하는 애노테이션.

<br>

* MockMvc
	* 애플리케이션을 띄우지 않고 Spring MVC의 동작을 수행하여 테스트를 수행할 수 있는 인스턴스를 의미함.
	*	서버사이드 Spring MVC 테스트의 핵심 클래스로써, Spring MVC 테스트를 위한 메인 진입점이라 할 수 있음.
	* 컨트롤러 단의 테스트 시 자주 사용함.
	* @AutoConfigureMockMvc
		* MockMvc 인스턴스 의존성 주입을 수행하기 위해 사용하는 애노테이션.

<br>

* @MockBean
	* 테스트를 위한 Mock 빈 생성시에 사용하는 애노테이션으로써, ApplicationContext에 들어있는 동일타입 빈을 Mock 인스턴스로 교체함.
	* Mockito를 사용해서 Mock 인스턴스를 만들고 빈으로 등록해주며, 모든 테스트마다 자동으로 리셋됨.

<br>

* @WebMvcTest
	* Spring MVC 테스트에 사용할 수 있는 어노테이션으로써, Spring MVC 컴포넌트에만 초점을 맞춤.
	* Spring MVC 테스트에 관련된 구성만 적용되므로, @SpringBootTest에 비해 상대적으로 빠르게 수행할 수 있음.
	* @AutoConfigureMockMvc 애노테이션을 포함하므로, MockMvc 인스턴스 의존성 주입을 자동으로 수행해줌.

<br>

* given
	* org.mockito.BDDMockito에 위치한 정적 메서드로써, 테스트를 위한 조건을 줄 때 사용함.

* verify
	* org.mockito.Mockito에 위치한 정적 메서드로써, 해당하는 테스트에서 관련 메서드가 수행되었는지 확인할 때 사용함.

<br>

* Testcontainers
	* 테스트 수행 시, Docker 컨테이너를 이용하여 DB와의 연동을 수행할 수 있는 라이브러리.
		* JPA 또는 하이버네이트가 만들어주는 쿼리가 DB에 따라 다르기 때문에, 연동이 이루어지는 DB에 맞춰 테스트를 수행해야 할 필요가 있음.
		* 그렇지만 테스트 목적으로 사용하기 위한 DB를 운영하는 것은 번거롭기 때문에, 이를 보완할 수 있는 Tool의 필요성이 대두됨.
		* 실제 운영환경에 더 가까운 테스트 수행이 가능해짐.
		* 기존 H2 Database에 비해 조금 느리게 수행됨.
		* TestContainers에서 junit-jupiter, 운영용 DB 관련 의존성을 추가해야 하며, 관련 구성을 진행해야 함.
		* Docker Daemon이 설치되어있어야 하며, 싱글턴 패턴을 이용하여 모든 테스트에서 오직 한개의 컨테이너만 사용하도록 함.
		* CI/CD Pipeline이 구성된 원격 Repository에 TestContainers를 가지고 테스트를 수행한 프로젝트를 올리는 경우, 그에 맞게 유의하여 구성해줘야 함.
```
build.gradle

dependencies {
    testImplementation 'org.testcontainers:junit-jupiter:1.17.6'
    testImplementation 'org.testcontainers:mysql:1.17.6'
}

application-test.yml

spring:
  datasource:
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
    url: "jdbc:tc:mysql:8.0.31:///studytest"


@Testcontainers
public abstract class AbstractContainerBaseTest {

    private static final MySQLContainer<?> MY_SQL_CONTAINER;

    static {
        MY_SQL_CONTAINER = new MySQLContainer<>("mysql:8.0.31");
        MY_SQL_CONTAINER.start();
    }

}
```

<br>

* 참고
  * 인프런 <스프링 부트 개념과 활용> - 백기선
