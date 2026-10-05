# 인덱스와 실행 계획

### 인덱스(Index)
- 테이블의 특정 컬럼 값을 정렬된 형태로 따로 저장해두어, 조건에 맞는 행을 빠르게 찾을 수 있도록 하는 자료구조
  - 책의 맨 뒤에 있는 색인(찾아보기)과 같은 역할을 함
  - 인덱스가 없으면 조건에 맞는 행을 찾기 위해 테이블 전체를 처음부터 끝까지 읽어야 함(Full Table Scan)
- 대부분의 RDBMS는 기본 인덱스로 B-Tree(Balanced Tree)를 사용함
  - 값이 정렬되어 있으므로 동등 조건(=)뿐 아니라 범위 조건(<, >, BETWEEN)과 정렬(ORDER BY)에도 활용됨
- 인덱스는 공짜가 아님
  - 데이터를 INSERT, UPDATE, DELETE 할 때마다 인덱스도 함께 갱신해야 하므로 쓰기 성능이 떨어짐
  - 별도의 저장 공간을 차지함
  - 따라서 실제로 자주 사용하는 조회 조건을 기준으로 필요한 만큼만 만들어야 함

<br>

### 복합 인덱스(Composite Index)
- 여러 컬럼을 묶어서 만든 인덱스로써, 컬럼의 순서가 중요함
  - (suite_id, started_at)으로 만든 인덱스는 suite_id로 먼저 정렬되고, 같은 suite_id 안에서 started_at으로 정렬됨
  - 따라서 suite_id 조건 없이 started_at만으로 조회하면 이 인덱스를 효율적으로 사용할 수 없음
- 일반적으로 동등 조건으로 쓰이는 컬럼을 앞에, 범위 조건이나 정렬에 쓰이는 컬럼을 뒤에 둠
- 정렬 방향까지 지정해두면, "특정 스위트의 최신 실행 N건"처럼 정렬 후 일부만 가져오는 조회를 정렬 작업 없이 인덱스 순서대로 읽고 멈출 수 있음
```
CREATE INDEX idx_run_suite_time ON test_run(suite_id, started_at DESC);
CREATE INDEX idx_run_time       ON test_run(started_at DESC, id DESC);
```
- 범위 조건은 인덱스를 어디서부터 읽기 시작할지 정해주는 역할을 함
  - "이 실행 직전의 실행 1건"을 찾을 때 id < ? 조건만 있으면 인덱스(started_at 순)의 처음부터 훑어야 하지만, started_at <= ? 조건을 함께 주면 그 위치부터 거슬러 읽으므로 훨씬 빨라짐
  - 논리적으로는 없어도 같은 결과가 나오는 조건이, 성능을 위해서는 필요할 수 있음

<br>

### 실행 계획(Execution Plan)
- DBMS의 옵티마이저(Optimizer)가 쿼리를 어떤 방식으로 수행할지 결정한 계획
  - 어떤 인덱스를 사용할지, 어떤 순서로 테이블을 조인할지, 몇 건이 나올지 추정하여 비용이 가장 낮은 방법을 고름
- PostgreSQL에서는 EXPLAIN으로 실행 계획을 확인할 수 있음
  - EXPLAIN: 실제로 실행하지 않고 예상 계획만 보여줌
  - EXPLAIN ANALYZE: 실제로 실행한 후, 단계별 실제 소요 시간과 행 수를 함께 보여줌
  - EXPLAIN (ANALYZE, BUFFERS): 읽은 페이지 수까지 확인할 수 있음
```
EXPLAIN ANALYZE
SELECT * FROM test_run
WHERE suite_id = 1
ORDER BY started_at DESC
FETCH FIRST 20 ROWS ONLY;
```
- 실행 계획에서 자주 보이는 탐색 방식
  - Seq Scan: 테이블 전체를 순서대로 읽음
  - Index Scan: 인덱스로 위치를 찾은 뒤 테이블에서 행을 읽음
  - Index Only Scan: 필요한 컬럼이 모두 인덱스에 있어 테이블을 읽지 않음
  - Bitmap Heap Scan: 인덱스로 대상 위치를 모아둔 뒤 테이블을 한꺼번에 읽음
- 옵티마이저가 행 수를 잘못 추정하면 잘못된 계획을 고르기도 하므로, 예상 행 수(rows)와 실제 행 수(actual rows)의 차이도 함께 확인해야 함
- 데이터가 적을 때는 어떤 쿼리든 빠르므로, 운영 규모에 가까운 데이터를 만들어두고 측정해야 의미가 있음
  - PostgreSQL에서는 generate_series()와 random()으로 대량의 데이터를 손쉽게 만들 수 있음

<br>

### 이력이 쌓일수록 느려지는 조회 개선하기
- 집계 값을 미리 저장해두기(반정규화)
  - 실행 목록에 통과·실패 건수를 보여주기 위해 매번 결과 테이블을 조인하여 COUNT하면, 결과가 쌓일수록 느려짐
  - 실행 테이블에 passed_count, failed_count 같은 컬럼을 두고 결과를 저장하는 트랜잭션에서 함께 올려두면, 목록 조회는 실행 테이블만 읽으면 됨
  - 같은 트랜잭션에서 갱신해야 저장된 결과와 집계 값이 어긋나지 않음
- 페이지 번호 기반(OFFSET) 조회의 한계
  - OFFSET 10000은 앞의 10000건을 읽고 버린 뒤에 다음 건을 가져오므로, 뒤쪽 페이지일수록 느려짐
  - 마지막으로 본 행의 값을 기준으로 다음 페이지를 가져오는 방식(Keyset Pagination)을 쓰면 페이지 위치와 무관하게 일정한 속도를 낼 수 있음
- 인덱스를 추가했는데 오히려 느려지는 경우도 있으므로, 추가 전후를 반드시 측정해서 비교해야 함

<br>

### 고급 SQL 문법
- 창 함수(Window Function)
  - 행을 묶어서 하나로 줄이는 GROUP BY와 달리, 각 행을 유지하면서 같은 그룹(PARTITION) 안의 다른 행 값을 참조함
  - LAG(): 정렬 기준으로 바로 앞 행의 값을 가져옴. 직전 실행과 비교할 때 사용함
  - ROW_NUMBER(), RANK(): 그룹 안에서의 순번을 매김
```
SELECT id,
       LAG(id) OVER (PARTITION BY suite_id ORDER BY started_at, id) AS previous_id
FROM test_run;
```
- 백분위 집계
  - PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY ...): 값을 정렬했을 때 95% 지점의 값을 구함(p95)
  - 응답 시간처럼 일부 극단값이 평균을 왜곡하는 지표에서는 평균보다 p50, p95 같은 백분위를 많이 사용함
- LATERAL 조인
  - 오른쪽 하위 쿼리에서 왼쪽 테이블의 컬럼을 참조할 수 있도록 해주는 조인
  - "케이스마다 최근 결과 20건"처럼 그룹별로 상위 N건을 가져올 때, 그룹마다 인덱스를 필요한 만큼만 읽도록 할 수 있음
```
SELECT c.id, r.*
FROM test_case c
CROSS JOIN LATERAL (
    SELECT * FROM test_result r
    WHERE r.test_case_id = c.id
    ORDER BY r.id DESC
    FETCH FIRST 20 ROWS ONLY
) r;
```
- FETCH FIRST n ROWS ONLY는 LIMIT에 대응하는 SQL 표준 문법임
- PostgreSQL에서 하위 쿼리에 OFFSET 0을 붙이면, 옵티마이저가 하위 쿼리를 바깥 쿼리와 합쳐서 계획하지 못하도록 막는 경계가 됨
  - 옵티마이저가 행 수를 잘못 추정하여 잘못된 계획을 고를 때 사용하는 방법이며, 표준 동작이 아니므로 이유를 주석으로 남겨두는 것이 좋음

<br>

#### 참고
- PostgreSQL Documentation <Indexes> - https://www.postgresql.org/docs/current/indexes.html
- PostgreSQL Documentation <Using EXPLAIN> - https://www.postgresql.org/docs/current/using-explain.html
- PostgreSQL Documentation <Window Functions> - https://www.postgresql.org/docs/current/tutorial-window.html

#### 배워가는 것들
- 인덱스는 무조건 많을수록 좋은 것이 아니라, 컬럼 순서와 정렬 방향까지 조회 방식에 맞춰야 효과가 있다는 것을 알게 되었다.
- 결과가 같은 조건이라도 범위 조건이 있느냐 없느냐에 따라 인덱스를 읽는 양이 크게 달라질 수 있다는 점이 인상적이었다. 실행 계획을 확인하지 않으면 알기 어려운 부분이다.
- 데이터가 적을 때의 측정은 의미가 없다는 것을 배웠다. 성능을 고민할 때는 실제 규모의 데이터를 먼저 만들고, 변경 전후를 같은 조건에서 비교해야겠다.
