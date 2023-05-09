# CQS가 적용된 HTTP Method에서의 CRUD 활용

### CRUD(Create/Read/Update/Delete)
* 데이터를 다룰 때 사용하는 기본적인 네가지 동작을 의미함.
  * Create/Update/Update의 경우, 대상 데이터의 상태가 변함(Command).
  * Read의 경우, 대상 데이터의 상태가 변하지 않으므로 분산처리 혹은 캐싱이 수월함(Query).

<br>

### CQS(Command-Query Separation)
* 상태를 변환하는 메서드인 명령(Command)과 상태를 반환하는 메서드인 질의(Query)를 구분하는 디자인 패턴.
* 하나의 메서드에서 명령과 질의를 동시에 처리할 수 없다는 의미를 가지고 있음.

<br>

### HTTP Method에서의 CRUD 활용
* Collection Pattern과 HTTP Method를 이용해 CRUD를 표현할 수 있음.
  * GET -> Read
  * POST -> Create
  * PUT, PATCH -> Update
  * DELETE -> Delete
* 예시 - 게시물
  * GET /posts -> 게시물 목록 조회(Read, Collection)
  * GET /posts/{id} -> 게시물 상세정보 조회(Read, Element)
  * POST /posts -> 게시물 생성(Create)
  * PUT 또는 PATCH /posts/{id} -> 게시물 수정(Update, Element)
  * DELETE /posts/{id} -> 게시물 삭제(Delete, Element)
* 예시 - 게시물에 매칭된 댓글(특정 게시물 아래로 표현한다 가정)
  * GET /posts/{post_id}/comments -> 특정 게시물에 대한 댓글 목록 조회
  * GET /posts/{post_id}/comments/{id} -> 특정 게시물에 달린 특정 댓글 상세정보 조회
  * POST /posts/{post_id}/comments -> 특정 게시물에 대한 댓글 작성
  * PUT 또는 PATCH /posts/{post_id}/comments/{id} -> 특정 게시물에 달린 특정 댓글 수정
  * DELETE /posts/{post_id}/comments/{id} -> 특정 게시물에 달린 특정 댓글 삭제
* 예시 - 세션(로그인/로그아웃)
  * POST /session -> 로그인
  * DELETE /session -> 로그아웃
    * CRUD 활용을 위해 구현한 것이며, 실제로는 Spring Security를 이용하여 구현함.

#### 배워가는 것들
* CRUD에 대해 정리할 수 있었다. 
  * 사실 대부분의 구현에 대한 본질은 해당 데이터에 대한 CRUD라 생각한다. API 설계 시에 걸림돌이 되지 않도록 잘 익혀두어야 할 것이다.
