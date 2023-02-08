# REST API

* API(Application Programming Interface)
  * 응용 소프트웨어와 시스템 소프트웨어(OS) 간의 통신을 쉽게 해줄 수 있는 인터페이스.
  * 애플리케이션을 빌드하고 통합하기 위한 정의 및 프로토콜 세트를 의미함.

* 라이브러리(Library)
  * API를 기반으로 특정 기능들이 구현된 함수 혹은 메서드들의 집합체를 의미함.

* 프레임워크(Framework)
  * 여러 API와 라이브러리들을 합쳐서 특정 기능들을 제공해주는 골격 및 뼈대를 의미함.  
  * 완성된 하나의 제품을 만들기 위한 기반을 제공해줌.

* 정보은닉(Information Hiding)
  * 객체지향 언어적 요소를 활용하여 인스턴스에 대한 구체적인 정보를 노출시키지 않도록 하는 기법.
  * 정보은닉의 종류
    * 인스턴스의 구체적인 타입 은닉(상위타입 캐스팅)
    * 인스턴스의 필드/메서드 은닉(캡슐화)
    * 구현 은닉(추상 클래스, 인터페이스)

* 캡슐화(Encapsulation)
  * 정보은닉 기법중 하나로써, 클래스에 있는 속성(필드)과 행위(메서드)들을 감싸서 외부 접근을 특정 메서드로만 허용하는 것.
  * Java의 경우, 접근제어자(public, protected, default(생략 가능), private)를 이용하여 필드와 메서드들의 접근을 강제할 수 있다.

* REST(REpresentational State Transfer) 아키텍처
  * 인터넷상의 시스템 간 상호 운용성(Interoperability)을 제공하는 방법 중 하나.
  * 시스템 제각각의 독립적인 진화를 보장하기 위한 방법.

* REST API
  * REST 아키텍처 스타일을 따르는 API.

* REST 아키텍처 스타일
  * Client-Server
  * Stateless
  * Cache
  * Uniform Interface
    * Identification of Resources: URI를 이용한 리소스 식별
    * Manipulation of Resources through Represenations: 표현을 통한 리소스 조작
    * Self-descriptive Messages
      * 메시지 스스로 메시지에 대한 설명이 가능해야 한다.
      * 서버가 변해서 메시지가 변해도 클라이언트는 그 메시지를 보고 해석이 가능하다.
      * 확장 가능한 커뮤니케이션
    * HATEOAS(hypermedia as the engine of appliaction state)
      * 하이퍼미디어(링크)를 통해 애플리케이션 상태 변화가 가능해야한다.
      * Versioning 할 필요 없이 링크 정보를 동적으로 바꿀 수 있다.
    * Layered System
    * Code-On-Demand(optional)
