# SQL 집계와 그룹핑

### 집계 함수(Aggregate Function)
- 여러 행의 데이터를 바탕으로 하나의 요약된 결과 값을 계산하는 함수
- 총 주문 건수, 총매출, 최고가 상품처럼 데이터 전체의 규모와 경향을 파악할 때 사용함
- 아래 예제는 고객명, 카테고리, 상품명, 가격, 수량, 주문일을 한 테이블에 담은 order_stat(주문 통계) 테이블을 사용함

| 함수 | 설명 |
| --- | --- |
| COUNT(*) | NULL과 상관없이 전체 행의 수를 반환함 |
| COUNT(expression) | 값이 NULL이 아닌 행의 수를 반환함 |
| SUM(expression) | 합계를 반환함 |
| AVG(expression) | 평균값을 반환함 |
| MAX(expression) | 최댓값을 반환함 |
| MIN(expression) | 최솟값을 반환함 |

```
SELECT
    COUNT(*) AS `전체 주문 건수`,
    COUNT(category) AS `카테고리 등록 건수`,
    SUM(price * quantity) AS `총 매출액`,
    AVG(price * quantity) AS `평균 주문 금액`,
    MAX(price) AS `최고가`,
    MIN(order_date) AS `최초 주문일`
FROM order_stat;
```

<br>

### 집계 함수와 NULL
- COUNT(*)와 COUNT(컬럼)
  - COUNT(*)의 *는 행 자체를 의미하므로, NULL 여부와 상관없이 물리적인 행의 수를 모두 셈. 출석부의 총인원을 세는 것과 같음
  - COUNT(컬럼)은 해당 컬럼의 값이 NULL이 아닌 행만 셈. 출석부에서 이름이 실제로 적힌 학생 수만 세는 것과 같음
  - 11건의 주문 중 1건의 category가 NULL이면, COUNT(*)는 11, COUNT(category)는 10이 됨
- SUM(), AVG()는 계산할 때 NULL을 자동으로 제외함
  - NULL을 0으로 취급하지 않으므로, AVG는 NULL이 아닌 값들만의 평균이 됨
- MAX(), MIN()은 숫자뿐 아니라 날짜에도 사용하여 데이터의 범위를 파악할 수 있음

<br>

### COUNT(DISTINCT)
- 중복을 제외한 고유한 값의 개수를 셈
- 한 고객이 여러 번 주문했다면 COUNT(customer_name)은 주문 건수가 되므로, 순수 고객 수를 구하려면 DISTINCT를 함께 사용해야 함
```
SELECT
    COUNT(customer_name) AS `총 주문 건수`,          -- 11
    COUNT(DISTINCT customer_name) AS `순수 고객 수` -- 5
FROM order_stat;
```

<br>

### GROUP BY
- 특정 컬럼의 값이 같은 행들을 하나의 그룹으로 묶음
- 그룹으로 묶은 뒤 집계 함수를 적용하면, 전체가 아닌 그룹별 통계를 낼 수 있음
  - 학교 전체 평균이 아니라 1학년, 2학년, 3학년의 평균을 각각 구하는 것과 같음
```
-- 카테고리별 주문 건수
SELECT category, COUNT(*) AS `카테고리별 주문 건수`
FROM order_stat
GROUP BY category;

-- 고객별 구매 활동(VIP 고객 찾기)
SELECT
    customer_name,
    COUNT(*) AS `총 주문 횟수`,
    SUM(quantity) AS `총 구매 수량`,
    SUM(price * quantity) AS `총 구매 금액`
FROM order_stat
GROUP BY customer_name
ORDER BY `총 구매 금액` DESC;
```
- GROUP BY는 NULL도 하나의 독립된 그룹으로 취급함
  - 실무에서는 NULL 그룹을 보고 데이터가 누락되었다는 것을 발견하기도 함
- 여러 컬럼을 쉼표로 나열하면, 앞의 컬럼으로 묶은 뒤 그 안에서 다시 뒤의 컬럼으로 묶어 더 세분화된 그룹을 만듦
```
-- 고객이 어떤 카테고리에서 얼마를 사용했는가
SELECT customer_name, category, SUM(price * quantity) AS `카테고리별 구매 금액`
FROM order_stat
GROUP BY customer_name, category
ORDER BY customer_name, `카테고리별 구매 금액` DESC;
```

<br>

### ORDER BY에서 별칭을 사용할 때의 주의점
- SELECT에서 만든 별칭을 ORDER BY에서 사용할 때, 공백 등이 있는 별칭은 반드시 백틱(`)으로 감싸야 함
- MySQL에서 작은따옴표(')나 큰따옴표(")로 감싸면 컬럼이 아니라 문자 상수로 인식함
  - ORDER BY '총 구매 금액' DESC는 모든 행을 같은 문자 값으로 정렬하는 것이 되어, 정렬 효과가 전혀 없음
  - 오류도 발생하지 않으므로 알아차리기 어려움

<br>

### GROUP BY 사용 시 주의사항
- GROUP BY를 사용하면 SELECT 절에는 GROUP BY에 사용한 컬럼과 집계 함수만 올 수 있음
- 그룹으로 묶으면 여러 행이 하나의 행으로 요약되므로, 그룹을 대표할 수 없는 개별 행의 컬럼은 하나의 값을 특정할 수 없기 때문임
  - '전자기기' 그룹의 category는 '전자기기', COUNT(*)는 5로 명확하지만, product_name은 5개 상품 중 무엇을 보여줘야 할지 알 수 없음
  - 3반의 학생 수는 알려줄 수 있지만, "3반 학생 이름 하나"는 누구를 말해야 할지 모르는 것과 같음
```
-- 잘못된 쿼리
SELECT category, product_name, COUNT(*)
FROM order_stat
GROUP BY category;
-- Error 1055: Expression #2 of SELECT list is not in GROUP BY clause ...
-- this is incompatible with sql_mode=only_full_group_by
```
- MySQL 5.7부터 ONLY_FULL_GROUP_BY 모드가 기본으로 켜져 있어 이러한 쿼리를 오류로 막아줌

<br>

### HAVING
- GROUP BY로 그룹화한 결과에 대해 조건을 걸어 그룹을 필터링함
- WHERE 절에서는 집계 함수를 사용할 수 없음
  - WHERE는 그룹화하기 전에 개별 행을 검사하므로, 아직 계산되지 않은 SUM() 같은 값을 조건으로 쓸 수 없음(Error 1111: Invalid use of group function)
- HAVING은 GROUP BY 바로 뒤에 위치하며, 조건에 집계 함수를 사용할 수 있음
```
-- 총 매출액이 50만 원 이상인 핵심 카테고리
SELECT category, SUM(price * quantity) AS total_sales
FROM order_stat
GROUP BY category
HAVING SUM(price * quantity) >= 500000;

-- 3회 이상 주문한 충성 고객
SELECT customer_name, COUNT(*) AS order_count
FROM order_stat
GROUP BY customer_name
HAVING COUNT(*) >= 3;
```
- HAVING에서 SELECT의 별칭 사용
  - SQL 표준상 HAVING은 SELECT보다 먼저 처리되므로 원칙적으로는 별칭을 사용할 수 없음
  - MySQL, PostgreSQL 등은 편의를 위해 허용하지만, SQL Server, Oracle 등은 오류가 발생함
  - 여러 DB에서 문제없이 동작하게 하려면 HAVING에는 집계 함수 표현식을 직접 쓰는 것이 안전함

<br>

### WHERE와 HAVING

| 구분 | WHERE | HAVING |
| --- | --- | --- |
| 작동 시점 | GROUP BY 이전 | GROUP BY 이후 |
| 필터링 대상 | 개별 행(Row) | 그룹(Group) |
| 집계 함수 | 사용 불가 | 사용 가능 |

- 운동장에 모인 학생들에 비유하면 다음과 같음
  1. "안경 쓴 학생만 앞으로 나와!" → WHERE(개별 행 필터링)
  2. "나온 학생들은 같은 반끼리 모여!" → GROUP BY(그룹화)
  3. "반 평균이 80점 이상인 반만 남아!" → HAVING(그룹 필터링)
- 개별 행에 대한 조건은 WHERE에, 그룹에 대한 조건은 HAVING에 작성하여 함께 사용할 수 있음
```
-- 10만 원 이상인 고가 상품 중에서, 2건 이상 팔린 카테고리
SELECT category, COUNT(*) AS premium_order_count
FROM order_stat
WHERE price >= 100000     -- 1. 개별 행을 가격으로 필터링
GROUP BY category         -- 2. 남은 행을 카테고리별로 그룹화
HAVING COUNT(*) >= 2;     -- 3. 건수가 2개 이상인 그룹만 남김
```
- WHERE에서 category != '도서'처럼 부정 조건을 쓰면 NULL인 행도 함께 제외되므로, NULL도 남기려면 OR category IS NULL을 함께 써야 함

<br>

### SQL의 논리적 실행 순서
- SQL은 작성하는 순서(SELECT → FROM → WHERE ...)가 아니라 다음의 논리적 순서로 실행됨
  1. FROM: 데이터를 가져올 테이블을 결정함
  2. WHERE: 개별 행을 필터링함
  3. GROUP BY: 남은 행들로 그룹을 만듦
  4. HAVING: 그룹을 필터링함
  5. SELECT: 조회할 컬럼을 고르고, 집계 함수를 계산하고, 별칭을 붙임
  6. ORDER BY: 결과를 정렬함
  7. LIMIT: 반환할 행의 개수를 제한함
- 이 순서 때문에 WHERE에서는 SELECT의 별칭을 사용할 수 없지만, ORDER BY에서는 사용할 수 있음
  - WHERE가 실행되는 시점에는 SELECT가 아직 실행되지 않아 별칭이 존재하지 않음(Error 1054: Unknown column)
```
SELECT customer_name, SUM(price * quantity) AS total_purchase -- 5
FROM order_stat                                              -- 1
WHERE order_date < '2025-05-14'                              -- 2
GROUP BY customer_name                                       -- 3
HAVING COUNT(*) >= 2                                         -- 4
ORDER BY total_purchase DESC                                 -- 6
LIMIT 1;                                                     -- 7
```
- 논리적 실행 순서와 물리적 실행 순서
  - DB 엔진은 성능 최적화를 위해 내부적으로 순서를 재배열하여 실제(물리적) 실행 순서를 결정함
  - 물리적 순서가 바뀌어도 같은 결과를 보장하므로, 사용자는 논리적 실행 순서에 맞춰 쿼리를 작성하면 됨

<br>

#### 참고
- 인프런 <김영한의 실전 데이터베이스 입문 - SQL부터 차근차근> - 김영한
- MySQL Reference Manual <Aggregate Functions> - https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions.html
- MySQL Reference Manual <MySQL Handling of GROUP BY> - https://dev.mysql.com/doc/refman/8.4/en/group-by-handling.html

#### 배워가는 것들
- COUNT(*)와 COUNT(컬럼)의 차이, 그리고 SUM과 AVG가 NULL을 제외한다는 점을 알고 나니, 누락된 데이터가 집계 결과에 어떤 영향을 주는지 의식하게 되었다.
- GROUP BY를 쓰면 SELECT에 올 수 있는 것이 제한되는 이유를 "그룹을 대표할 수 있는 값인가"로 이해하니 규칙을 외울 필요가 없었다.
- WHERE와 HAVING, 그리고 별칭을 어디서 쓸 수 있는지의 차이는 모두 SQL의 논리적 실행 순서에서 나온다. 복잡한 쿼리를 읽을 때는 실행 순서대로 따라가 보는 습관을 들여야겠다.
