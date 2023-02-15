# CORS

* 동일 출처 정책(SOP, Same-Origin Policy)
  * 웹 브라우저를 통해 요청하는 주체(Origin)와 그 응답을 받는 주체가 서로 다른 경우, 그에 따른 접근을 할 수 없게하는 보안정책. 그 주체에는 포트도 포함됨.
    * 즉, 임의의 서버(ex. http://localhost:8080)에서 제공하는 API를 다른 애플리케이션(ex. http://localhost:18080)에서 호출할 수 없게 됨.

<br>

* JSONP(JSON with Padding)
  * JSON 데이터를 가지고 서로 다른 주체끼리 통신할 때, HTML의 script 태그를 이용하여 SOP를 우회하기 위한 방법.
  * HTTP 메서드 중 GET에서만 사용 가능.

<br>

* CORS(Cross-Origin Resource Sharing)
  * 서로 다른 주체끼리 리소스를 공유할 수 있는 기능을 제공하는 표준.
  * 기본 적용된 SOP(Single-Origin Policy)를 우회하기 위한 표준 기술.
  * Frontend, Backend Module간 통신을 위해 관련 설정을 적절하게 가져가야 함.
    * REST API를 이용하여 통신할 때, Backend 쪽에서 응답 헤더에 Access-Control-Allow-Origin 속성을 추가하여 적용함.

<br>

* Spring Boot에서의 CORS 활용
  * 허용하려는 클래스 혹은 메서드 위에 @CrossOrigin 애노테이션을 붙여줘야 함.
  * 모든 컨트롤러에 전역적으로 설정하려는 경우, WebMvcConfigurer 인터페이스를 구현하는 Configuration 클래스에 addCorsMappings 메서드를 오버라이딩하여 관련 설정을 적절하게 넣어줘야 함.
    * addMapping 메서드에 허용할 경로(Path)를 넣어줌.
    * allowedMethods 메서드에 허용할 HTTP 메서드를 넣어줌.
    * allowedOrigins 메서드에 허용할 요청 URL을 넣어줌. 만약 따로 쓰지않는 경우, 모든 요청 URL 허용됨.

```
// 별도의 Configuration 클래스를 선언하여 설정하는 경우
@Configuration 
public class WebConfig implements WebMvcConfigurer { 
    @Override 
    public void addCorsMappings(CorsRegistry registry) { 
        registry.addMapping("/**") 
                .allowedMethods("GET", "POST", "PATCH", "DELETE", "OPTIONS")
                .allowedOrigins("http://localhost:3000"); 
    } 
}
```

```
// @SpringBootApplication 애노테이션이 있는 메인 애플리케이션 클래스에 설정하는 경우
@SpringBootApplication
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @Bean
    public WebMvcConfigurer webMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**");
            }
        };
    }

}
```

<br>

* 참고
  * 인프런 <스프링 부트 개념과 활용> - 백기선
