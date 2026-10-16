# 데이터 타입과 제약 조건

### 데이터 타입(Data Type)
- 각 열에 어떤 종류의 데이터가 저장될지를 지정하는 것
- 타입을 세분화해서 고르는 이유는 효율성과 정확성 때문임
  - 0~150 사이인 나이를 수십억까지 표현하는 타입에 저장하면 디스크와 메모리를 낭비함
  - 데이터의 성격을 타입으로 명확히 해두면 잘못된 값이 들어오는 것을 줄일 수 있음
- 세부 항목은 외울 필요 없이 개념만 이해하고, 필요할 때 공식 문서를 찾아보면 충분함

<br>

### 숫자 타입
- 정확한 숫자 타입(Exact Value Numeric Types)

| 타입 | 저장 공간 | 설명 |
| --- | --- | --- |
| TINYINT | 1 Byte | -128 ~ 127 |
| SMALLINT | 2 Byte | -32,768 ~ 32,767 |
| MEDIUMINT | 3 Byte | -8,388,608 ~ 8,388,607 |
| INT, INTEGER | 4 Byte | 약 -21억 ~ 21억. 가장 일반적으로 사용하는 정수 타입 |
| BIGINT | 8 Byte | 약 -922경 ~ 922경 |
| DECIMAL(M, D), NUMERIC | 가변 | 고정 소수점 숫자. M은 전체 자릿수, D는 소수점 이하 자릿수 |
| BIT(M) | 가변 | 1~64개의 비트 값을 저장함 |

- UNSIGNED 속성을 붙이면 0부터 양수만 저장하며, 저장할 수 있는 양수 범위가 두 배로 늘어남
  - UNSIGNED INT는 0 ~ 4,294,967,295까지 저장할 수 있음
- 금액처럼 오차가 있으면 안 되는 값은 DECIMAL을 사용함
- 근사치 숫자 타입(Approximate Value Numeric Types)
  - FLOAT(4 Byte), DOUBLE(8 Byte): 부동 소수점 숫자로, 매우 크거나 작은 수를 표현할 수 있지만 약간의 오차가 생길 수 있음
  - 정밀도가 아주 중요하지 않은 과학 계산이나 공학 데이터에 주로 사용함

<br>

### 문자열 타입
- VARCHAR(n): 최대 n글자까지 저장하는 가변 길이 문자열
  - '홍길동'을 VARCHAR(10)에 저장하면 실제 3글자만큼의 공간만 차지함
  - 길이 정보를 저장하기 위해 1~2바이트가 추가로 필요함
- CHAR(n): 항상 n글자의 공간을 차지하는 고정 길이 문자열
  - 선언한 길이보다 짧은 값을 저장하면 나머지를 공백으로 채움
  - 최대 255자까지 저장할 수 있음
- TEXT: 상품 상세 설명, 게시글 본문처럼 매우 긴 텍스트를 저장함. 더 긴 MEDIUMTEXT, LONGTEXT도 있음

| 구분 | VARCHAR | CHAR |
| --- | --- | --- |
| 저장 공간 | 실제 데이터 길이만큼 + 길이 정보 | 항상 선언한 길이만큼 |
| 최대 길이 | 65,535 Byte(행 전체 크기 제한을 함께 받음) | 255자 |
| 공간 효율 | 데이터 길이가 다양할 때 유리함 | 모든 값의 길이가 같을 때 유리함 |
| 수정 시 | 길이가 바뀌면 행 크기가 변해 단편화가 생길 수 있음 | 행 크기가 변하지 않음 |

- 실무에서는 특별한 이유가 없다면 VARCHAR로 시작하는 것이 일반적임
  - 이름, 이메일, 주소처럼 대부분의 문자열은 길이가 제각각이라 VARCHAR가 공간을 크게 절약함
  - 국가 코드, 우편번호, MD5 해시값(32자리)처럼 모든 값이 예외 없이 같은 길이일 때만 CHAR를 고려함

<br>

### 날짜와 시간 타입

| 타입 | 형식 | 저장 공간 | 범위 | 사용 사례 |
| --- | --- | --- | --- | --- |
| DATE | YYYY-MM-DD | 3 Byte | 1000-01-01 ~ 9999-12-31 | 생년월일, 출시일 |
| TIME | HH:MM:SS | 3 Byte | -838:59:59 ~ 838:59:59 | 영업 시각, 재생 시간 |
| DATETIME | YYYY-MM-DD HH:MM:SS | 5 Byte | 1000년 ~ 9999년 | 예약 시간, 로그 시각 |
| TIMESTAMP | YYYY-MM-DD HH:MM:SS | 4 Byte | 1970-01-01 ~ 2038-01-19(UTC) | 생성일, 수정일 |
| YEAR | YYYY | 1 Byte | 1901 ~ 2155 | 졸업 연도 |

- DATETIME: 입력된 값을 그대로 저장하며, 서버의 타임존 설정이 바뀌어도 저장된 값은 변하지 않음
- TIMESTAMP: 저장할 때 서버 타임존 기준으로 UTC로 변환해서 저장하고, 조회할 때 현재 세션의 타임존으로 다시 변환해서 보여줌
- 최근에는 TIMESTAMP보다 DATETIME 사용을 권장함
  - TIMESTAMP는 2038년까지만 저장할 수 있음
  - 행 생성/수정 시각을 자동으로 기록하는 기능(DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)을 MySQL 5.6.5부터 DATETIME도 지원함
  - 저장 공간 차이는 현대 하드웨어에서 미미함
  - 글로벌 서비스라면 애플리케이션에서 UTC 기준으로 바꿔서 DATETIME에 저장하면 됨

<br>

### 기타 타입
- BLOB: 이미지, 오디오, 비디오 같은 이진(Binary) 대용량 데이터를 저장함
- ENUM: 미리 정의한 값 목록 중 하나만 선택하여 저장함
- SET: 미리 정의한 값 목록 중 여러 개를 동시에 선택하여 저장함
- 실무에서 ENUM, SET은 잘 사용하지 않음

<br>

### 제약 조건(Constraints)
- 테이블에 데이터를 저장할 때 특정 규칙을 지키도록 강제하는 장치
- 잘못되거나 일관성 없는 데이터가 들어오는 것을 원천적으로 막아, 데이터의 무결성(결점이 없는 상태)을 유지하는 것이 목표임
- 종류
  - NOT NULL: NULL을 허용하지 않는 필수 입력 항목. 이름, 아이디, 가격 등에 설정함
  - UNIQUE: 테이블 안에서 중복을 허용하지 않음. 이메일, 사업자 등록번호 등에 설정하며, 여러 열에 설정할 수 있음
  - PRIMARY KEY: 각 행을 유일하게 식별하는 대표 식별자. NOT NULL과 UNIQUE의 특징을 모두 가지며, 테이블당 하나만 존재함
    - AUTO_INCREMENT: MySQL에서 정수형 PK에 자주 사용하는 옵션으로, 데이터가 추가될 때마다 1씩 증가하는 번호를 자동으로 부여함
  - FOREIGN KEY: 다른 테이블의 PK를 참조하여 테이블 간의 관계를 맺고, 참조 무결성을 강제함
  - DEFAULT: 값을 지정하지 않았을 때 자동으로 입력될 기본값. 포인트 0점, 판매 상태 '판매중', 등록 시각 등에 사용함
  - CHECK: 입력되는 값이 특정 조건을 만족하는지 검사함. 음수 가격이나 18세 미만 회원의 저장을 막는 식으로 사용함

<br>

### 제약 조건을 적용한 테이블 설계
- 고객, 상품, 주문 테이블에 제약 조건을 적용하면 다음과 같음
```
CREATE TABLE customers (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    address VARCHAR(255) NOT NULL,
    join_date DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price INT NOT NULL,
    stock_quantity INT NOT NULL DEFAULT 0
);

CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    order_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) NOT NULL DEFAULT '주문접수',
    CONSTRAINT fk_orders_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    CONSTRAINT fk_orders_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```
- 날짜 컬럼의 자동 기록
  - DEFAULT CURRENT_TIMESTAMP: 행을 추가할 때 값을 지정하지 않으면 현재 시각이 들어감
  - DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP: 행을 수정할 때마다 현재 시각으로 자동 갱신됨
- 외래 키 제약 조건의 형식
```
CONSTRAINT [제약조건_이름]
    FOREIGN KEY ([자식_테이블의_컬럼명])
    REFERENCES [부모_테이블명]([부모_테이블의_컬럼명])
    [ON DELETE 옵션] [ON UPDATE 옵션]
```
- 제약 조건 이름에 fk_자식테이블_부모테이블처럼 참조 방향을 담아두면 이름만 보고도 관계를 파악할 수 있음

<br>

### 제약 조건 위반
- 규칙을 어기는 데이터가 들어오면 DBMS는 입력을 거부하고 오류를 발생시킴
  - NOT NULL 위반: 필수 컬럼을 빼고 INSERT하면 Error 1364(Field 'name' doesn't have a default value)가 발생함
  - UNIQUE 위반: 이미 사용 중인 이메일로 INSERT하면 Error 1062(Duplicate entry)가 발생함
  - FK 위반: 부모 테이블에 없는 customer_id로 주문을 INSERT하면 Error 1452(a foreign key constraint fails)가 발생함
- 제약 조건은 INSERT뿐 아니라 UPDATE, DELETE에서도 먼저 검증되므로, 어떤 경로로 데이터가 바뀌든 무결성이 보호됨

<br>

### ERD(Entity Relationship Diagram)
- 테이블과 테이블 사이의 관계를 그림으로 표현한 데이터베이스의 지도
- 테이블이 수십, 수백 개로 늘어나면 ERD 없이 구조를 파악하기 어려우며, 기획자 등 다른 팀원과 소통할 때도 활용됨
- MySQL Workbench에서는 Database > Reverse Engineer로 기존 데이터베이스의 ERD를 만들 수 있음
- 위 쇼핑몰 테이블의 관계
  - 한 명의 고객은 여러 번 주문할 수 있음: customers(1) ---< orders(N)
  - 하나의 상품은 여러 번 주문될 수 있음: products(1) ---< orders(N)
  - ---< 는 까마귀발 표기법(Crow's Foot Notation)을 간단히 표현한 것으로, < 쪽이 다수(Many)를 의미함
- 테이블 간의 관계에는 1:1, 1:N, N:1, N:M 등이 있음

<br>

### 테이블과 컬럼 이름 규칙
- 백틱 없이 사용하는 이름(권장)
  - 영문 대소문자, 숫자, 밑줄(_), 달러 기호($), 한글 같은 다국어 문자를 사용할 수 있음
  - 숫자로 시작할 수 없고, SELECT나 ORDER 같은 예약어는 사용할 수 없으며, 최대 64자까지 가능함
- 백틱(`)으로 감싼 이름
  - 공백, 하이픈, 특수문자, 숫자로 시작하는 이름, 예약어까지 거의 모든 문자를 사용할 수 있음
- 실무에서 권장하는 명명 규칙
  - user_orders, product_name처럼 영문 소문자와 밑줄을 조합한 스네이크 케이스(snake_case)를 사용함
  - 프로젝트 안에서 하나의 규칙을 정해 일관되게 적용함
  - 백틱을 쓰면 예약어도 쓸 수 있지만, 혼란을 피하기 위해 사용하지 않음
  - prdct_nm처럼 줄이기보다 product_name처럼 의미가 드러나게 지음
  - 한글 이름은 호환성과 인코딩 문제를 피하기 위해 사용하지 않음

<br>

#### 참고
- 인프런 <김영한의 실전 데이터베이스 입문 - SQL부터 차근차근> - 김영한
- MySQL Reference Manual <Data Types> - https://dev.mysql.com/doc/refman/8.4/en/data-types.html
- MySQL Reference Manual <FOREIGN KEY Constraints> - https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html

#### 배워가는 것들
- 데이터 타입은 외우는 것이 아니라, 데이터의 예상 범위와 성격에 맞춰 고르는 것이라는 점을 알게 되었다.
- VARCHAR와 CHAR, DATETIME과 TIMESTAMP의 차이를 정리하면서, 과거의 원칙보다 현재의 실무 기준이 왜 달라졌는지 이해할 수 있었다.
- 제약 조건은 애플리케이션 코드의 검증과 별개로 데이터베이스가 직접 지켜주는 마지막 방어선이다. 테이블을 설계할 때부터 꼼꼼하게 걸어두어야겠다.
