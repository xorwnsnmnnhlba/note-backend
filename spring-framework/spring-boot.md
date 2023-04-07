# Spring Boot

### Spring Initializr
* 웹 상에서 Spring 프로젝트를 만들어주는 기능을 제공하는 도구.

<br>

### Web Server와 Web Application Server(WAS)
* Web Server
  * 클라이언트의 HTTP 요청을 받아 HTML, CSS, 이미지 등등의 정적 콘텐츠를 제공해주는 서버.
  * Apache, Nginx 등등 여러가지가 있음.
* Web Application Server(WAS)
  * Web Server에 Web Container를 추가하여 비즈니스 로직 처리가 필요한 동적 콘텐츠를 제공해주는 서버.
  * DB나 Zookeeper 등등과 연동하여 로직 수행 및 관련 데이터 처리가 가능함.
  * Tomcat, JBoss, Jetty 등등 여러가지가 있음(여기에서는 Spring Boot에서 기본적으로 사용하는 Tomcat에 대한 설명만 기재).
  * Tomcat
    * Apache 소프트웨어 재단에서 만든 WAS이며, JSP/Servlet이 실행할 수 있는 환경을 제공해줌.

<br>

### Spring Boot
* Spring 기반 애플리케이션을 쉽게 사용하도록 도와주는 도구.
  * 여러 Setting의 복잡성을 최소화하고 비즈니스 로직 구현에 전념할 수 있게 함.
  * 독립형 Application을 만드는데 최적화됨.
* 기본적으로 내장 WAS(Embedded Tomcat)를 가지고 있으며, 설정해놓은 Spring Boot 버전에 따라 구동되는 Tomcat의 Version도 달라짐.
  * Spring Boot 3.0을 이용하여 Tomcat 구동 시, Tomcat 10이 구동됨.
  * 만약 Tomcat이 아닌 다른 WAS를 사용할 경우, 그에 맞춰 설정 수정이 필요함.
* Spring Boot 자체를 서버로 볼수는 없음.
* WAS로 구동하여 사용 가능하도록 자동 설정을 통해 아래 모든 과정을 보다 상세하고 유연하게 설정하여 실행할 수 있도록 도와줌.
  * Tomcat 객체 생성
  * Port 설정
  * Tomcat에 컨텍스트 추가
  * Servlet 만들기
  * Tomcat에 Servlet 추가
  * 컨텍스트에 Servlet 매핑
  * Tomcat 실행 및 대기
