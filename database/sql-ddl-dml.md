# SQL 데이터 정의와 조작(DDL, DML)

### 데이터베이스 생성과 선택
- CREATE DATABASE: 데이터를 담을 독립된 저장 공간(데이터베이스)을 만듦
- USE: 앞으로 실행할 SQL이 어느 데이터베이스를 대상으로 하는지 선택함
  - 선택하지 않고 테이블을 다루면 Error 1046(No database selected)이 발생함
  - MySQL Workbench를 다시 시작하면 USE를 다시 실행해야 함
- SQL은 세미콜론(;) 단위로 하나의 문장으로 인식함
- 데이터베이스 이름은 my_shop, shopping_mall_db처럼 소문자와 밑줄을 조합하는 것이 일반적인 관례임
```
CREATE DATABASE my_shop;
USE my_shop;
```

<br>

### 테이블 생성과 확인
- CREATE TABLE: 테이블을 구성할 열과 각 열의 데이터 타입, 제약 조건을 정의하여 테이블을 만듦
  - 이 과정을 테이블을 설계(정의)한다고 표현함
  - 모든 테이블에는 반드시 기본 키(PRIMARY KEY)가 있어야 함
```
CREATE TABLE sample (
    product_id INT PRIMARY KEY,
    name VARCHAR(100),
    price INT,
    stock_quantity INT,
    release_date DATE
);
```
- DESCRIBE(DESC): 테이블의 열 이름, 타입, NULL 허용 여부, 키, 기본값 등 구조를 보여줌
- SHOW DATABASES: 서버에 있는 데이터베이스 목록을 보여줌
  - information_schema, mysql, performance_schema, sys는 DBMS 운영을 위해 내부에서 사용하는 기본 데이터베이스임
- SHOW TABLES: 현재 선택된 데이터베이스의 테이블 목록을 보여줌
- -- 뒤에 공백을 넣으면 주석이 되며, 실행되지 않음
```
DESC sample;
SHOW DATABASES;
SHOW TABLES;
```

<br>

### 대소문자 구분
- SELECT, FROM, CREATE 같은 SQL 키워드는 대소문자를 구분하지 않음
- 직접 지은 테이블명, 컬럼명은 DBMS 종류나 서버 설정(운영체제 등)에 따라 대소문자를 구분하기도 함
- 데이터 자체는 대소문자를 구분할 수 있음
  - name = 'kim'과 name = 'KIM'은 컬럼의 콜레이션(Collation)에 따라 다른 결과가 나올 수 있음
- 키워드는 대문자, 이름은 소문자로 쓰는 것이 일반적인 관례이며, 편의상 모두 소문자로 쓰는 경우도 많음
- 어떤 환경에서든 같은 결과를 얻으려면 데이터베이스, 테이블, 열 이름은 항상 소문자로 작성하는 습관을 들이는 것이 좋음

<br>

### ALTER TABLE
- 이미 만든 테이블의 구조를 변경함
```
-- 열 추가
ALTER TABLE customers ADD COLUMN point INT NOT NULL DEFAULT 0;

-- 열 타입 변경
ALTER TABLE customers MODIFY COLUMN address VARCHAR(500) NOT NULL;

-- 열 삭제
ALTER TABLE customers DROP COLUMN point;
```
- 데이터가 수백만 건 이상인 테이블의 구조 변경은 많은 시간과 자원을 소모하며, 작업 중에 테이블이 잠겨 서비스가 멈출 수도 있음
  - 따라서 사용자가 적은 새벽 점검 시간에 작업하는 것이 일반적임
  - 최신 MySQL은 열 추가 정도는 온라인으로 적용할 수 있지만, 라이브 환경에서 실행하기 전에는 사용하는 버전의 매뉴얼에서 잠금 범위를 반드시 확인해야 함

<br>

### DROP, TRUNCATE
- DROP TABLE: 테이블의 데이터와 구조(설계도)를 모두 삭제함. 다시 사용하려면 CREATE TABLE부터 해야 함
- DROP DATABASE: 데이터베이스 안의 모든 테이블과 데이터를 함께 삭제함
  - 운영 중인 서비스에서는 절대 함부로 사용하면 안 되며, 삭제 전에 백업 여부와 대상을 반드시 여러 번 확인해야 함
- TRUNCATE TABLE: 테이블의 구조는 남기고 내부 데이터만 모두 삭제함
  - 한 줄씩 지우는 DELETE와 달리 테이블을 초기화하는 개념이라 훨씬 빠름
  - AUTO_INCREMENT 값도 초기화되어 다시 1부터 시작함
- 다른 테이블이 외래 키로 참조하고 있는 테이블은 DROP, TRUNCATE를 할 수 없음
  - DROP 시 Error 3730, TRUNCATE 시 Error 1701이 발생함
  - 이를 허용하면 상품 정보를 찾을 수 없는 주문 같은 데이터가 생기기 때문임
- 외래 키 검사를 잠시 끄고 작업할 수도 있음
  - 무결성이 깨질 수 있으므로 작업이 끝나면 반드시 다시 켜야 함
  - SET으로 바꾼 설정은 현재 접속(세션) 동안만 유효함
```
SET FOREIGN_KEY_CHECKS = 0; -- 비활성화
TRUNCATE TABLE products;
SET FOREIGN_KEY_CHECKS = 1; -- 활성화
```

<br>

### CRUD
- 데이터 관리의 가장 핵심적인 네 가지 작업을 묶어서 CRUD라고 부름
  - Create(INSERT), Read(SELECT), Update(UPDATE), Delete(DELETE)
- 조회(SELECT)는 SQL 조회와 정렬 노트에서 자세히 다룸

<br>

### INSERT
- 테이블에 새로운 행을 추가함
```
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```
- 열 목록과 값 목록은 순서와 개수가 정확히 일치해야 함
- 숫자는 그대로 쓰고, 문자열과 날짜는 작은따옴표(')로 감쌈
- 등록 방법
  - 모든 열에 값 넣기: 열 목록을 생략하고 테이블의 열 순서대로 모든 값을 넣음. AUTO_INCREMENT 열에는 NULL을 넣으면 다음 번호가 자동으로 채워짐
  - 원하는 열만 골라서 넣기: AUTO_INCREMENT, DEFAULT가 있는 열이나 NULL을 허용하는 열은 생략할 수 있음. 실무에서 가장 흔하게 사용하는 방식임
  - 한 번에 여러 행 넣기: VALUES 뒤에 여러 행을 쉼표로 구분하여 나열함
```
INSERT INTO customers (name, email, password, address)
VALUES ('세종대왕', 'sejong@example.com', 'hashed_password_456', '서울시 종로구');

INSERT INTO products (name, price, stock_quantity) VALUES
    ('검정 양말', 5000, 100),
    ('갈색 양말', 5000, 150),
    ('흰색 양말', 5000, 200);
```
- 열 목록은 항상 명시하는 것이 좋음
  - 열 목록을 생략한 INSERT는 나중에 ALTER TABLE로 열이 추가되면 모두 오류가 발생함
- 실무에서 비밀번호는 반드시 암호화(해시)해서 저장해야 함

<br>

### UPDATE
- 이미 존재하는 행의 값을 수정함
```
UPDATE products
SET price = 9800, stock_quantity = 580
WHERE product_id = 1;
```
- SET: 수정할 열과 새로운 값을 지정하며, 쉼표로 여러 열을 한 번에 수정할 수 있음
- WHERE: 수정할 행을 선택함. 생략하면 테이블의 모든 행이 수정되므로 극도로 주의해야 함
- 실행 결과의 Rows matched는 조건에 맞는 행 수, Changed는 실제로 값이 바뀐 행 수를 의미함

<br>

### 안전 업데이트 모드(Safe Update Mode)
- WHERE 절에 키 컬럼을 사용하지 않고(또는 LIMIT 없이) UPDATE, DELETE를 실행하면 오류(Error 1175)를 내서 막아주는 기능
  - WHERE를 빠뜨려서 모든 상품의 가격이 990원이 되는 것 같은 사고를 예방함
- 기본 설정
  - MySQL 서버(전역)와 애플리케이션 커넥션: 비활성화(0)
  - MySQL Workbench: 활성화(1)
- SELECT @@SQL_SAFE_UPDATES로 현재 상태를 확인할 수 있음
- 대량 수정이 꼭 필요하면 잠시 끄고 실행한 뒤 바로 다시 켜는 것이 안전함
```
SET SQL_SAFE_UPDATES = 0; -- 안전 모드 끄기
UPDATE products SET price = 19800 WHERE name = '베이직 반팔 티셔츠';
SET SQL_SAFE_UPDATES = 1; -- 안전 모드 켜기
```
- UPDATE, DELETE 전에는 반드시 같은 WHERE 절로 SELECT를 먼저 실행해서, 변경할 대상이 맞는지 눈으로 확인하는 습관을 들여야 함

<br>

### DELETE
- 테이블에서 행을 삭제함
```
DELETE FROM products
WHERE product_id = 1;
```
- DELETE는 행 자체를 없애므로 UPDATE보다 더 파괴적이며, WHERE를 빠뜨리면 테이블의 모든 데이터가 사라짐
- WHERE 없는 UPDATE, DELETE를 실행해버렸을 때의 대처
  1. 즉시 DBA와 상급자에게 보고하고, 가능하다면 서비스를 점검 상태로 전환하여 추가 피해를 막음
  2. 가장 최근의 백업을 확인하고, 백업 이후의 데이터 유실을 감수하고 복구함
  3. 백업이 없다면 데이터베이스 로그를 분석하여 변경 전 값을 추적함. 매우 어렵고 오래 걸림

<br>

### DELETE와 TRUNCATE 비교

| 구분 | DELETE FROM table; | TRUNCATE TABLE table; |
| --- | --- | --- |
| 종류 | DML | DDL |
| 처리 방식 | 한 줄씩 삭제하며 WHERE로 조건 지정 가능 | 테이블 전체를 한 번에 삭제하며 WHERE 사용 불가 |
| 속도 | 각 행의 삭제를 기록하므로 느림 | 테이블을 초기화하는 개념이라 매우 빠름 |
| AUTO_INCREMENT | 유지됨 | 1부터 다시 시작하도록 초기화됨 |
| 되돌리기 | 트랜잭션 안에서 ROLLBACK 가능 | 즉시 적용되어 ROLLBACK 불가 |

- 탈퇴한 회원 한 명, 특정 조건의 주문처럼 선별적인 삭제가 필요한 일반적인 비즈니스 로직은 항상 DELETE를 사용함
- 테스트용으로 넣었던 데이터를 모두 지우고 처음부터 다시 시작할 때는 TRUNCATE가 효율적임

<br>

#### 참고
- 인프런 <김영한의 실전 데이터베이스 입문 - SQL부터 차근차근> - 김영한
- MySQL Reference Manual <ALTER TABLE Statement> - https://dev.mysql.com/doc/refman/8.4/en/alter-table.html
- MySQL Reference Manual <TRUNCATE TABLE Statement> - https://dev.mysql.com/doc/refman/8.4/en/truncate-table.html

#### 배워가는 것들
- DROP, TRUNCATE, DELETE가 모두 무언가를 지우지만, 지우는 대상과 되돌릴 수 있는지가 전혀 다르다는 것을 명확히 구분할 수 있었다.
- INSERT에 열 목록을 명시하는 작은 습관이 나중의 스키마 변경에서 오류를 막아준다는 점을 기억해야겠다.
- WHERE 없는 UPDATE, DELETE 한 번이 서비스 전체의 사고로 이어질 수 있다. 데이터를 바꾸기 전에는 반드시 같은 조건으로 SELECT부터 해보는 습관을 들여야겠다.
