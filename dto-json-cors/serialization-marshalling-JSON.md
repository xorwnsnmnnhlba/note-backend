# 직렬화(Serialization), Marshalling, JSON

### 직렬화(Serialization)
- 객체 내부에 있는 내용을 직접 입출력할 수 있도록 데이터를 Byte Stream으로 변환해주는 기능
- 네트워크를 통해 객체 자체를 전송하지는 못하므로, 이를 전송할 수 있는 데이터로 변환해줘야 함
- Marshalling의 한 방법
- Java의 경우, Serializable 인터페이스를 구현해야 함

<br>

### 역직렬화(Deserialization)
- Byte Stream으로 들어온 데이터를 프로그램에서 인지할 수 있는 객체로 변환해주는 기능
- Unmarshalling의 한 방법

<br>

### Marshalling
- 직렬화와 유사하며, 전송 가능한 데이터로 변환하는 과정을 의미함

<br>

### Unmarshalling
- 전송받은 데이터를 프로그램이 인지할 수 있는 데이터로 변환하는 과정을 의미함

<br>

### JSON(JavaScript Object Notation)
- Javascript 객체 문법으로 구조화된 데이터를 표현하기 위한 문자 기반의 표준 포맷
- 웹 상에서 데이터를 전송할 때 주로 사용함
- Key-Value 형식으로 이루어져 있으며, Key/Value 양쪽에 "(큰따옴표)를 붙임
- 사람이 이해하기 쉽고, 기계 입장에서도 해석이 용이함
- {}(중괄호)를 이용하여 하나의 객체를 구성하고, \[\](대괄호)를 이용하여 여러 값을 구성할 수 있음
- JavaScript로 구현된 Frontend와 Java로 구현된 Backend와의 통신을 위해 사용함
  - Frontend에서 데이터 전송 시 JSON.stringify를 이용하여 직렬화 후 Backend로 넘겨주게 됨
  - Backend로부터 전송된 데이터를 받아서 사용할 때는 JSON.parse를 이용하여 역직렬화 후 사용함

<br>

### Jackson
- Java에서 인스턴스를 JSON 형태의 데이터로 변환(직렬화) 및 그 반대로 변환(역직렬화)할 수 있는 기능을 제공해주는 라이브러리
- 변환 작업을 수행하는 클래스를 Mapper라 하며, Jackson Version에 따라 사용하는 클래스와 패키지가 다름
  - Jackson 2: com.fasterxml.jackson.databind 패키지에 있는 ObjectMapper
  - Jackson 3: tools.jackson.databind 패키지에 있는 JsonMapper
- Spring Boot 4.x부터 Jackson 3가 기본으로 사용되며, 자동 설정을 통해 JsonMapper가 Bean으로 등록됨
  - Jackson 2도 사용 가능하지만 Deprecated 처리되었으며, Jackson 3로의 이전을 돕기 위한 목적으로만 제공되므로 향후 4.x 버전에서 제거될 예정임
  - 두 Version이 모두 존재하는 경우, spring.http.converters.preferred-json-mapper 속성으로 사용할 Version을 지정할 수 있음
- Jackson은 spring-boot-starter-json에 포함되어 있으며, spring-boot-starter-webmvc를 추가하면 함께 따라오므로 별도 설정 없이 DTO를 JSON 형식의 데이터로 자동 변환해주는 기능을 사용할 수 있음(Spring Boot 4.x 이전에는 spring-boot-starter-web)

<br>

### HttpMessageConverter
- 요청 본문(Request Body)을 Java 객체로, Java 객체를 응답 본문(Response Body)으로 변환해주는 인터페이스
- @RequestBody와 @ResponseBody가 선언된 데이터의 변환을 담당하며, 요청의 Content-Type 헤더와 Accept 헤더를 보고 사용할 구현체를 결정함
  - 컨트롤러에서 반환한 객체가 JSON 문자열로 바뀌어 응답 본문에 담기는 과정이 이 인터페이스의 구현체를 통해 이루어짐
- 주요 구현체
  - JacksonJsonHttpMessageConverter: Jackson을 이용하여 JSON을 변환함. Spring Framework 6 이전에는 MappingJackson2HttpMessageConverter가 그 역할을 수행함
  - StringHttpMessageConverter: 문자열을 변환함
  - ByteArrayHttpMessageConverter: 바이트 배열을 변환함
  - FormHttpMessageConverter: 폼 데이터를 변환함
  - JacksonXmlHttpMessageConverter: XML을 변환함

<br>

### @JsonProperty
- Mapper를 이용하여 DTO를 JSON 데이터로 변환 시, 전달이 이루어지는 JSON 데이터 Key가 Dto의 필드와 다른 경우 사용하는 애노테이션

<br>

### @JsonIgnoreProperties
- JSON 데이터를 객체로 변환(역직렬화) 시, 매핑 대상이 아닌 필드를 어떻게 처리할 것인지 지정하는 애노테이션
- ignoreUnknown 속성을 true로 지정하면, 객체에 선언되지 않은 필드가 JSON에 포함되어 있어도 무시하고 변환을 수행함
  - 지정하지 않은 상태에서 선언되지 않은 필드가 들어오는 경우, 변환에 실패하여 예외가 발생함
- 외부 API를 호출하여 받은 응답을 객체로 변환할 때 특히 유용함
  - 응답에는 사용하지 않는 필드가 함께 담겨오는 경우가 많으며, 필요한 필드만 선언하여 사용할 수 있음
  - API 제공측에서 응답 필드를 추가하더라도 호출하는 쪽이 깨지지 않음
```
@JsonIgnoreProperties(ignoreUnknown = true)
public record Quote(String type, Value value) {

}
```
- 활용 사례는 [RestClient](/spring-framework/rest-client.md) 참고

<br>

#### 참고
- Spring 공식문서 <Building a RESTful Web Service> - https://spring.io/guides/gs/rest-service
- Spring Boot Reference Documentation <JSON> - https://docs.spring.io/spring-boot/reference/features/json.html

#### 배워가는 것들
- 백엔드 모듈을 구현할 때 데이터 흐름을 머릿속으로 잘 생각하며 구현하기 위해 필요한 개념들을 학습했다.
  - 메시지를 이용하여 백엔드와 프론트엔드 간 통신을 할 때 가장 많이 사용하는 포맷인 JSON에 대해 학습했다.
  - 데이터를 주고받을 때 사용하는 개념이라 할 수 있는 직렬화와 Marshalling에 대해 학습했다.
- 컨트롤러가 반환한 객체가 JSON으로 바뀌는 과정을 HttpMessageConverter가 담당한다는 것을 익힐 수 있었다. 자동으로 이루어지는 변환이라 하더라도 그 주체가 무엇인지는 알고 있어야 할 것이다.
- Jackson이 3으로 올라가면서 패키지와 Mapper 클래스가 바뀌었다는 것을 알게 되었다. 라이브러리의 Major Version 변경은 패키지 경로까지 바뀔 수 있으므로, 의존성 Version을 올릴 때 반드시 확인해야 한다.
