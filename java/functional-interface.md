# 람다식과 함수형 인터페이스

* 람다식(Lambda Expression)
  * Java에서 함수형 프로그래밍(Functional Programming)을 구현하는 방식.
  * 클래스를 생성하지 않고 함수의 호출만으로 기능을 수행. 즉, 익명 함수(Anonymous Function)를 지칭함.
  * 함수형 인터페이스를 선언함.
  * Java 1.8부터 지원되는 기능.
  * 프로그램 내에서 변수처럼 사용할 수 있음.

* 함수형 프로그래밍(Functional Programming)
  * 매개변수만을 사용하여 외부 자료에 부수적인 영향(Side Effect)이 발생하지 않는 순수함수(Pure Function)를 구현하고 호출함.
  * 입력받은 자료를 기반으로 수행되고 외부에 영향을 미치지 않으므로, 병렬처리 등에 가능한 안정성이고 확장성 있는 프로그래밍 방식이라 할 수 있음.

* 함수형 인터페이스(@FunctionalInterface)
  * 구현해야 할 추상 메서드가 오직 하나만 정의된 인터페이스로써, 람다식 호출이 가능함.
  * default 메서드와 static 메서드의 경우, 여러개 정의 가능함.
