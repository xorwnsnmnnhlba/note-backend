# HTTP 메서드

### HTTP API 설계의 출발점 - 리소스 식별
- 잘못된 URI 설계 예시 - 행위 중심
```
회원 목록 조회  /read-member-list
회원 조회       /read-member-by-id
회원 등록       /create-member
회원 수정       /update-member
회원 삭제       /delete-member
```
  - URI에 행위(동사)가 포함되어 있다는 점이 문제임
- 리소스 개념의 이해
  - 회원을 등록/수정/조회하는 행위 자체는 리소스가 아니며, 회원이라는 개념 자체가 리소스임
  - 리소스를 어떻게 식별할 것인지에 집중해야 하며, 행위는 모두 배제해야 함
- 리소스와 행위의 분리
  - URI는 리소스만 식별함
  - 리소스(명사)와 리소스를 대상으로 하는 행위(동사)를 분리함
    - 리소스: 회원
    - 행위: 조회, 등록, 삭제, 변경
  - 행위는 HTTP 메서드로 구분함
- 개선된 URI 설계 - 계층 구조 활용
```
회원 목록 조회  /members
회원 조회       /members/{id}
회원 등록       /members/{id}
회원 수정       /members/{id}
회원 삭제       /members/{id}
```
  - 계층 구조상 상위를 컬렉션으로 보고 복수 단어 사용을 권장함(member -> members)

<br>

### HTTP 메서드의 종류
- 주요 메서드
  - GET: 리소스 조회
  - POST: 요청 데이터 처리. 주로 등록에 사용함
  - PUT: 리소스를 대체하며, 해당 리소스가 없으면 생성함
  - PATCH: 리소스 부분 변경
  - DELETE: 리소스 삭제
- 기타 메서드
  - HEAD: GET과 동일하지만 메시지 부분을 제외하고 상태 줄과 헤더만 반환함
  - OPTIONS: 대상 리소스에 대한 통신 가능 옵션(메서드)을 설명하며, 주로 CORS에서 사용함
  - CONNECT: 대상 리소스로 식별되는 서버에 대한 터널을 설정함
  - TRACE: 대상 리소스에 대한 경로를 따라 메시지 루프백 테스트를 수행함

<br>

### GET
- 리소스를 조회할 때 사용함
- 서버에 전달하고 싶은 데이터는 query(쿼리 파라미터, 쿼리 스트링)를 통해 전달함
- 메시지 바디로도 데이터 전달이 가능하지만, 지원하지 않는 곳이 많으므로 권장하지 않음
- URL에 데이터가 노출되므로, 보안이 필요한 민감한 데이터를 교환할 때는 사용하지 않아야 함
- 요청 정보가 URL에 모두 담기므로 북마크하여 사용할 수 있음
```http
GET /members/100 HTTP/1.1
Host: localhost:8080
```
```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 34

{
  "username": "young",
  "age": 20
}
```

<br>

### POST
- 요청 데이터를 처리할 때 사용함
- 메시지 바디를 통해 서버로 요청 데이터를 전달하며, 서버는 바디로 들어온 데이터를 처리하는 모든 기능을 수행함
- 주로 전달된 데이터로 신규 리소스를 등록하거나 프로세스를 처리할 때 사용함
- 요청 데이터가 바디에 담기므로 북마크하여 사용할 수 없음
```http
POST /members HTTP/1.1
Content-Type: application/json

{
  "username": "young",
  "age": 20
}
```
```http
HTTP/1.1 201 Created
Content-Type: application/json
Content-Length: 34
Location: /members/100

{
  "username": "young",
  "age": 20
}
```
- 신규 리소스의 식별자는 서버가 생성하며, 응답의 Location 헤더로 전달함
- 스펙에서 말하는 요청 데이터 처리의 실제 사례
  - HTML 양식에 입력된 필드와 같은 데이터 블록을 데이터 처리 프로세스에 제공(ex. HTML FORM으로 회원 가입, 주문)
  - 게시판, 뉴스 그룹, 메일링 리스트, 블로그 등에 메시지 게시(ex. 게시판 글쓰기, 댓글 달기)
  - 서버가 아직 식별하지 않은 새 리소스 생성(ex. 신규 주문 생성)
  - 기존 자원에 데이터 추가(ex. 한 문서 끝에 내용 추가)
  - 결론적으로 특정 URI에 POST 요청이 오면 요청 데이터를 어떻게 처리할지는 리소스마다 따로 정해야 하며, 정해진 규칙이 없음
- POST의 세가지 사용처
  1. 새 리소스 생성(등록): 서버가 아직 식별하지 않은 새 리소스를 생성함
  2. 요청 데이터 처리: 단순 생성이나 변경을 넘어 프로세스의 상태가 변경되는 경우에 사용함
     - ex. 주문에서 결제완료 -> 배달시작 -> 배달완료로 상태가 변경되는 경우
     - 이 경우 POST의 결과로 새로운 리소스가 생성되지 않을 수도 있음(ex. POST /orders/{orderId}/start-delivery, 컨트롤 URI)
  3. 다른 메서드로 처리하기 애매한 경우: JSON으로 조회 데이터를 넘겨야 하는데 GET 메서드 사용이 어려운 경우 등. 애매하면 POST를 사용함

<br>

### PUT
- 리소스를 대체할 때 사용함
  - 리소스가 있으면 대체(덮어쓰기)하고, 없으면 생성함
- 클라이언트가 리소스를 식별한다는 점이 중요함
  - 클라이언트가 리소스의 위치를 알고 URI를 직접 지정함
  - 이 점이 POST와의 핵심적인 차이임
```http
PUT /members/100 HTTP/1.1
Content-Type: application/json

{
  "username": "old",
  "age": 50
}
```
- 리소스를 완전히 대체한다는 점에 주의해야 함
  - 기존 리소스가 {username: young, age: 20}인 상태에서 username 필드 없이 PUT {age: 50}을 요청하면, {age: 50}으로 대체되어 username 필드가 삭제됨
  - 부분 변경이 아니라 전체 교체라는 점을 반드시 인지해야 함

<br>

### PATCH
- 리소스를 부분 변경할 때 사용함
```http
PATCH /members/100 HTTP/1.1
Content-Type: application/json

{
  "age": 50
}
```
- 기존 리소스가 {username: young, age: 20}인 상태에서 PATCH {age: 50}을 요청하면, {username: young, age: 50}이 되어 age만 변경됨

<br>

### DELETE
- 리소스를 제거할 때 사용함
```http
DELETE /members/100 HTTP/1.1
Host: localhost:8080
```

<br>

### HTTP 메서드의 속성
- 안전(Safe)
  - 호출해도 리소스를 변경하지 않는 성질
  - 계속 호출해서 로그가 쌓여 장애가 발생하는 경우는 고려하지 않으며, 해당 리소스만을 대상으로 판단함
- 멱등(Idempotent)
  - f(f(x)) = f(x). 한 번을 호출하든 100번을 호출하든 결과가 동일한 성질
  - GET: 몇 번을 조회해도 같은 결과가 나오므로 멱등함
  - PUT: 결과를 대체하므로 여러 번 수행해도 최종 결과가 동일하여 멱등함
  - DELETE: 여러 번 삭제해도 삭제된 결과는 동일하므로 멱등함
  - POST: 두 번 호출하면 같은 결제가 중복 발생할 수 있으므로 멱등하지 않음
  - 활용: 서버가 TIMEOUT 등으로 정상 응답을 주지 못했을 때, 클라이언트가 같은 요청을 다시 해도 되는지 판단하는 근거가 됨(자동 복구 메커니즘)
  - 한계: 재요청 중간에 외부 요인으로 리소스가 변경되는 것까지는 고려하지 않음
- 캐시가능(Cacheable)
  - 응답 결과 리소스를 캐시해서 사용해도 되는지에 대한 성질
  - 스펙상으로는 GET, HEAD, POST, PATCH가 캐시 가능함
  - 실제로는 GET, HEAD 정도만 캐시로 사용하며, POST와 PATCH는 본문 내용까지 캐시 키로 고려해야 하므로 구현이 쉽지 않음
- 메서드별 속성 정리
  - GET: 안전 O, 멱등 O, 캐시가능 O. 데이터는 쿼리 파라미터로 전달함
  - HEAD: 안전 O, 멱등 O, 캐시가능 O. 바디를 사용하지 않음
  - POST: 안전 X, 멱등 X, 캐시가능은 스펙상 O이나 실무에서는 사용하지 않음. 바디 사용함
  - PUT: 안전 X, 멱등 O, 캐시가능 X. 바디 사용함
  - PATCH: 안전 X, 멱등 X, 캐시가능은 스펙상 O이나 실무에서는 사용하지 않음. 바디 사용함
  - DELETE: 안전 X, 멱등 O, 캐시가능 X. 바디를 사용하지 않음

<br>

### 클라이언트에서 서버로 데이터 전송
- 전송 방식은 크게 두가지로 나뉨
  - 쿼리 파라미터를 통한 전송: GET을 사용하며, 정렬이나 필터(검색어)에 주로 활용함
  - 메시지 바디를 통한 전송: POST, PUT, PATCH를 사용하며, 회원 가입, 상품 주문, 리소스 등록/변경에 주로 활용함
- 데이터 전송의 네가지 상황
  1. 정적 데이터 조회: 이미지, 정적 텍스트 문서
  2. 동적 데이터 조회: 주로 검색, 게시판 목록에서의 정렬 필터(검색어)
  3. HTML Form을 통한 데이터 전송: 회원 가입, 상품 주문, 데이터 변경
  4. HTTP API를 통한 데이터 전송: 서버 to 서버, 앱 클라이언트, 웹 클라이언트(Ajax)

<br>

### 정적 데이터 조회
```http
GET /static/star.jpg HTTP/1.1
Host: localhost:8080
```
- 이미지, 정적 텍스트 문서를 대상으로 함
- 조회이므로 GET을 사용하며, 쿼리 파라미터 없이 리소스 경로만으로 단순 조회가 가능함

<br>

### 동적 데이터 조회
```http
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```
- 주로 검색, 게시판 목록에서의 정렬 필터(검색어)에 사용함
- 조회 조건을 줄여주는 필터나 조회 결과를 정렬하는 정렬 조건에 활용함
- 조회이므로 GET을 사용하며, 데이터는 쿼리 파라미터로 전달함
- 서버는 쿼리 파라미터를 기반으로 정렬하고 필터해서 결과를 동적으로 생성함

<br>

### HTML Form 데이터 전송
- POST 전송(기본)
```html
<form action="/save" method="post">
  <input type="text" name="username" />
  <input type="text" name="age" />
  <button type="submit">전송</button>
</form>
```
  - 브라우저가 생성하는 요청 메시지는 아래와 같음
```http
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded

username=kim&age=20
```
- GET 전송
```http
GET /save?username=kim&age=20 HTTP/1.1
Host: localhost:8080
```
  - GET은 조회에만 사용해야 하며, 리소스 변경이 발생하는 곳에 사용하면 안 됨
  - 조회 목적(ex. /members?username=kim&age=20)이라면 GET 전송이 적절함
- multipart/form-data(파일 업로드)
```html
<form action="/save" method="post" enctype="multipart/form-data">
  <input type="text" name="username" />
  <input type="text" name="age" />
  <input type="file" name="file1" />
  <button type="submit">전송</button>
</form>
```
```http
POST /save HTTP/1.1
Host: localhost:8080
Content-Type: multipart/form-data; boundary=-----XXX
Content-Length: 10457

------XXX
Content-Disposition: form-data; name="username"

kim
------XXX
Content-Disposition: form-data; name="age"

20
------XXX
Content-Disposition: form-data; name="file1"; filename="intro.png"
Content-Type: image/png

109238a9o0p3eqwokjasd09ou3oirjwoe9u34ouief...
------XXX--
```
  - 마지막 boundary 끝에는 --가 추가됨
- 정리
  - HTML Form을 submit하면 POST 전송이 기본임(회원 가입, 상품 주문, 데이터 변경)
  - Content-Type으로 application/x-www-form-urlencoded를 사용함
    - form의 내용을 메시지 바디를 통해 전송함(Key=Value 형태의 쿼리 파라미터 형식)
    - 전송 데이터를 url encoding 처리함(ex. abc김 -> abc%EA%B9%80)
  - HTML Form은 GET 전송도 가능함
  - Content-Type으로 multipart/form-data를 사용하는 경우
    - 파일 업로드 같은 바이너리 데이터 전송 시 사용함
    - 다른 종류의 여러 파일과 폼의 내용을 함께 전송할 수 있어서 multipart라는 이름이 붙음
  - HTML Form 전송은 GET, POST만 지원함

<br>

### HTTP API 데이터 전송
```http
POST /members HTTP/1.1
Content-Type: application/json

{
  "username": "young",
  "age": 20
}
```
- 사용 상황
  - 서버 to 서버: 백엔드 시스템 간 통신
  - 앱 클라이언트: 아이폰, 안드로이드
  - 웹 클라이언트: HTML Form 전송 대신 자바스크립트를 통한 통신(AJAX)에 사용함(ex. React, Vue.js 같은 웹 클라이언트와의 API 통신)
- 전송 규칙
  - POST, PUT, PATCH: 메시지 바디를 통해 데이터를 전송함
  - GET: 조회에 사용하며, 쿼리 파라미터로 데이터를 전달함
  - Content-Type은 application/json을 주로 사용하며, 사실상 표준으로 자리잡음
  - TEXT, XML, JSON 등을 사용할 수 있음

<br>

### HTTP API 설계 예시
- 세가지 설계 유형
  - HTTP API 컬렉션: POST 기반 등록이며, 서버가 URI를 결정함(ex. 회원 관리 API)
  - HTTP API 스토어: PUT 기반 등록이며, 클라이언트가 URI를 결정함(ex. 정적 컨텐츠 관리, 원격 파일 관리)
  - HTML FORM 사용: POST 기반 등록이며, 서버가 URI를 결정함. GET, POST만 지원함(ex. 웹 페이지 회원 관리)
- 회원 관리 시스템 - POST 기반 등록(컬렉션)
```
회원 목록  /members        -> GET
회원 등록  /members        -> POST
회원 조회  /members/{id}   -> GET
회원 수정  /members/{id}   -> PATCH, PUT, POST
회원 삭제  /members/{id}   -> DELETE
```
  - 클라이언트는 등록될 리소스의 URI를 모름
  - 서버가 새로 등록된 리소스의 URI를 생성해줌(HTTP/1.1 201 Created + Location: /members/100)
  - 컬렉션(Collection): 서버가 관리하는 리소스 디렉토리이며, 서버가 리소스의 URI를 생성하고 관리함. 여기서는 /members가 컬렉션에 해당함
- 파일 관리 시스템 - PUT 기반 등록(스토어)
```
파일 목록      /files              -> GET
파일 조회      /files/{filename}   -> GET
파일 등록      /files/{filename}   -> PUT
파일 삭제      /files/{filename}   -> DELETE
파일 대량 등록  /files              -> POST
```
  - 클라이언트가 리소스의 URI를 알고 있어야 함(ex. PUT /files/star.jpg)
  - 클라이언트가 직접 리소스의 URI를 지정함
  - 스토어(Store): 클라이언트가 관리하는 리소스 저장소이며, 클라이언트가 리소스의 URI를 알고 관리함. 여기서는 /files가 스토어에 해당함
- HTML FORM 사용
```
회원 목록     /members             -> GET
회원 등록 폼  /members/new         -> GET
회원 등록     /members/new 또는 /members            -> POST
회원 조회     /members/{id}        -> GET
회원 수정 폼  /members/{id}/edit   -> GET
회원 수정     /members/{id}/edit 또는 /members/{id} -> POST
회원 삭제     /members/{id}/delete -> POST
```
  - HTML FORM은 GET, POST만 지원하므로 제약이 존재함
  - AJAX 같은 기술로 해결할 수 있으나, 여기서는 순수 HTML FORM만 사용하는 경우를 전제로 함
  - 제약 해결을 위해 동사로 된 리소스 경로를 사용하며, 이를 컨트롤 URI라 함
    - 위 예시의 /new, /edit, /delete가 컨트롤 URI에 해당함
    - HTTP 메서드로 해결하기 애매한 경우에 사용하며, HTTP API에서도 동일하게 적용됨

<br>

### 참고하면 좋은 URI 설계 개념
- 문서(document): 단일 개념을 의미하며, 파일 하나 혹은 객체 인스턴스, DB row에 해당함(ex. /members/100, /files/star.jpg)
- 컬렉션(collection): 서버가 관리하는 리소스 디렉터리이며, 서버가 URI를 생성하고 관리함(ex. /members)
- 스토어(store): 클라이언트가 관리하는 자원 저장소이며, 클라이언트가 URI를 알고 관리함(ex. /files)
- 컨트롤러(controller), 컨트롤 URI: 문서, 컬렉션, 스토어로 해결하기 어려운 추가 프로세스를 실행할 때 사용하며, 동사를 직접 사용함(ex. /members/{id}/delete)
- Collection Pattern에 대한 상세한 내용은 [Collection Pattern](/rest-api/collection-pattern.md), [CQS가 적용된 HTTP Method에서의 CRUD 활용](/rest-api/crud-http-method-using-cqs.md) 참고

<br>

#### 참고
- 인프런 <모든 개발자를 위한 HTTP 웹 기본 지식> - 김영한
- RFC 7231(HTTP/1.1 Semantics and Content): https://tools.ietf.org/html/rfc7231
- https://restfulapi.net/resource-naming

<br>

#### 배워가는 것들
- URI는 리소스만 식별하고 행위는 HTTP 메서드로 표현한다는 원칙을 다시 정리할 수 있었다. API 설계 시 URI에 동사가 들어가려 할 때마다 이 원칙을 기준으로 점검해야 할 것이다
- 안전, 멱등, 캐시가능이라는 세가지 속성이 단순한 분류가 아니라 재시도 로직이나 캐시 적용 여부를 판단하는 실질적인 근거가 된다는 점을 알게 되었다
- 컬렉션과 스토어의 차이가 결국 리소스의 URI를 누가 결정하는가에 달려있다는 점이 인상적이었다. POST와 PUT 중 어느 것을 쓸지 고민될 때 이 기준으로 판단할 수 있을 것이다
