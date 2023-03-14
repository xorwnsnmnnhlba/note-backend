# TDD, BDD

* 테스트 주도 개발(TDD, Test-Driven Development)
	* 애플리케이션 로직 작성 전, 테스트 코드를 먼저 작성 후 그에 맞춰 로직을 구현하고 이를 검증하는 방안.
	* 요구사항에 따른 테스트 케이스를 작성하고, 이를 통과할 수 있는 테스트 코드를 작성한다. 그리고, 그 코드를 리팩터링하여 진행한다.

<br>

* 행위 주도 개발(BDD, Behavior-Driven Development)
	* 비즈니스와 사용자 관점에서 애플리케이션이 어떻게 행동하는지에 대한 명세를 우선 구성하고, 그에 맞춰 구현 후 검증을 진행하는 방안.
	* TDD에서 파생됨.

<br>

* Given-When-Then 패턴
	* 테스트코드를 작성하는 표현 방식으로써, 준비 - 실행 - 검증 단계를 거침.
	* "이러이러한 코드가 주어지고(Given) 이렇게 실행(When)되었을 때, 이러이러한 결과(Then)가 나올 것이다." 라는 의미를 가지게 됨.
	* Mockito에서는 BddMockito라는 클래스를 통해 BDD 스타일의 API를 제공함.