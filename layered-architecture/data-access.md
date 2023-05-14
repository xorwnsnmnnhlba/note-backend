# Data Access

### DAO(Data Access Object)
* 영속성을 가진 모듈인 DB에 접근하여 데이터를 처리하는 객체를 의미함.
* JPA에서는 DB에 직접 접근하는 Repository 인터페이스의 구현체로 만든 인스턴스들이 DAO 역할을 한다고 볼 수 있음.
* 영속성 계층에 위치하고 있으며, DB와 연동하여 CRUD 작업만 수행함. 그에 따른 비즈니스 로직은 비즈니스 계층의 Service 클래스에서 수행함.

<br>

### List
* 인덱스(Index) 번호에 따른 순차적 데이터의 집합.
* 크기가 정적으로 정해진 배열과 다르게, 특정 타입에 해당하는 데이터를 동적으로 채워넣어줄 수 있는 기능 제공.
* List와 Set의 공통적 요소를 담아놓은 Collection 인터페이스의 자식 인터페이스이며, 구현체로 ArrayList와 LinkedList가 있음.
* ArrayList
  * 배열을 활용한 선형 리스트로써, 크기가 가변적으로 변하는 List 인터페이스의 구현체.
* LinkedList
  * ArrayList와 다르게 각각의 데이터를 노드로 간주하여, 그 노드를 연결하여 구성함.
  * 모든 노드가 서로 연결되어 있으므로, 각각의 노드에는 이전/다음 값에 대한 정보를 가지고 있음.

<br>

### Map
* Key-Value 형태로 구성된 데이터의 집합.
* 리스트처럼 특정 타입에 해당하는 데이터를 동적으로 채워넣어줄 수 있는 기능 제공.
* 리스트와 마찬가지로 컬렉션(Collection) 인터페이스의 일종이며, 구현체로 HashMap을 가장 많이 사용함.
* HashMap
  * 해시테이블 방식을 사용한 Map 인터페이스의 구현체.
  * Key를 통해 해당하는 Value에 대한 빠른 접근이 가능함.
  * 데이터 연결 순서를 보장하는 LinkedHashMap도 있으며, 이 클래스의 경우 HashMap을 상속받음.

<br>

#### 참고
* https://medium.com/@jang.wangsu/설계-용어-응집도와-결합도-b5e2b7b210ff
* https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html
* https://github.com/ulid/spec
* https://github.com/f4b6a3/tsid-creator

#### 배워가는 것들
* 프로그램에서 사용하는 데이터의 영속화를 위해 필요한 개념인 DAO에 대해 학습했다.
* DAO를 통해 가져온 데이터를 분류하기 위해 필요한 자료구조인 List와 Map에 대해 학습했다. Stream과 함께 자주 쓰일테니, 막힘없이 잘 활용할 수 있도록 해야겠다.
