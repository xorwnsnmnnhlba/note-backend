# TDD, BDD

### 테스트 주도 개발(TDD, Test-Driven Development)
* 애플리케이션 로직 작성 전, 테스트 코드를 먼저 작성 후 그에 맞춰 로직을 구현하고 이를 검증하는 방안.
* 아래 사이클에 맞춰서 진행하게 된다. 이를 Red-Green-Refactor 주기라 한다.
  * Red 단계: 요구사항에 따른 테스트 케이스가 포함된 테스트 코드 작성.
  * Green 단계: 작성한 테스트 코드를 통과할 수 있는 실제 코드 작성. 
  * Refactor 단계: 중복된 코드 제거 및 리팩터링.
* 반드시 순서대로 진행해야 하며, 단계를 건너뛰지 않아야한다. 이를 통해 지엽적인 설계/구현을 줄여서 코드 자체의 품질 및 안정성을 높일 수 있게 된다.

<figure><img src="./images/tdd-cycle.webp" alt=""></figure>

<br>

### 행위 주도 개발(BDD, Behavior-Driven Development)
* 비즈니스와 사용자 관점에서 애플리케이션이 어떻게 행동하는지에 대한 명세를 우선 구성하고, 그에 맞춰 구현 후 검증을 진행하는 방안.
* TDD에서 파생됨.

<br>

### Given-When-Then 패턴
* 테스트 코드를 작성하는 표현 방식으로써, 준비 - 실행 - 검증 단계를 거침.
* "이러이러한 코드가 주어지고(Given) 이렇게 실행(When)되었을 때, 이러이러한 결과(Then)가 나올 것이다." 라는 의미를 가지게 됨.
* Mockito에서는 BddMockito라는 클래스를 통해 BDD 스타일의 API를 제공함.

<br>

#### 참고
* https://www.samsungsds.com/kr/insights/test-driven-development.html
* https://marsner.com/blog/why-test-driven-development-tdd/

<br>

#### 배워가는 것들
* TDD의 중요성과 이를 수행하는 방법을 익힐 수 있었다.
* 단순히 테스트 코드를 작성하는 것이 TDD의 전부라 할 수는 없다. 테스트 코드 자체를 작성하는 건 TDD 과정 중의 하나일 뿐이다.
* 중요한 것은 테스트 코드와 실제 코드 작성 순서를 엄격하게 지켜가면서 진행하는 것이다. 이것이 TDD의 핵심이다.
