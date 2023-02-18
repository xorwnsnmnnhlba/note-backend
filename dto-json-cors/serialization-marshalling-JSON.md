# 직렬화(Serialization), Marshalling, JSON

* 직렬화(Serialization)
  * 객체 내부에 있는 내용을 직접 입출력할 수 있도록 데이터를 Byte Stream으로 변환해주는 기능.
  * 네트워크를 통해 객체 자체를 전송하지는 못하므로, 이를 전송할 수 있는 데이터로 변환해줘야 함.
  * Marshalling의 한 방법.
  * Java의 경우, Serializable 인터페이스를 구현해야 함.

<br>

* 역직렬화(Deserialization)
  * Byte Stream으로 들어온 데이터를 프로그램에서 인지할 수 있는 객체로 변환해주는 기능.
  * Unmarshalling의 한 방법.

<br>

* Marshalling
  * 직렬화와 유사하며, 전송 가능한 데이터로 변환하는 과정을 의미함.

<br>

* Unmarshalling
  * 전송받은 데이터를 프로그램이 인지할 수 있는 데이터로 변환하는 과정을 의미함.

<br>

* JSON(JavaScript Object Notation)
  * Javascript 객체 문법으로 구조화된 데이터를 표현하기 위한 문자 기반의 표준 포맷.
  * 웹 상에서 데이터를 전송할 때 주로 사용함.
  * Key-Value 형식으로 이루어져 있으며, Key/Value 양쪽에 "(큰따옴표)를 붙임.
  * 사람이 이해하기 쉽고, 기계 입장에서도 해석이 용이함.
  * {}(중괄호)를 이용하여 하나의 객체를 구성하고, \[\](대괄호)를 이용하여 여러 값을 구성할 수 있음.
  * JavaScript로 구현된 FrontEnd와 Java로 구현된 Backend와의 통신을 위해 사용함.
    * Frontend에서 데이터 전송 시 JSON.stringify를 이용하여 직렬화 후 Backend로 넘겨주게 됨.
    * Backend로부터 전송된 데이터를 받아서 사용할 때는 JSON.parse를 이용하여 역직렬화 후 사용함.

* Jackson ObjectMapper
  * Java에서 인스턴스를 JSON 형태의 데이터로 변환(직렬화) 및 그 반대로 변환(역직렬화)할 수 있는 기능을 제공해주는 클래스.
  * Spring Boot의 경우, 기본적으로 spring-boot-starter-web에서 DTO를 JSON 형식의 데이터로 자동 변환해주는 기능을 지원함.

* @JsonProperty
  * ObjectMapper를 이용하여 DTO를 JSON 데이터로 변환 시, 전달이 이루어지는 JSON 데이터 Key가 Dto의 필드와 다른 경우 사용하는 애노테이션.
