# Spring Web MVC

* Model-View-Controller(MVC) 아키텍처 패턴
  * 모델(Model): 데이터 혹은 데이터를 처리하는 영역.
  * 뷰(View): 응답으로 온 결과 화면을 만들어내는데 사용하는 영역.
  * 컨트롤러(Controller): 뷰와 모델의 중간에 위치하여 여러 요청을 처리하는 주체.
  * 동작원리
    1. URL을 가지고 컨트롤러를 호출.
    2. 컨트롤러에서 입력값 검증 후, 모델에 접근하여 필요한 데이터를 가져옴.
    3. 가져온 데이터를 가공하여 뷰에 노출시킴.
  
* 관심사의 분리(Seperation of Concern)
  * 서로 같은 역할을 하는 부분(관심사)들끼리 분리시키는 것.
  * SOLID 원칙 중 하나인 단일책임의 원칙에 부합하기 위함.

* Spring Web MVC
  * MVC 아키텍처 패턴을 기반으로 하여 핸들러에 요청을 보내는 DispatcherServlet을 중심으로 설계된 프레임워크.
  * 기본 핸들러의 경우, @Controller와 @RequestMapping을 기반으로 하고 있으며 광범위하고 유연한 핸들러 메서드를 제공함.

* Java Annotation
  * Java 5부터 사용한 기능으로써, 데이터에 대한 데이터인 메타데이터(Metadata)를 추가할 때 사용.
  * @Override
    * 부모 클래스에 있는 메서드를 다시 정의하겠다는 의미로 사용. 해당 메서드가 부모 클래스에 존재하지 않는 경우, 컴파일 오류 발생.

* Spring Annotation
  * @Controller
    * 컨트롤러를 구현하는 클래스에 사용하는 애노테이션.
  * @RestController
    * @Controller에서 결과값으로 뷰를 만들지 않고 데이터 자체를 반환할 때 사용하는 애노테이션. JSON 형식으로 반환하여 많이 사용함.
    * @ResponseBody
      * HttpMessageConverter를 사용하여 데이터를 응답 본문 메시지로 보낼 때 사용하는 애노테이션.
      * @RestController 애노테이션이 사용된 클래스의 모든 메서드는 @ResponseBody 애노테이션이 기본으로 내장되어 있음.
  * @RequestMapping
    * 클라이언트를 통해 들어온 요청을 임의의 메서드와 매핑하기 위해 사용하는 애노테이션.
    * Request 키워드 대신 앞부분에 HTTP 메서드를 넣어서 사용할 수 있음. -> @GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping
