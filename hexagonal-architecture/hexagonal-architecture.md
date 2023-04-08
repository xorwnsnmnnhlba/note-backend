# Hexagonal Architecture

### Hexagonal Architecture
* 애플리케이션 내에 존재하는 도메인 객체에 접근하기 위한 설계요소. 포트와 어댑터 아키텍처(Ports and Adapters Architecture)라고도 함.
* 여기서 포트는 접근을 위한 애플리케이션 인터페이스라 볼 수 있으며, 어댑터는 사용자에게 제공할 인터페이스를 애플리케이션 인터페이스로 매칭시켜주는 역할을 수행하는 매개체라 볼 수 있음.
* Hexagonal이라는 것은 문자 그대로 표현하면 육각형인데, 육각형이라는 것이 중요한게 아니라 포트와 어댑터를 추가할 수 있는 공간을 어느정도 자유롭게 확보할 수 있다는 것을 의미함.
* 사용자가 직접 조작하는 UI Layer와 비즈니스 로직들이 있는 애플리케이션 Layer와의 혼선을 막기 위해 포트라는 인터페이스를 통해 통신이 가능하도록 설계할 수 있음.
* 여러 API들을 제공해주는 하나의 Backend Module을 구현하고, WebApp과 MobileApp으로 구현된 클라이언트 단에서 이 Module의 API들을 호출할 수 있도록 구현할 수 있음.
* 클라이언트 단에서 Backend Module API로 연결할 때 사용하는 포트를 Incoming Port라 하며, API에서 DB나 Zookeeper 같은 내부 데이터 저장소로 연결할 때 사용하는 포트를 Outgoing Port라 함.
  * 이때, 클라이언트 입장에서는 Backend Module의 저장소가 어떠한 것인지는 알 필요가 없게 됨. 설령, 저장소가 바뀐다 하더라도 클라이언트 단의 로직에 따른 영향도는 없음.

<figure><img src="./images/hexagonal-architecture.png" alt=""></figure>

<br>

### POJO(Plain Old Java Object)
* 말 그대로 오래된 방식의 Java 객체로써, 여러 중량 프레임워크에 종속된 무거운 객체에 반발하여 나오게 된 간단한 인스턴스를 지칭함.
* 임의의 클래스를 상속받거나 임의의 인터페이스를 구현하면 안되며, 애노테이션 포함도 해서는 안되는 것이 원칙.
* 그렇지만 실무에서 애노테이션을 포함하지 않고 클래스를 만들기는 현실적으로 어렵기 때문에, 애노테이션을 추가하기 전에 POJO를 만족한다면 POJO로 간주 가능함.
* JavaBeans의 경우, POJO의 변형이라 할 수 있음.

<br>

#### 참고
