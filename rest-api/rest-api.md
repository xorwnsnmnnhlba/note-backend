# REST API

### API(Application Programming Interface)
* 응용 소프트웨어와 시스템 소프트웨어(OS) 간의 통신을 쉽게 해줄 수 있는 인터페이스.
* 애플리케이션을 빌드하고 통합하기 위한 정의 및 프로토콜 세트를 의미함.

<br>

### 라이브러리(Library)
* API를 기반으로 특정 기능들이 구현된 함수 혹은 메서드들의 집합체를 의미함.

<br>

### 프레임워크(Framework)
* 여러 API와 라이브러리들을 합쳐서 특정 기능들을 제공해주는 골격 및 뼈대를 의미함.
* 완성된 하나의 제품을 만들기 위한 기반을 제공해줌.

<br>

### 정보은닉(Information Hiding)
* 객체지향 언어적 요소를 활용하여 인스턴스에 대한 구체적인 정보를 노출시키지 않도록 하는 기법.
* 정보은닉의 종류
  * 인스턴스의 구체적인 타입 은닉(상위타입 캐스팅)
  * 인스턴스의 필드/메서드 은닉(캡슐화)
  * 구현 은닉(추상 클래스, 인터페이스)

### 캡슐화(Encapsulation)
* 정보은닉 기법중 하나로써, 클래스에 있는 속성(필드)과 행위(메서드)들을 감싸서 외부 접근을 특정 메서드로만 허용하는 것.
* Java의 경우, 접근제어 지시자(public, protected, default(생략 가능), private)를 이용하여 필드와 메서드들의 접근을 강제할 수 있음.

<br>

### REST(REpresentational State Transfer) 아키텍처
* 인터넷상의 시스템 간 상호 운용성(Interoperability)을 제공하는 방법 중 하나.
* 시스템 제각각의 독립적인 진화를 보장하기 위한 방법.

<br>

### REST API
* REST 아키텍처 스타일을 따르는 API.

<br>

### REST 아키텍처 스타일
* Starting with the Null Style
  * 어떠한 제약조건도 없는 상태에서 필요에 따라 제약조건들을 하나씩 추가해나감.
* Client-Server
  * 클라이언트(Client)가 서버(Server)에 요청을 보내면, 서버는 그에 따른 응답을 클라이언트에게 보내줌.
* Stateless
  * 클라이언트의 애플리케이션 상태를 서버에서 관리하지 않음.
* Cache
  * 서버에서 가져온 리소스를 클라이언트에서 일정시간동안 다시 사용함.
    이를 통해, 서버와 클라이언트 간 통신을 줄여서 네트워크 사용량을 감소시켜 효율적인 처리를 가능하게 함.
* Uniform Interface
  * 구성요소 간 균일한 인터페이스를 강조함으로써, 전체 시스템 아키텍처를 단순화하고 상호 작용의 가시성을 향상시킴.
  * 필딩 제약조건
    * Identification of Resources: URI를 이용한 리소스 식별
    * Manipulation of Resources through Represenations: 표현을 통한 리소스 조작
    * Self-descriptive Messages: 자기 서술 메시지
      * 메시지 스스로 메시지에 대한 설명이 가능해야 함.
      * 서버가 변해서 메시지가 변해도, 클라이언트는 그 메시지를 보고 해석이 가능해야 함.
      * 확장 가능한 커뮤니케이션.
    * HATEOAS(Hypermedia as the Engine of Appliaction State)
      * 하이퍼미디어(링크)를 통해 애플리케이션 상태 변화가 가능해야 함.
      * Versioning할 필요 없이 링크 정보를 동적으로 바꿀 수 있음.
  * HAL(Hypertext Application Language)을 이용하여 Self-descriptive Messages와 HATEOAS를 충족시킬 수 있음.
    * HAL의 _links에 profile 링크를 추가하여 메시지에 대한 설명을 넣을 수 있음.
* Layered System
* Code-On-Demand(optional)

<br>

### MIME(Multipurpose Internet Mail Extensions) type
* 인터넷 상에서 콘텐츠들을 주고받을 때 표현하는 타입. 콘텐츠 타입(Content Type), 미디어 타입(Media Type)이라고도 함.
* HTTP 헤더에 Content-Type 속성으로 전달하며, (type)/(subtype) 형태로 사용함.
* 자주 사용하는 MIME type
  * text/plain: 텍스트 파일에 대한 기본 type이며, E-mail에서 자주 사용함.
  * text/html: 일반적인 HTML 문서를 주고받을 때 사용함.
  * text/css: CSS 파일을 주고받을 때 사용함.
  * text/javascript: JS 파일을 주고받을 때 사용함.
  * application/xml: XML 형태 데이터를 주고받을 때 사용함. Self-descriptive하기 상대적으로 어려운 편.
  * application/json: JSON 형태 데이터를 주고받을 때 사용함. Self-descriptive하기 상당히 어려움.

<br>

#### 참고
* 인프런 <스프링 기반 REST API 개발> - 백기선

#### 배워가는 것들
* API, 라이브러리, 프레임워크에 대한 정의를 익힐 수 있었다.
* REST API에 대한 내용을 익힐 수 있었다. REST 아키텍처를 만족하기 위해 어떠한 것을 수행해야 하는지에 대한 내용을 익힐 수 있었다.
* MIME Type에 대한 내용을 익힐 수 있었다. 파일을 주고받을 때 자주 사용하기 때문에, 각각의 내용들을 잘 알고 적용해야 할 것이다.
