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

* 참고
  * https://terms.tta.or.kr/dictionary/dictionaryView.do?subject=%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4
