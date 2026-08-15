# HTTP 헤더

### HTTP 헤더 개요
```
header-field = field-name ":" OWS field-value OWS
```
- OWS는 띄어쓰기 허용을 의미하며, field-name은 대소문자를 구분하지 않음
- HTTP 전송에 필요한 모든 부가정보를 담는 용도로 사용함
  - ex. 메시지 바디의 내용, 크기, 압축, 인증, 요청 클라이언트 정보, 서버 정보, 캐시 관리 정보 등
- 표준 헤더가 매우 많으며, 필요 시 임의의 헤더를 추가할 수도 있음

<br>

### 헤더 분류의 변화
- 과거 분류(RFC2616 기준)
  - General 헤더: 메시지 전체에 적용되는 정보(ex. Connection: close)
  - Request 헤더: 요청 정보(ex. User-Agent: Mozilla/5.0 (Macintosh; ..))
  - Response 헤더: 응답 정보(ex. Server: Apache)
  - Entity 헤더: 엔티티 바디 정보(ex. Content-Type: text/html, Content-Length: 3423)
- 과거의 BODY 개념(RFC2616)
  - 메시지 본문(message body)은 엔티티 본문(entity body)을 전달하는 데 사용함
  - 엔티티 본문은 요청이나 응답에서 전달할 실제 데이터를 의미함
  - 엔티티 헤더는 엔티티 본문의 데이터를 해석할 수 있는 정보를 제공함(ex. 데이터 유형, 데이터 길이, 압축 정보)
- 표준의 변화
  - 1999년 RFC2616이 폐기되고, 2014년 RFC7230~7235가 등장함
  - 핵심 변화는 엔티티(Entity)가 표현(Representation)으로 바뀐 것임
    - Representation = Representation Metadata + Representation Data
    - 표현 = 표현 메타데이터 + 표현 데이터
- 현재의 BODY 개념(RFC7230)
  - 메시지 본문(message body)을 통해 표현 데이터를 전달하며, 메시지 본문을 페이로드(payload)라 함
  - 표현은 요청이나 응답에서 전달할 실제 데이터를 의미함
  - 표현 헤더는 표현 데이터를 해석할 수 있는 정보를 제공함(ex. 데이터 유형, 데이터 길이, 압축 정보)

<br>

### 표현 헤더(Representation Headers)
- 요청과 응답 양쪽 모두에서 사용함
- Content-Type: 표현 데이터의 형식을 설명함(미디어 타입, 문자 인코딩)
  - ex. text/html; charset=utf-8, application/json, image/png
  - application/json은 기본이 UTF-8 형식임
  - MIME type에 대한 상세한 내용은 [REST API](/rest-api/rest-api.md) 참고
- Content-Encoding: 표현 데이터를 압축하기 위해 사용함
  - 데이터를 전달하는 쪽에서 압축한 후 인코딩 헤더를 추가하고, 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축을 해제함
  - ex. gzip, deflate, identity
```http
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Encoding: gzip
Content-Length: 521
```
- Content-Language: 표현 데이터의 자연 언어를 표현함(ex. ko, en, en-US)
- Content-Length: 표현 데이터의 길이를 바이트 단위로 표현함
  - Transfer-Encoding(전송 코딩)을 사용하는 경우에는 Content-Length를 사용하면 안 됨

<br>

### 협상(콘텐츠 네고시에이션)
- 클라이언트가 선호하는 표현을 요청하는 기능이며, 협상 헤더는 요청 시에만 사용함
- Accept: 클라이언트가 선호하는 미디어 타입을 전달함
- Accept-Charset: 클라이언트가 선호하는 문자 인코딩을 전달함
- Accept-Encoding: 클라이언트가 선호하는 압축 인코딩을 전달함
- Accept-Language: 클라이언트가 선호하는 자연 언어를 전달함
- 동작 예시
  - Accept-Language 미적용: GET /event 요청 시 기본 언어인 Content-Language: en으로 응답함
  - Accept-Language 적용: GET /event + Accept-Language: ko 요청 시 Content-Language: ko로 응답함(한국어 지원 시)
  - 서버가 ko를 지원하지 않고 de, en만 지원하는 경우: 기본 언어인 Content-Language: de로 응답함
- 협상과 우선순위 1 - Quality Values(q)
  - q 값을 사용하며, 범위는 0~1이고 값이 클수록 높은 우선순위를 가짐. 생략하면 1로 취급함
```
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
```
  - 위 예시의 우선순위는 ko-KR(q 생략, 1) > ko(0.9) > en-US(0.8) > en(0.7) 순임
  - 서버가 de와 en만 지원한다면 우선순위에 따라 en으로 응답함
- 협상과 우선순위 2 - 구체적인 것이 우선함
```
Accept: text/*, text/plain, text/plain;format=flowed, */*
```
  - 위 예시의 우선순위는 text/plain;format=flowed > text/plain > text/* > */* 순임
- 협상과 우선순위 3 - 구체적인 것을 기준으로 미디어 타입을 맞춤
```
Accept: text/*;q=0.3, text/html;q=0.7, text/html;level=1,
        text/html;level=2;q=0.4, */*;q=0.5
```
  - text/html;level=1: 1
  - text/html: 0.7
  - text/plain: 0.3
  - image/jpeg: 0.5
  - text/html;level=2: 0.4
  - text/html;level=3: 0.7

<br>

### 전송 방식
- 단순 전송: Content-Length를 사용하며, 콘텐츠의 길이를 알 때 한 번에 요청하고 응답함
```http
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 3423

<html>
  <body>...</body>
</html>
```
- 압축 전송: Content-Encoding을 사용함
```http
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Encoding: gzip
Content-Length: 521
```
- 분할 전송: Transfer-Encoding을 사용함
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

5
Hello
5
World
0
\r\n
```
  - 데이터를 chunk 단위로 쪼개어 도착하는 대로 표시할 수 있음
  - 분할 전송 시에는 Content-Length를 넣으면 안 됨. 전체 길이를 예측할 수 없으며, 각 chunk에 크기가 명시되어 있기 때문임
- 범위 전송: Range, Content-Range를 사용함
```http
GET /event
Range: bytes=1001-2000
```
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Range: bytes 1001-2000 / 2000
```
  - 다운로드가 중단된 경우의 이어받기 등에 활용함

<br>

### 일반 정보 헤더
- From: 유저 에이전트의 이메일 정보를 담으며, 요청에서 사용함
  - 일반적으로 잘 사용하지 않으며, 검색 엔진 같은 곳에서 주로 사용함
- Referer: 이전 웹 페이지의 주소를 담으며, 요청에서 사용함
  - A에서 B로 이동하는 경우, B를 요청할 때 Referer: A를 포함해서 요청함
  - Referer를 사용하여 유입 경로를 분석할 수 있음
  - referer는 referrer의 오타가 그대로 표준이 된 사례임
- User-Agent: 유저 에이전트 애플리케이션 정보를 담으며, 요청에서 사용함
  - 클라이언트의 애플리케이션 정보(웹 브라우저 정보 등)를 의미함
  - ex. user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36
  - 통계 정보로 활용할 수 있으며, 어떤 종류의 브라우저에서 장애가 발생하는지 파악할 수 있음
- Server: 요청을 처리하는 ORIGIN 서버의 소프트웨어 정보를 담으며, 응답에서 사용함
  - ex. Server: Apache/2.2.22 (Debian), server: nginx
- Date: 메시지가 생성된 날짜와 시간을 담으며, 응답에서 사용함
  - ex. Date: Tue, 15 Nov 1994 08:12:31 GMT

<br>

### 특별한 정보 헤더
- Host: 요청한 호스트 정보(도메인)를 담음
  - 요청에서 사용하며, 필수 헤더임
  - 하나의 서버가 여러 도메인을 처리해야 하거나, 하나의 IP 주소에 여러 도메인이 적용되어 있을 때 필요함
  - 가상호스트를 통해 여러 도메인을 한 서버에서 처리하는 구조에서, Host 헤더가 없으면 어느 애플리케이션으로 보낼지 판별할 수 없음
```http
GET /hello HTTP/1.1
Host: aaa.com
```
- Location: 페이지 리다이렉션에 사용함
  - 웹 브라우저는 3xx 응답 결과에 Location 헤더가 있으면 해당 위치로 자동 이동함(리다이렉트)
  - 201 Created의 경우, Location 값은 요청에 의해 생성된 리소스의 URI를 의미함
  - 3xx Redirection의 경우, Location 값은 요청을 자동으로 리다이렉션하기 위한 대상 리소스를 가리킴
- Allow: 허용 가능한 HTTP 메서드를 담음
  - 405(Method Not Allowed) 응답에 포함해야 함(ex. Allow: GET, HEAD, PUT)
- Retry-After: 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간을 담음
  - 503(Service Unavailable) 응답 시 서비스가 언제까지 불능인지 알려줄 수 있음
  - 날짜로 표기하거나(ex. Retry-After: Fri, 31 Dec 1999 23:59:59 GMT), 초 단위로 표기함(ex. Retry-After: 120)

<br>

### 인증 헤더
- Authorization: 클라이언트 인증 정보를 서버에 전달할 때 사용함(ex. Authorization: Basic xxxxxxxxxxxxxxxx)
- WWW-Authenticate: 리소스 접근 시 필요한 인증 방법을 정의할 때 사용하며, 401 Unauthorized 응답과 함께 사용함
  - ex. WWW-Authenticate: Newauth realm="apps", type=1, title="Login to \"apps\"", Basic realm="simple"
- 인증과 인가에 대한 상세한 내용은 [Authentication](/spring-security/authentication.md) 참고

<br>

### 쿠키
- Set-Cookie: 응답에서 사용하며, 서버에서 클라이언트로 쿠키를 전달함
- Cookie: 요청에서 사용하며, 클라이언트가 서버에서 받은 쿠키를 저장했다가 HTTP 요청 시 서버로 전달함
- 쿠키가 없을 때의 문제
  1. 처음 welcome 페이지에 접근하면 "안녕하세요. 손님"이 노출됨
  2. 로그인(POST /login, user=홍길동)하면 "홍길동님이 로그인했습니다"가 노출됨
  3. 로그인 이후 welcome 페이지에 다시 접근하면 또 "안녕하세요. 손님"이 노출됨
  - 원인은 HTTP가 무상태(Stateless) 프로토콜이기 때문임
    - 클라이언트와 서버가 요청과 응답을 주고받으면 연결이 끊어짐
    - 클라이언트가 다시 요청해도 서버는 이전 요청을 기억하지 못하며, 서로 상태를 유지하지 않음
- 대안으로 모든 요청에 사용자 정보를 포함하는 방법
```http
GET /welcome?user=홍길동 HTTP/1.1
GET /board?user=홍길동 HTTP/1.1
GET /order?user=홍길동 HTTP/1.1
```
  - 모든 요청과 모든 링크에 사용자 정보가 포함되도록 개발해야 하며, 브라우저를 완전히 종료하고 다시 열면 정보가 사라진다는 문제가 있음
- 쿠키를 사용한 해결
  - 로그인 시
```http
POST /login HTTP/1.1
user=홍길동
```
```http
HTTP/1.1 200 OK
Set-Cookie: user=홍길동
```
    - 웹 브라우저 내부의 쿠키 저장소에 user=홍길동이 저장됨
  - 이후 요청 시
```http
GET /welcome HTTP/1.1
Cookie: user=홍길동
```
    - 쿠키 저장소에서 자동으로 조회되어 모든 요청에 쿠키 정보가 자동으로 포함됨
- 쿠키의 사용처와 주의사항
```
set-cookie: sessionId=abcde1234; expires=Sat, 26-Dec-2020 00:00:00 GMT; path=/; domain=.google.com; Secure
```
  - 사용처: 사용자 로그인 세션 관리, 광고 정보 트래킹
  - 쿠키 정보는 항상 서버에 전송되어 네트워크 트래픽을 추가로 유발하므로, 최소한의 정보만 사용해야 함(세션 id, 인증 토큰)
  - 서버에 전송하지 않고 브라우저 내부에만 데이터를 저장하려면 웹 스토리지(localStorage, sessionStorage)를 사용해야 함
  - 보안에 민감한 데이터는 저장하면 안 됨(주민번호, 신용카드 번호 등)
- 쿠키 - 생명주기(Expires, max-age)
  - Set-Cookie: expires=Sat, 26-Dec-2020 04:39:21 GMT: 만료일이 되면 쿠키를 삭제함
  - Set-Cookie: max-age=3600: 3600초 후에 삭제하며, 0이나 음수를 지정하면 즉시 삭제함
  - 세션 쿠키: 만료 날짜를 생략하면 브라우저 종료 시까지만 유지됨
  - 영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지됨
- 쿠키 - 도메인(Domain)
  - 명시한 경우(ex. domain=example.org): 명시한 문서 기준 도메인과 서브 도메인을 포함하므로, example.org는 물론 dev.example.org에서도 쿠키에 접근할 수 있음
  - 생략한 경우: 현재 문서 기준 도메인만 적용되므로, example.org에서만 접근 가능하며 dev.example.org에서는 접근할 수 없음
- 쿠키 - 경로(Path)
  - 지정한 경로를 포함한 하위 경로 페이지만 쿠키에 접근할 수 있으며, 일반적으로 path=/ 루트로 지정함
  - ex. path=/home으로 지정한 경우, /home, /home/level1, /home/level1/level2는 접근 가능하지만 /hello는 접근할 수 없음
- 쿠키 - 보안(Secure, HttpOnly, SameSite)
  - Secure: 쿠키는 기본적으로 http와 https를 구분하지 않고 전송되지만, Secure를 적용하면 https인 경우에만 전송함
  - HttpOnly: XSS 공격을 방지함. 자바스크립트에서 접근할 수 없으며(document.cookie), HTTP 전송에만 사용함
  - SameSite: XSRF 공격을 방지함. 요청 도메인과 쿠키에 설정된 도메인이 같은 경우에만 쿠키를 전송함

<br>

### 헤더 기능별 분류 정리
- 표현: Content-Type, Content-Encoding, Content-Language, Content-Length
- 협상(요청 전용): Accept, Accept-Charset, Accept-Encoding, Accept-Language
- 전송 방식: Transfer-Encoding, Range, Content-Range
- 일반 정보: From, Referer, User-Agent, Server, Date
- 특별한 정보: Host, Location, Allow, Retry-After
- 인증: Authorization, WWW-Authenticate
- 쿠키: Set-Cookie, Cookie
- 캐시 제어: Cache-Control, Pragma, Expires
- 캐시 검증: ETag, Last-Modified
- 조건부 요청: If-Match, If-None-Match, If-Modified-Since, If-Unmodified-Since
- 캐시 관련 헤더에 대한 상세한 내용은 [HTTP 캐시와 조건부 요청](/http/http-cache.md) 참고

<br>

#### 참고
- 인프런 <모든 개발자를 위한 HTTP 웹 기본 지식> - 김영한
- RFC 7230(HTTP/1.1 Message Syntax): https://tools.ietf.org/html/rfc7230
- RFC 7235(HTTP/1.1 Authentication): https://tools.ietf.org/html/rfc7235
- HTTP 헤더 필드 목록: https://en.wikipedia.org/wiki/List_of_HTTP_header_fields

<br>

#### 배워가는 것들
- 엔티티(Entity)에서 표현(Representation)으로 용어가 바뀐 배경을 알게 되었다. 오래된 자료와 최신 스펙에서 용어가 다르게 쓰이는 이유를 이해할 수 있었다
- 헤더를 표현, 협상, 전송 방식, 인증, 쿠키, 캐시라는 기능 단위로 묶어서 볼 수 있게 되었다. 새로운 헤더를 만났을 때 어느 갈래에 속하는지 판단하는 기준이 될 것이다
- 쿠키는 모든 요청에 자동으로 실려나가기 때문에 최소한의 정보만 담아야 하며, 보안 속성(Secure, HttpOnly, SameSite)을 반드시 함께 고려해야 한다는 점을 익혔다
