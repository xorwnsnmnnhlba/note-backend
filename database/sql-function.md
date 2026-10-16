# SQL 데이터 가공

### 산술 연산
- SELECT 절에서 숫자 타입의 컬럼에 사칙연산(+, -, *, /)을 그대로 적용하여 새로운 값을 계산할 수 있음
- 계산된 컬럼은 AS로 의미 있는 별칭을 붙여주는 것이 좋음
```
SELECT
    name,
    price * stock_quantity AS total_stock_value, -- 재고 자산 총액
    price + 3000 AS expected_price,              -- 배송비 포함 가격
    price - 1000 AS discounted_price,            -- 할인 가격
    price / 10 AS monthly_payment                -- 10개월 할부 월 납부액
FROM products;
```
- MySQL에서 정수끼리 /로 나누면 소수점이 있는 결과(10000 / 10 = 1000.0000)가 나옴

<br>

### 문자열 함수
- 함수는 값을 받아서 특별한 처리를 한 뒤 그 결과를 반환하는 기능임
- CONCAT(str1, str2, ...): 전달된 문자열을 순서대로 이어 붙임
- CONCAT_WS(separator, str1, str2, ...): 첫 번째 인자로 받은 구분자를 각 문자열 사이에 넣어 이어 붙임(With Separator)
- UPPER(), LOWER(): 문자열을 모두 대문자, 소문자로 바꿈. 대소문자 구분 없이 비교할 때 유용함
- CHAR_LENGTH(): 글자 수를 반환함
- LENGTH(): 바이트 수를 반환함. UTF-8에서 한글은 한 글자가 3바이트이므로 '이순신'은 9가 됨
```
SELECT CONCAT(name, ' (', email, ')') AS name_and_email FROM customers;
-- 이순신 (yisunsin@example.com)

SELECT CONCAT_WS(' - ', name, email, address) AS customer_details FROM customers;
-- 이순신 - yisunsin@example.com - 서울특별시 중구 세종대로

SELECT name, CHAR_LENGTH(name) AS char_length, LENGTH(name) AS byte_length FROM customers;
-- 이순신, 3, 9
```

<br>

### NULL 함수
- NULL을 그대로 노출하면 화면에 아무것도 나오지 않거나 'NULL'이라는 글자가 보일 수 있으므로, 원하는 값으로 대체해서 보여줘야 할 때가 많음
- IFNULL(expr1, expr2): expr1이 NULL이 아니면 expr1을, NULL이면 expr2를 반환함
- COALESCE(value1, value2, ...): 왼쪽부터 차례대로 확인하여 처음으로 NULL이 아닌 값을 반환하며, 모두 NULL이면 NULL을 반환함
  - 여러 대안을 순서대로 제시할 수 있어 IFNULL보다 확장성이 좋음
  - IFNULL은 MySQL의 방언이고, COALESCE는 SQL 표준 함수이므로 호환성도 더 높음
```
SELECT name, IFNULL(description, '상품 설명 없음') AS description FROM products;

-- 짧은 설명 → 긴 설명 → 상품명 → '정보 없음' 순서로 사용(두 설명 컬럼이 있다고 가정)
SELECT COALESCE(short_description, long_description, name, '정보 없음') AS display_text
FROM products;
```

<br>

### SQL 표준 함수
- 대부분의 RDBMS에서 거의 동일하게 동작하는 함수로, DB를 바꾸더라도 쿼리의 호환성을 유지하는 데 도움이 됨
- 함수는 외우지 말고, 이런 것이 있다는 정도만 알아두고 필요할 때 찾아보면 됨

| 분류 | 함수 |
| --- | --- |
| 문자열 | UPPER, LOWER, SUBSTRING, TRIM, POSITION, CHAR_LENGTH |
| 숫자 | ABS, MOD, ROUND, CEILING, FLOOR, SQRT, POWER |
| 날짜와 시간 | CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP, EXTRACT |
| 조건과 변환 | CAST, COALESCE, NULLIF |
| 집계 | COUNT, SUM, AVG, MAX, MIN |

- NULLIF(expr1, expr2): 두 값이 같으면 NULL을, 다르면 expr1을 반환함. 0으로 나누는 것을 피할 때 자주 사용함

<br>

### MySQL 전용 함수
- 각 DBMS는 표준 외에 고유한 함수와 문법을 제공하며, 이를 데이터베이스 방언(Dialect)이라고 함
  - 편리하고 강력하지만, 다른 DB와 호환되지 않는다는 단점이 있음
- 문자열
  - LEFT, RIGHT: 왼쪽, 오른쪽에서 지정한 길이만큼 자름
  - LPAD, RPAD: 지정한 길이가 될 때까지 왼쪽, 오른쪽에 문자를 채움
  - INSTR, LOCATE: 부분 문자열이 처음 나타나는 위치를 반환함(1부터 시작)
  - REPLACE: 문자열 안의 특정 부분을 모두 치환함
  - SUBSTRING_INDEX(str, delim, count): 구분자를 기준으로 count번째까지의 부분 문자열을 반환함
  - FORMAT(N, D): 숫자를 천 단위 콤마와 소수점 D자리로 포맷함
  - GROUP_CONCAT: GROUP BY로 묶인 여러 행의 문자열을 한 줄로 합침
- 날짜와 시간
  - NOW(): 현재 날짜와 시간을 반환함
  - DATE_FORMAT, STR_TO_DATE: 날짜를 지정한 형식의 문자열로, 문자열을 날짜로 변환함
  - DATE_ADD, DATE_SUB: 날짜에 INTERVAL 1 DAY처럼 기간을 더하거나 뺌
  - DATEDIFF, TIMESTAMPDIFF: 두 날짜의 차이를 일 단위, 또는 지정한 단위로 반환함
  - LAST_DAY: 해당 달의 마지막 날을 반환함
- 숫자
  - RAND: 0 이상 1 미만의 무작위 실수를 반환함
  - TRUNCATE(X, D): 소수점 D자리에서 반올림 없이 버림
- 흐름 제어
  - IF(condition, value_if_true, value_if_false): 조건에 따라 다른 값을 반환함
- 윈도우 함수(MySQL 8.0+): ROW_NUMBER, RANK, DENSE_RANK, NTILE, LAG, LEAD, FIRST_VALUE, LAST_VALUE
  - OVER() 절과 함께 사용하여 행을 유지한 채로 순번, 순위, 앞뒤 행의 값 등을 계산함
- JSON 함수(MySQL 5.7+): JSON_OBJECT, JSON_ARRAY, JSON_EXTRACT(->, ->> 단축 연산자), JSON_SET, JSON_CONTAINS 등
- 기타
  - UUID(): 범용 고유 식별자를 생성함
  - INET_ATON, INET_NTOA: IPv4 주소와 숫자를 서로 변환함
  - GET_LOCK, RELEASE_LOCK: 이름을 붙인 락을 획득, 해제하며 중복 실행을 막을 때 사용함
```
-- 이메일에서 @ 앞부분(아이디)만 추출하고 글자 수 구하기
SELECT
    email,
    SUBSTRING_INDEX(email, '@', 1) AS user_id,
    CHAR_LENGTH(SUBSTRING_INDEX(email, '@', 1)) AS id_length
FROM customers;
```

<br>

#### 참고
- 인프런 <김영한의 실전 데이터베이스 입문 - SQL부터 차근차근> - 김영한
- MySQL Reference Manual <Functions and Operators> - https://dev.mysql.com/doc/refman/8.4/en/functions.html

#### 배워가는 것들
- 조회한 데이터를 애플리케이션에서 다시 계산하지 않아도, SQL 안에서 바로 계산하고 가공할 수 있다는 점을 익힐 수 있었다.
- IFNULL과 COALESCE처럼 비슷한 기능이라도 표준인지 방언인지에 따라 호환성이 달라진다. 가능하면 표준 함수를 먼저 고려해야겠다.
- 함수는 외우는 대상이 아니라 찾아 쓰는 대상이다. 어떤 종류의 함수가 있는지 큰 그림만 기억해두고, 필요할 때 공식 문서를 확인하는 습관을 들여야겠다.
