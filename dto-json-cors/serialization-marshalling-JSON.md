# 직렬화(Serialization), Marshalling, JSON

* 직렬화(Serialization)
  * 객체 내부에 있는 내용을 직접 입출력할 수 있도록 데이터를 Byte Stream으로 변환해주는 기능.
  * Java의 경우, Serializable 인터페이스를 구현해야 함.
  
* 역직렬화(Deserialization)
  * Byte Stream으로 들어온 데이터를 프로그램에서 인지할 수 있는 객체로 변환해주는 기능.

* Marshalling
  * 직렬화와 유사하며, 전송 가능한 데이터로 변환하는 과정을 의미함.

* Unmarshalling
  * 전송받은 데이터를 프로그램이 인지할 수 있는 데이터로 변환하는 과정을 의미함.

* JSON(JavaScript Object Notation)
  * Javascript 객체 문법으로 구조화된 데이터를 표현하기 위한 문자 기반의 표준 포맷.
  * 웹 상에서 데이터를 전송할 때 주로 사용함.
  * Key-Value 형식으로 이루어져 있으며, Key/Value 양쪽에 "(큰따옴표)를 붙임.
  * {}(중괄호)를 이용하여 하나의 객체를 구성하고, [](대괄호)를 이용하여 여러 값을 구성할 수 있음.
