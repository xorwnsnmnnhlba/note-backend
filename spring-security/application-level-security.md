# 애플리케이션 수준의 보안

### 인증과 인가
* 인증(Authentication)
  * 사용자의 신원(Identity)을 확인하는 절차로써, 인증을 수행하기 위한 요소를 가지고 진행함.
  * 실생활에서의 사례
    * 은행에 방문하여 계좌에 있는 거액의 돈을 인출하려 할 때, 은행원은 요청자에게 신분증을 요구할 수 있음. -> 인증수행요소: 신분증
    * 무인발급기를 통해 개인정보가 들어있는 문서를 출력할 때, 지문 스캔 과정을 거침. -> 인증수행요소: 지문
  * 컴퓨터 보안에서의 인증
    * 임의의 시스템에 대한 접근권한을 확인하기 위한 절차.
    * ID/Password를 통한 인증을 가장 많이 수행하며, 상황에 따라 SMS를 통한 2단계 인증(Two-Factor Authentication)도 추가로 수행할 수 있음.
    * 최근에는 SMS 인증뿐만 아니라, 인증 전문 소프트웨어(ex. Google OTP, PASS 등등)와 연동하여 인증 절차를 수행하고 있음.
    * 인증을 거치게 되면 시스템과 통신을 하기 위한 경로(세션)가 만들어지게 되며, 정해진 시간동안 인증절차 없이 접근할 수 있음.
* 인가(Authorization)
  * 임의의 시스템에 대한 특정 기능 접근권한을 얻는 작업을 의미함.
  * 실무에서는 주로 관련 세부사항이 담긴 Access Token을 이용함.
    * 사용자가 로그인을 수행하게 되면, 그 사용자 정보에 따른 Access Token이 만들어짐. 이것을 가지고 특정 기능에 대한 접근 허용여부를 결정짓게 됨.
  * 인가를 위해 만들어진 Access Token은 단순히 특정 기능 접근 허용여부에 필요한 데이터이며, 이를 가지고 사용자의 신원을 온전하게 파악할수는 없음.
  * 사례
    * 쇼핑몰에 특정 상품정보를 올리거나 수정하려 할 때, 해당 쇼핑몰 관리자만 올릴 수 있어야 한다. 즉, 관리자 페이지에 대한 접근권한을 부여해야한다.
    * OTT 서비스를 이용할 때, 해당 구독모델에 따라 영상콘텐츠 해상도 설정 접근권한이 달라져야 한다.

<br>

### HTTP Stateless
* HTTP는 클라이언트-서버 간 통신에서 클라이언트의 요청 상태를 보존하지 않음.
* 따라서, Cookie나 Session, LocalStorage 등등을 이용하여 클라이언트 상태를 유지시켜줘야 함.

<br>

### Cookie vs WebStorage
* Cookie
  * Key와 Value 형태의 상태정보 파일. 클라이언트와 서버가 상태정보에 대한 요청과 응답을 지속적으로 주고받으며, 그 정보가 클라이언트에 저장됨.
  * Session Cookie: 메모리에 임시로 저장되는 쿠키로써, 만료시간이 설정되어있음. Non-Persistent Cookie, Temporary Cookie라고도 하며, 브라우저가 종료되면 삭제됨.
  * Persistent Cookie: 유효기간 설정을 통해 장시간 유지될 수 있는 쿠키. 디스크에 저장되기 때문에 브라우저를 종료하거나 브라우저를 띄운 장비를 재시작해도 남아있음. 유효기간 만료 시 파기됨.
* WebStorage
  * Key와 Value 형태로 저장하는 새로운 형태의 상태정보 파일로써, HTML5 명세(Spec)에 포함되어 있음.
  * 기존 쿠키(Cookie)와 유사하지만, 몇가지 차이점이 존재함.
    * HTTP를 통한 통신을 수행할 때 쿠키와 다르게 데이터를 서버로 전송하지 않기 때문에, 네트워크 트래픽에 대한 부담을 줄일 수 있음.
    * 객체 형태로 저장 가능.
  * 지속성과 용량제한에 따라 아래 두 가지로 구분함.
    * LocalStorage: 직접 데이터를 지우지 않는 이상 영구적으로 보존 가능하며, 용량제한이 없음.
    * SessionStorage: 브라우저를 닫을 때까지 저장되며, 5MB의 용량제한을 가짐.

<br>

### Interceptor vs Filter in Spring

<figure><img src="./images/filter-interceptor.png" alt=""></figure>

* Filter
  * J2EE 표준스펙에 있는 기능으로써, 들어온 요청과 처리가 이루어진 응답을 걸러내는 역할을 수행함.
  * Spring MVC의 핵심이라 할 수 있는 DispatcherServlet으로 요청을 전달하기 전에 부가적인 작업을 처리할 수 있는 기능 제공.
  * Spring과 무관하게 웹 컨텍스트(Web Context)에서 동작함.
  * javax.servlet.Filter 인터페이스를 구현해야 하며, 아래와 같이 세가지 메서드를 가지고 있음.
    * init(filterConfig): Filter 인스턴스를 초기화하고 서비스에 추가할 때 사용함.
    * doFilter(request, response, chain): HTTP 요청이 DispatcherServlet에 전달되기 전, 웹 컨테이너에 의해 실행됨. 여기서 FilterChain을 이용하여 다음 대상으로 요청을 전달함.
    * destroy(): Filter 인스턴스를 제거하고 해당하는 리소스를 반환할 때 사용함.
```
public interface Filter {

    public default void init(FilterConfig filterConfig) throws ServletException {}
    
    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;
            
    public default void destroy() {

    }

}
```
* Interceptor
  * Spring에서 제공하는 기술로써, DispatcherServlet에서 컨트롤러(Controller)를 호출하기 전후에 요청을 가로채서 응답을 가공할 수 있는 기능 제공.
  * Filter와 다르게 Spring Context 내부에서 동작함.
  * 핸들러 매핑에 설정할 수 있는 인터셉터인 HandlerInterceptor 인터페이스를 구현해야 하며, 아래와 같이 세가지 메서드를 가지고 있음.
    * preHandle(request, response, handler)
      * 핸들러 실행 전에 호출되며, 핸들러에 대한 정보를 사용할 수 있기 때문에 Filter에 비해 보다 세밀한 로직 구현 가능함.
      * 요청 데이터 전처리가 필요할 때 주로 사용.
    * postHandle(request, response, modelAndView)
      * 핸들러 실행이 끝나고 DispatcherServlet이 뷰를 렌더링하기 전에 호출됨. View에 전달할 추가 모델 객체를 노출시킬 수 있음.
      * 후처리 작업이 필요할 때 주로 사용.
      * 인터셉터 역순으로 호출되며, 비동기적인 요청 처리 시에는 호출되지 않음.
    * afterCompletion(request, response, handler, ex)
      * 요청 처리가 완전히 끝난 후(뷰 렌더링 끝난 후)에 호출됨. preHandle 메서드에서 true를 리턴한 경우에만 호출됨.
      * postHandle 메서드와 마찬가지로 체인의 각 인터셉터에서 역순으로 호출되므로 첫 번째 인터셉터가 가장 마지막에 호출됨.
```
public interface HandlerInterceptor {

    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }
    
    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {

    }
        
    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {

    }
        
}
```

<br>

### 암호화와 복호화
단방향 암호화
Hashing Algorithm
Salt가 필요한 이유

<br>

### 참고
* https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API
* https://auth0.com/intro-to-iam/authentication-vs-authorization
* https://erwinousy.medium.com/쿠키-vs-로컬스토리지-차이점은-무엇일까-28b8db2ca7b2
* https://velog.io/@ejchaid/localstorage-sessionstorage-cookie의-차이점
* https://dev-coco.tistory.com/173
