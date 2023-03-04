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
	* 테스트코드 작성을 위해 build.gradle에 Test Scope로 spring-boot-starter-test를 추가해줘야 함.
```
dependencies { 
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

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
	* 테스트를 위한 가짜 빈 생성시에 사용하는 애노테이션으로써, ApplicationContext에 들어있는 동일타입 빈을 Mock 인스턴스로 교체함.
	* Mockito를 사용해서 Mock 인스턴스를 만들고 빈으로 등록해주며, 모든 테스트마다 자동으로 리셋됨.

<br>

* @WebMvcTest
	*	Spring MVC 테스트에 사용할 수 있는 어노테이션으로써, Spring MVC 컴포넌트에만 초점을 맞춤.
	* Spring MVC 테스트에 관련된 구성만 적용되므로, @SpringBootTest에 비해 상대적으로 빠르게 수행할 수 있음.
	* @AutoConfigureMockMvc 애노테이션을 포함하므로, MockMvc 인스턴스 의존성 주입을 자동으로 수행해줌.

<br>

* verify
	* org.mockito.Mockito에 위치한 메서드로써, 해당하는 테스트에서 관련 메서드가 수행되었는지 확인할 때 사용함.

* 참고
  * 인프런 <스프링 부트 개념과 활용> - 백기선
