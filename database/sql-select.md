# SQL 조회와 정렬

### SELECT
- 테이블에 저장된 데이터를 꺼내보는 조회 명령어로, SQL에서 가장 많이 사용함
  - SELECT: 무엇을 가져올 것인가(조회할 열)
  - FROM: 어디에서 가져올 것인가(테이블)
- *(애스터리스크)는 모든 열을 의미함
```
SELECT * FROM customers;
SELECT name, email FROM customers;
```
- 실무에서는 SELECT * 사용을 최소화하고, 필요한 열만 명시해서 조회해야 함
  - 성능 저하: 필요 없는 열까지 모두 읽어오느라 DB에 부담을 줌
  - 가독성 저하: 수십 개의 열이 함께 나오면 원하는 정보를 한눈에 파악하기 어려움
  - 네트워크 낭비: DB 서버에서 클라이언트로 필요 없는 데이터까지 전송함

<br>

### 별칭(Alias)
- AS 키워드로 조회 결과의 열 이름을 바꿀 수 있음
```
SELECT name AS 고객명, email AS 이메일 FROM customers;
```
- 별칭을 사용하는 이유
  - 보고서처럼 다른 사람이 보는 결과의 가독성을 높임
  - 여러 테이블을 조인할 때 같은 이름의 열을 구분함
- AS는 생략할 수 있지만, 쿼리를 명확하게 하기 위해 써주는 것이 좋음
- 별칭도 테이블, 컬럼 이름 규칙을 따르며, 공백 같은 특수문자가 필요하면 백틱(`)으로 감쌈

<br>

### WHERE
- FROM 절 뒤에 위치하며, 조건을 만족하는 행만 걸러내는 필터 역할을 함
```
SELECT * FROM customers WHERE email = 'yisunsin@example.com';
SELECT * FROM products WHERE price >= 10000;
```
- 비교 연산자

| 연산자 | 의미 |
| --- | --- |
| = | 같다 |
| !=, <> | 같지 않다 |
| >, < | 크다, 작다 |
| >=, <= | 크거나 같다, 작거나 같다 |

- 논리 연산자
  - AND: 양쪽 조건이 모두 참일 때 참
  - OR: 양쪽 조건 중 하나라도 참이면 참
  - NOT: 조건을 부정함. IN, LIKE, BETWEEN, IS NULL 등과 함께 사용함
```
SELECT * FROM products WHERE price >= 5000 AND stock_quantity >= 50;
SELECT * FROM products WHERE price = 20000 OR stock_quantity >= 100;
```

<br>

### 편리한 조건 검색
- BETWEEN a AND b: a 이상 b 이하의 범위에 있는 값을 찾음(양 끝값 포함)
  - price >= 5000 AND price <= 15000을 더 직관적으로 표현할 수 있음
  - NOT BETWEEN은 범위를 벗어난 값을 찾음
- IN (목록): 목록에 있는 값 중 하나와 일치하는 값을 찾음
  - OR를 여러 번 반복하는 것보다 간결함
  - NOT IN은 목록에 없는 값을 찾음
- LIKE: 문자열의 일부로 검색함(패턴 매칭)
  - %: 0개 이상의 모든 문자. 'sejong%'는 sejong으로 시작, '%@example.com'은 @example.com으로 끝, '%서울%'은 서울을 포함하는 문자열
  - _: 정확히 한 개의 문자. '이_신'은 이순신은 찾지만 이나라신은 찾지 않음
  - NOT LIKE는 패턴에 맞지 않는 값을 찾음
```
SELECT * FROM products WHERE price BETWEEN 5000 AND 15000;
SELECT * FROM products WHERE name IN ('갤럭시', '아이폰', '에어팟');
SELECT * FROM customers WHERE email LIKE 'sejong%';
SELECT * FROM customers WHERE address NOT LIKE '서울특별시%';
```

<br>

### ORDER BY
- 조회 결과를 특정 열의 값을 기준으로 정렬함
- ORDER BY가 없으면 결과의 순서는 보장되지 않음
  - 어떨 때는 PK 순서로, 어떨 때는 삽입된 순서로 보이는 등 일관성이 없으므로, 순서가 필요하면 반드시 명시해야 함
- ASC(Ascending): 오름차순. 기본값이라 생략할 수 있으며, 실무에서도 대부분 생략함
- DESC(Descending): 내림차순. 반드시 명시해야 함
- 여러 열을 쉼표로 나열하면 앞에 쓴 열부터 우선 적용되며, 앞 기준의 값이 같을 때만 다음 기준으로 정렬함
  - 각 열마다 ASC, DESC를 따로 지정할 수 있음
```
SELECT * FROM customers ORDER BY join_date DESC;
SELECT * FROM products ORDER BY stock_quantity DESC, price;
```

<br>

### LIMIT
- 조회할 결과의 최대 개수를 제한하며, ORDER BY 뒤에 위치함
- 결과의 순서는 보장되지 않으므로, 의미 있는 상위 N개를 뽑으려면 반드시 ORDER BY로 먼저 정렬한 뒤 LIMIT으로 잘라내야 함
```
SELECT * FROM products ORDER BY price DESC LIMIT 2;
```
- 페이징(Pagination)
  - LIMIT offset, row_count: 앞에서부터 offset개를 건너뛰고 row_count개를 가져옴
  - offset = (페이지 번호 - 1) * 페이지당 개수
  - 사용자에게 일관된 순서를 보여주기 위해 PK 등으로 정렬하는 것을 잊지 말아야 함
  - 마지막 페이지에서는 row_count보다 적은 행이 반환될 수 있음
```
-- 한 페이지에 2개씩, 2페이지 조회
SELECT * FROM products ORDER BY product_id LIMIT 2, 2;
SELECT * FROM products ORDER BY product_id LIMIT 2 OFFSET 2; -- 위와 같음
```
- LIMIT은 MySQL, PostgreSQL 등에서 지원하는 문법이며, SQL 표준 문법은 OFFSET n ROWS FETCH FIRST n ROWS ONLY임
- 뒤쪽 페이지로 갈수록 OFFSET만큼 읽고 버리는 양이 늘어나므로, 데이터가 많아지면 Keyset Pagination 같은 방법을 고려해야 함

<br>

### DISTINCT
- 조회 결과에서 중복된 행을 제거하고 고유한 값만 남김
```
SELECT DISTINCT customer_id FROM orders;
SELECT DISTINCT customer_id, product_id FROM orders;
```
- 여러 열에 적용하면 지정한 모든 열의 조합이 유일한 행만 남김
- "주문한 고객 목록", "판매된 상품의 카테고리 종류"처럼 데이터의 종류를 파악할 때 유용함
- 내부적으로 중복을 제거하는 과정이 필요하므로, 데이터가 아주 많다면 성능을 확인해야 함

<br>

### NULL
- 값이 존재하지 않거나 알 수 없음을 나타내는 특별한 상태
  - 숫자 0이나 빈 문자열('')과는 완전히 다른 개념으로, 0과 ''는 값이 있는 것이고 NULL은 값이 있는지조차 모르는 것임
- NULL은 =로 비교할 수 없음
  - 어떤 값 = NULL은 항상 알 수 없음(UNKNOWN)이 되며, NULL = NULL조차 참이 아님
  - WHERE 절은 조건이 참(TRUE)인 행만 반환하므로, WHERE description = NULL은 아무것도 조회하지 못함
- NULL인지 검사할 때는 반드시 IS NULL, IS NOT NULL을 사용해야 함
```
SELECT * FROM products WHERE description IS NULL;
SELECT * FROM products WHERE description IS NOT NULL;
```

<br>

### NULL 정렬
- MySQL은 NULL을 가장 작은 값으로 취급함
  - ASC: NULL이 가장 먼저 나옴
  - DESC: NULL이 가장 나중에 나옴
- DBMS마다 정책이 다르므로 사용하는 DB의 규칙을 알아야 함
  - Oracle, PostgreSQL은 NULL을 가장 큰 값으로 취급하며, NULLS FIRST, NULLS LAST로 위치를 지정할 수 있음
- MySQL에서 NULL의 위치를 바꾸고 싶다면 IS NULL을 정렬 기준으로 사용함
  - MySQL은 TRUE를 1, FALSE를 0으로 표현하므로, 컬럼 IS NULL은 NULL이면 1, 아니면 0이 됨
  - 이를 1차 정렬 기준으로, 원래 컬럼을 2차 정렬 기준으로 두면 NULL의 위치를 원하는 대로 제어할 수 있음
```
-- 설명을 내림차순으로 정렬하되, 설명이 없는 상품은 맨 앞으로
SELECT * FROM products
ORDER BY description IS NULL DESC, description DESC;
```

<br>

#### 참고
- 인프런 <김영한의 실전 데이터베이스 입문 - SQL부터 차근차근> - 김영한
- MySQL Reference Manual <SELECT Statement> - https://dev.mysql.com/doc/refman/8.4/en/select.html
- MySQL Reference Manual <Working with NULL Values> - https://dev.mysql.com/doc/refman/8.4/en/working-with-null.html

#### 배워가는 것들
- SELECT *는 편리하지만, 필요한 열만 골라서 조회하는 것이 성능과 가독성 모두에 좋다는 점을 다시 확인할 수 있었다.
- 데이터베이스는 결과의 순서를 보장하지 않는다. 순서가 중요한 조회와 LIMIT에는 항상 ORDER BY를 함께 써야겠다.
- NULL은 값이 아니라 상태라는 점을 이해하니, = NULL이 동작하지 않는 이유와 IS NULL을 써야 하는 이유가 자연스럽게 정리되었다.
