# Spring Boot

### Spring Initializr
- 웹 상에서 Spring 프로젝트를 만들어주는 기능을 제공하는 도구

<br>

### Web Server와 Web Application Server(WAS)
- Web Server
  - 클라이언트의 HTTP 요청을 받아 HTML, CSS, 이미지 등등의 정적 콘텐츠를 제공해주는 서버
  - Apache, Nginx 등등 여러가지가 있음
- Web Application Server(WAS)
  - Web Server에 Web Container를 추가하여 비즈니스 로직 처리가 필요한 동적 콘텐츠를 제공해주는 서버
  - DB나 Zookeeper 등등과 연동하여 로직 수행 및 관련 데이터 처리가 가능함
  - Tomcat, JBoss, Jetty 등등 여러가지가 있음(여기에서는 Spring Boot에서 기본적으로 사용하는 Tomcat에 대한 설명만 기재)
  - Tomcat
    - Apache 소프트웨어 재단에서 만든 WAS이며, JSP/Servlet이 실행할 수 있는 환경을 제공해줌

<br>

### Spring Boot
- Spring 기반 애플리케이션을 쉽게 사용하도록 도와주는 도구
  - 여러 Setting의 복잡성을 최소화하고 비즈니스 로직 구현에 전념할 수 있게 함
  - 독립형 Application을 만드는데 최적화됨
- 기본적으로 내장 WAS(Embedded Tomcat)를 가지고 있으며, 설정해놓은 Spring Boot 버전에 따라 구동되는 Tomcat의 Version도 달라짐
  - Spring Boot 4.x를 이용하여 Tomcat 구동 시, Tomcat 11이 구동됨(Servlet 6.1 기반)
  - 만약 Tomcat이 아닌 다른 WAS를 사용할 경우, 그에 맞춰 설정 수정이 필요함(Jetty 12.1 등)
- Spring Boot 4.x 기준 요구사항
  - Java 17 이상(Java 26까지 호환), Spring Framework 7 이상 기반
  - Build Tool은 Gradle 8.14 이상(8.x 또는 9.x) 또는 Maven 3.6.3 이상
  - Servlet 6.1 이상을 지원하는 컨테이너에 배포 가능
- Spring Boot 자체를 서버로 볼수는 없음
- WAS로 구동하여 사용 가능하도록 자동 설정을 통해 아래 모든 과정을 보다 상세하고 유연하게 설정하여 실행할 수 있도록 도와줌
  - Tomcat 객체 생성
  - Port 설정
  - Tomcat에 컨텍스트 추가
  - Servlet 만들기
  - Tomcat에 Servlet 추가
  - 컨텍스트에 Servlet 매핑
  - Tomcat 실행 및 대기

<br>

### Starter 의존성
- 특정 기능을 구현하는데 필요한 의존성들을 미리 묶어놓은 의존성 집합(Curated Dependency Set)
  - 개별 라이브러리를 하나하나 찾아 추가할 필요 없이, Starter 하나만 추가하면 관련 의존성이 함께 따라옴
  - 각 의존성의 Version을 Spring Boot가 관리해주므로, 라이브러리 간 Version 충돌을 신경쓰지 않아도 됨
  - `spring-boot-starter-{기능명}` 형태의 명명규칙을 가짐
- 주요 Starter
  - spring-boot-starter-webmvc: Spring MVC와 Tomcat을 이용하여 Web/RESTful 애플리케이션을 구현할 때 사용
    - 기존에 사용하던 spring-boot-starter-web은 Spring Boot 4.x부터 spring-boot-starter-webmvc로 대체되면서 Deprecated 처리됨
  - spring-boot-starter-data-jpa: Spring Data JPA와 Hibernate를 사용할 때 추가
  - spring-boot-starter-jdbc: JDBC를 사용할 때 추가
  - spring-boot-starter-actuator: 애플리케이션의 상태 확인 및 모니터링 기능을 사용할 때 추가
- Test용 Starter
  - spring-boot-starter-test: JUnit Jupiter, Hamcrest, Mockito 등 테스트에 필요한 라이브러리를 포함하는 기본 Starter
  - Spring Boot 4.x부터는 모듈 단위의 Test Starter가 함께 제공되므로, 테스트 대상에 맞춰 선택 가능함
    - spring-boot-starter-webmvc-test: Spring MVC와 Tomcat에 대한 테스트 수행 시 사용
    - spring-boot-starter-web-server-test: Web Server 자체에 대한 테스트 수행 시 사용
```
build.gradle

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-webmvc'
    testImplementation 'org.springframework.boot:spring-boot-starter-webmvc-test'
}
```

<br>

### 자동 설정(Auto-configuration)
- classpath에 어떤 라이브러리가 존재하는지, 어떤 빈이 이미 등록되어있는지, 어떤 속성이 설정되어있는지를 판단하여 필요한 빈을 자동으로 등록해주는 기능
  - 조건이 충족될 때에만 설정이 적용되는 조건부 설정(Conditional Configuration) 방식으로 동작함
  - 예를 들어 spring-webmvc가 classpath에 존재하면 해당 애플리케이션을 웹 애플리케이션으로 간주하고, DispatcherServlet 등록과 같은 핵심 동작을 활성화함
  - 내장 Tomcat 구동 역시 자동 설정의 결과물이며, 위에 기재한 Tomcat 객체 생성부터 실행까지의 과정을 개발자 대신 수행해줌
- 개발자가 직접 빈을 등록하거나 속성을 설정한 경우, 자동 설정보다 우선하여 적용됨
  - 즉 자동 설정은 기본값을 제공하는 역할이며, 필요한 부분만 선택적으로 재정의(Override)하여 사용할 수 있음
- XML 설정이나 인프라 구성에 해당하는 상용구 코드(Boilerplate)를 제거하여, 비즈니스 로직 구현에 집중할 수 있게 해줌

<br>

### @SpringBootApplication
- Spring Boot 애플리케이션의 시작점이 되는 메인 클래스에 선언하는 애노테이션
- 아래 3개의 애노테이션을 하나로 묶어놓은 편의성 애노테이션(Convenience Annotation)임
  - @Configuration
    - 해당 클래스를 애플리케이션 컨텍스트에 등록할 빈 정의(Bean Definition)의 소스로 지정함
  - @EnableAutoConfiguration
    - classpath 설정, 이미 등록된 다른 빈, 여러 속성 설정을 기반으로 필요한 빈을 추가하도록 지시함
    - 예를 들어 spring-webmvc가 classpath에 있으면 해당 애플리케이션을 웹 애플리케이션으로 지정하고, DispatcherServlet 설정과 같은 핵심 동작을 활성화함
  - @ComponentScan
    - 해당 클래스가 위치한 패키지(예: com/example) 및 그 하위에서 다른 컴포넌트, 설정, 서비스를 찾도록 지시함. 이를 통해 컨트롤러를 찾아 빈으로 등록할 수 있음
    - 메인 클래스를 최상위 패키지에 두어야 하위 패키지 전체를 스캔할 수 있음
```
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
```
- main 메서드에서 호출하는 SpringApplication.run()이 애플리케이션 컨텍스트를 생성하고 내장 WAS를 구동시킴

<br>

### 애플리케이션 실행
- Build Tool의 Plugin을 이용하여 실행
  - Gradle: `./gradlew bootRun`
  - Maven: `./mvnw spring-boot:run`
- 실행 가능한 JAR(Executable JAR)로 Packaging하여 실행
  - 애플리케이션 코드와 의존 라이브러리, 내장 WAS까지 하나의 JAR에 포함시키므로 별도의 WAS 설치 없이 독립 실행이 가능함
  - Gradle: `./gradlew build` 수행 후 `java -jar build/libs/{프로젝트명}-{버전}.jar`
  - Maven: `./mvnw package` 수행 후 `java -jar target/{프로젝트명}-{버전}.jar`
- 실행 시 콘솔 로그를 통해 자동 설정으로 등록된 빈과 구동된 내장 Tomcat의 Port를 확인할 수 있음

<br>

#### 참고
- Spring 공식문서 <Building an Application with Spring Boot> - https://spring.io/guides/gs/spring-boot

#### 배워가는 것들
- Spring Boot에 대한 기초개념을 정리할 수 있었다. 실무에서 자주 쓰이는 프레임워크라 하더라도, 그에 대한 정의와 기초개념을 소홀히하지 않아야한다는 것을 일깨워주었다
- 여러 노트에서 습관적으로 추가하기만 했던 Starter 의존성이 어떤 역할을 하는지 정리할 수 있었다. Version 관리까지 대신 해준다는 점을 알게 되면서, 의존성 추가 시 무엇을 확인해야하는지 감을 잡을 수 있었다
- 자동 설정이 조건부로 동작하며 개발자의 설정이 우선한다는 원리를 익힐 수 있었다. 프레임워크가 알아서 해주는 영역이라 하더라도, 그 동작 원리를 알아야 문제가 생겼을 때 대응할 수 있을 것이다
- @SpringBootApplication이 3개의 애노테이션을 묶어놓은 것이라는 점을 정리하면서, 메인 클래스의 위치가 왜 중요한지(@ComponentScan의 스캔 범위)까지 함께 이해할 수 있었다
