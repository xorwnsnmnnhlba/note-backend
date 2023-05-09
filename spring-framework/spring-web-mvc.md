# Spring Web MVC

### Model-View-Controller(MVC) 아키텍처 패턴
* 모델(Model): 데이터 혹은 데이터를 처리하는 영역.
* 뷰(View): 응답으로 온 결과 화면을 만들어내는데 사용하는 영역.
* 컨트롤러(Controller): 뷰와 모델의 중간에 위치하여 여러 요청을 처리하는 주체.
* 동작원리
  1. URL을 가지고 컨트롤러를 호출.
  2. 컨트롤러에서 입력값 검증 후, 모델에 접근하여 필요한 데이터를 가져옴.
  3. 가져온 데이터를 가공하여 뷰에 노출시킴.

<br>

### Spring Web MVC
* MVC 아키텍처 패턴을 기반으로 하여 핸들러에 요청을 보내는 DispatcherServlet을 중심으로 설계된 프레임워크.
* 기본 핸들러의 경우, @Controller와 @RequestMapping을 기반으로 하고 있으며 광범위하고 유연한 핸들러 메서드를 제공함.

<br>

### Java Annotation
* Java 5부터 사용한 기능으로써, 데이터에 대한 데이터인 메타데이터(Metadata)를 추가할 때 사용.
* @Override
  * 부모 클래스에 있는 메서드를 다시 정의하겠다는 의미로 사용. 해당 메서드가 부모 클래스에 존재하지 않는 경우, 컴파일 오류 발생.

<br>

### Spring Annotation
* @Controller
  * 컨트롤러를 구현하는 클래스에 사용하는 애노테이션.
* @RestController
  * @Controller에서 결과값으로 뷰를 만들지 않고 데이터 자체를 반환할 때 사용하는 애노테이션. JSON 형식으로 반환하여 많이 사용함.
* @RequestBody
  * 요청 본문 내용을 받아서 메서드 변수로 보낼 때 사용하는 애노테이션.
* @ResponseBody
  * HttpMessageConverter를 사용하여 데이터를 응답 본문 메시지로 보낼 때 사용하는 애노테이션.
  * @RestController 애노테이션이 사용된 클래스의 모든 메서드는 @ResponseBody 애노테이션이 기본으로 내장되어 있음.
* @PathVariable
  * URI에 있는 템플릿 변수를 읽을 때 사용. 메서드 변수명 앞에 @PathVariable("템플릿_변수명")처럼 넣어서 사용 가능.  
  * 메서드 변수명과 템플릿 변수명이 서로 같은 경우, 템플릿 변수명 생략 가능함.
* @RequestMapping
  * 클라이언트를 통해 들어온 요청을 임의의 메서드와 매핑하기 위해 사용하는 애노테이션.
  * Request 키워드 대신 앞부분에 HTTP 메서드를 넣어서 사용할 수 있음. @RequestMapping(method=RequestMethod.{METHOD_NAME})과 같은 의미를 가짐.
    * @GetMapping: GET 요청이 들어왔을 때 사용하는 애노테이션.
    * @PostMapping: POST 요청이 들어왔을 때 사용하는 애노테이션.
    * @PutMapping: PUT 요청이 들어왔을 때 사용하는 애노테이션.
    * @PatchMapping: PATCH 요청이 들어왔을 때 사용하는 애노테이션.
    * @DeleteMapping: DELETE 요청이 들어왔을 때 사용하는 애노테이션.
* @ExceptionHandler
  * Spring Web MVC에서 예외처리 시에 사용하는 애노테이션.
* @ResponseStatus
  * 응답으로 온 HTTP 상태코드에 대한 처리를 진행하는 애노테이션. 예외처리 시, @ExceptionHandler와 함께 주로 사용함.

#### 배워가는 것들
* MVC 패턴에 대해 익힐 수 있었다. 들어온 요청을 어떻게 처리하여 보여주는지에 대한 전반적인 원리를 익힐 수 있었다.
* Spring Web MVC에서 쓰이는 여러 애노테이션들의 용법을 익힐 수 있었다. 수없이 마주치게 될 애노테이션이기 때문에, 정확하게 알고 사용해야 할 것이다.
