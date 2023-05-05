# CQRS

### CQS(Command Query Separation)
* 명령(삽입/수정/삭제)을 의미하는 Command와 조회를 의미하는 Query를 분리하는 기법.
  * Command -> 상태를 변화시키지만, 그에 따른 결과를 반환시키지 않음.
  * Query -> 상태를 유지시키지만, 그 결과를 반환함.
* 인스턴스의 메서드를 위와 같이 두 가지 유형으로 분리시킴으로써 유지보수성을 향상시킬 수 있음.
* 예외적으로 Stack에서 제공하는 pop의 경우, 상태를 변환시키지만 그에 따른 결과를 반환함. 이처럼 상황에 따라 매우 유용하게 사용할 수 있는 메서드의 경우, CQS를 깨뜨릴수는 있음.

<br>

### CQRS(Command Query Responsibility Segregation)
* 명령(삽입/수정/삭제)을 의미하는 Command와 조회를 의미하는 Query의 역할을 분리하는 기법.
* 전통적인 방식으로 프로그램을 구현할 때, 영속성있는 데이터를 저장하는 DB에 데이터를 삽입/수정/삭제하고 이를 조회하는 CRUD 기법을 이용함.
  * 그렇지만, 조회 로직이 압도적으로 많아지는 경우 DB의 과부하로 이어질 수 있음.
  * 아울러, Command와 Query 역할을 하나로 두게 되면 특정 로직 변경 시 책임의 크기가 과하게 커질 수 있기 때문에 SOLID 원칙 중 하나인 단일 책임의 원칙을 위배할 수 있음.
* CQRS를 통해 Command와 Query를 분리하여 기능별 성능 향상과 보안성/확장성 향상을 기대할 수 있음.

<br>

### SSOT(Single Source Of Truth)
* 임의의 데이터에 대해 하나의 출처만 사용하는 방법론을 의미함.
* 데이터의 일관성을 보장해줄 수 있으며, 데이터 관리에 대한 복잡성을 줄일 수 있음.

<br>

### Materialized View
* DBMS에 Query 성능을 향상시킬 목적으로 구성해놓은 물리적인 뷰(View)를 의미함.
* 관련 데이터가 디스크에 물리적으로 저장되며, Indexing도 수행 가능함.
* 별도의 DB를 따로 구성히지 않고, 동일한 DB에 CQRS를 구성하려 할 때 유용하게 사용할 수 있음.
* MySQL과 MariaDB에서는 지원하지 않으며, PostgreSQL과 Oracle DB에서 지원함.

<br>

### OLTP(Online Transaction Processing)
* 온라인 상에서 일련의 트랜잭션 처리를 수행할 때 사용하는 기술.
* 다수의 사용자가 붙어서 삽입/수정/삭제 등의 Command 작업을 많이 수행하는 시스템에 적합함.

<br>

### OLAP(Online Analytical Processing)
* 온라인 상에서 비즈니스 데이터를 다각도로 분석할 때 사용하는 기술.
* 분석을 거친 특정 데이터를 얻기 위해 복잡한 Query를 많이 수행하는 시스템에 적합함.

<br>

### Event Sourcing
* 프로그램에서 발생되는 이벤트의 처리 과정을 하나하나 연속적으로 저장하는 기법.
* 이벤트가 발생한 일련의 기록들을 하나하나 저장하다보면 용량 이슈가 발생할 수 있으므로, 이를 스냅샷(Snapshot) 형태로 중간중간 나눠서 저장함. 
* 특정 작업내역에 대한 조회 시, 성능상의 이점을 얻기 위해 CQRS와 함께 사용함.

#### 참고
* https://martinfowler.com/bliki/CommandQuerySeparation.html
* https://learn.microsoft.com/ko-kr/azure/architecture/data-guide/relational-data/online-analytical-processing

<br>

#### 배워가는 것들
* CQS, CQRS에 대한 핵심개념 파악 및 활용도를 조사함.
* Command 유형과 Query 유형에 따른 분리설계 및 구현에 대한 여러 개념을 파악할 수 있었음.
* 서비스 아키텍처 구축 및 그에 따른 Module 개발 시 반드시 알아야하는 개념을 파악할 수 있었음.
