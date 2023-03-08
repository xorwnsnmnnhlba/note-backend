# Database

* 데이터베이스(DB, Database)
	* 다수의 사용자에 의해 쓰여지는 데이터의 집합을 의미함.
	* 서로 연관성있는 데이터들의 집합이기 때문에, 체계를 가지고 관리가 이루어짐.
	* 데이터베이스 주요 특징
		* 실시간 접근성(real-time accessibility): 사용자의 질의에 즉각적인 처리와 응답이 가능함.
		* 계속적인 변화(continuous evolution): 상황에 맞춰 데이터의 추가/삭제/수정을 통해 항상 최신의 데이터를 유지함.
		* 동시 공유(concurrent sharing): 여러 애플리케이션에서 사용하기 때문에, 다수의 사용자가 동시에 같은 내용의 데이터를 공유할 수 있음.
		* 내용에 의한 참조(content reference): 데이터 참조 시 데이터 레코드의 주소나 위치로 찾지 않고, 데이터 내용으로 데이터를 찾음.

<br>

* DBMS(Database Management System)
	* 데이터베이스 관리 주체.
	* 프로그램을 이용하는 사용자는 프로그램이 참조하는 DB에 저장된 데이터를 쉽게 찾고 효율적으로 추가/삭제/수정할 수 있어야 함.
	* 일상에서 쓰이는 데이터들이 모인 DB 자체로는 할 수 있는게 없기 때문에, 이를 관리하는 주체가 필요함.
	* 시스템 구축 시, 용도에 맞는 DBMS를 적재적소에 맞춰 구성해야 함.
	* DBMS 필수 기능
		* 정의(Definition)
			* 다양한 형태의 데이터 요구를 지원할 수 있도록 가장 적절한 데이터베이스 구조를 정의할 수 있어야 함.
		* 조작(Manipulation)
			* 사용자와 데이터베이스 사이의 인터페이스를 위한 수단을 제공해야 함.
			* 사용자의 요구에 따라 데이터베이스를 체계적으로 접근하고 조작 가능해야 함.
		* 제어(Control)
			* 적절한 권한 조정을 통해 데이터베이스에 있는 데이터의 정확성과 보안성을 유지해야 함.

<br>

* RDBMS(Relational Database Management System)
	* 관계형 데이터베이스를 관리하기 위한 시스템.
	* 데이터가 2차원 테이블(Table) 형태로 저장됨.
	* MySQL, MariaDB, PostgreSQL, MSSQL Server, Oracle DB 등등이 있음.

<br>

* 데이터베이스 언어
	* 데이터 정의 언어(DDL, Data Definition Language): 데이터베이스 명세라 할 수 있는 스키마를 정의할 때 사용하는 언어.
		* CREATE: 데이터베이스, 테이블 생성
		* ALTER: 테이블 수정
		* DROP: 데이터베이스, 테이블 삭제
		* TRUNCATE: 테이블 초기화
	* 데이터 조작 언어(DML, Data Manipulation Language): 데이터베이스에 데이터를 추가하거나 저장된 데이터를 조회/수정/삭제할 때 사용하는 언어.
		* SELECT: 데이터 조회
		* INSERT: 데이터 삽입
		* UPDATE: 데이터 수정
		* DELETE: 데이터 삭제
	* 데이터 제어 언어(DCL): 데이터베이스 접근 권한을 조정할 때 사용하는 언어.
		* GRANT: 특정 작업에 대한 수행 권한 부여
		* REVOKE: 특정 작업에 대한 수행 권한 회수
		* COMMIT: 트랜잭션 작업 저장
		* ROLLBACK: 트랜잭션 작업 취소 후 원상복구

<br>

* SQL(Structured Query Language)
	* DBMS에 저장된 데이터를 관리하기 위한 질의언어.

<br>

* 데이터 모델(Data Model)
	* 현실 세계에 있는 데이터를 데이터베이스 내부의 데이터로 옮기는 과정. 관점에 따라 아래와 같이 세 가지로 구분함.
		* 개념적(Conceptual) 데이터 모델
		* 논리적(Logical) 데이터 모델
		* 물리적(Physical) 데이터 모델
	* 관계형 데이터 모델
		* 데이터를 2차원 테이블(Table) 형태로 구성하는 작업.
		* 속성(Attribute): 이름과 타입으로 구성된 값. 테이블 내에서 유일한 값을 가짐.
		* 튜플(Tuple): 속성과 값으로 이루어진 쌍의 집합. 하나의 Row 혹은 Record로 간주함.
		* 관계(Relation): 튜플의 집합으로써, 하나의 Table로 간주함.

<br>

* 관계대수(Relational Algebra)
	* 기존의 릴레이션으로부터 특정 연산을 거쳐 새로운 릴레이션을 나타내는 과정.
	* 하나의 릴레이션을 가지고 수행하는 단항연산과 두 개 이상의 릴레이션을 가지고 수행하는 이항연산이 있음.
		* 단항연산: Selection, Projection, Rename 등등
		* 이항연산: Cartesian Product, Set Union, Set Difference 등등
	* 자주 사용하는 연산
		* Selection
			* 임의의 릴레이션이 가지고 있는 속성에 해당하는 값만 가지고 새로운 튜플을 구성함. SQL에서 SELECT절을 이용하여 표현가능.
		* Projection
			* 특정 조건을 만족하는 데이터를 바탕으로 새로운 튜플을 구성함. SQL에서 WHERE절을 이용하여 표현가능.
		* Cartesian Product
			* 두 릴레이션에 해당하는 모든 튜플들을 조건없이 합쳐서 새로운 튜플로 구성함. SQL에서 FROM절 뒤에 테이블 이름을 그대로 나열하면 됨.
		* Join
			* 두 릴레이션에 대해 특정 조건을 만족하는 데이터를 가지고 새로운 튜플을 구성함. SQL에서 JOIN 키워드를 이용하여 표현가능.
			* 실무에서는 Cartesian Product을 직접 사용하는 경우는 거의 없으며, 주로 아래와 같이 Left Outer Join을 많이 사용함.

```
SELECT a1, a2
FROM r1, r2
WHERE a1=a

SELECT r1.a1 AS r1_a1, r2.a3
FROM r1
LEFT OUTER JOIN r2 ON r1.a1 = r2.a2;
```

<br>

* 개체-관계 모델(Entity-Relationship Model)
	* 데이터들을 여러 개의 개체(Entity)들로 분류한 후, 각각의 개체에 따른 관계(Relationship)를 구성하여 기술한 모델.
	* 가장 인기있는 개념적 데이터 모델이라 할 수 있음.
	* 아래와 같이 하나의 Diagram(ERD, ER Diagram)으로 표현할 수 있음. 말이 되는 표현이 되게끔 설계가 이루어져야 함.
		* 개체는 사각형으로 표현하며, 개체에 있는 속성(Attribute)들은 원으로 표현함. 그리고 개체를 이어주는 관계는 마름모로 표현함.

<figure><img src="./images/ER_Diagram_MMORPG.png" alt=""></figure>

<br>

* 데이터베이스 정규화(Normalization)
	* 데이터베이스 설계 시, 중복을 최소화하여 무결성을 보장하기 위해 수행되는 일련의 과정.
	* 제1정규형(1NF), 제2정규형(2NF), 제3정규형(3NF), 보이스/코드 정규형(BCNF) 등등이 있음.
		* 제4정규형(4NF), 제5정규형(5NF)도 있으나, 실무에서는 주로 3NF, BCNF까지만 수행함.
	* 제1정규형(1NF): 모든 릴레이션에 있는 속성값들이 하나의 원자값으로 이루어져야 함.
	* 제2정규형(2NF): 1NF에서 모든 속성값들이 기본키(PK)에 대해 완전 함수 종속을 만족해야 함. 즉, 기본키가 아닌 속성이 기본키에 부분적으로 종속되지 않아야 함.
	* 제3정규형(3NF): 2NF에서 기본키가 아닌 속성이 다른 속성들을 결정하는 이행적 함수 종속이 없어야 함.
	* BCNF: 임의의 릴레이션에서 결정자이면서 후보키가 아닌 컬럼이 존재하지 않아야 함.

<br>

* 참고
  * https://terms.tta.or.kr/dictionary/dictionaryView.do?subject=%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4
	* https://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model
