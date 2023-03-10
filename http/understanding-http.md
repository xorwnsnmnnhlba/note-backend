# HTTP의 이해

* HTTP(HyperText Transfer Protocol)
  * HTML과 같은 하이퍼미디어 문서를 가져오기 위해 사용하는 응용 계층 프로토콜. Web 상에서 이루어지는 데이터 통신의 기초 프로토콜이라 할 수 있음.

<br>

* OSI 7계층(OSI 7-Layer)
  * 물리/데이터링크/네트워크/전송/세션/표현/응용 계층으로 구분함. 여기서는 데이터링크/네트워크/전송/응용 계층만 정리.
  * 데이터링크 계층: 소프트웨어 측면에서 하드웨어를 추상화하여 구분할 수 있는 계층. 이를 위해 하드웨어에서 제공하는 MAC Address로 구분함. 원격에서는 인지 불가능.
  * 네트워크 계층: 인터넷을 이용하여 서로 떨어진 대상을 구분할 수 있는 계층. 이를 위해 IP Address를 할당하여 구분함.
  * 전송 계층: 하나 혹은 복수의 하드웨어에서 프로그램 간 데이터통신을 위한 계층. 이를 위해 TCP/UDP와 Port Number를 통해 구분함.
  * 응용 계층: 응용프로그램을 이용하여 통신을 위한 인터페이스를 제공하는 계층. 대표적으로 웹 브라우징을 할 때 쓰는 HTTP/HTTPS가 있음.

<br>

* HTTP와 HTTPS의 차이
  * HTTPS의 경우, 보안을 위해 TLS 프로토콜을 한번 거쳐서 HTTP 프로토콜을 이용하여 통신할 수 있도록 함.

<br>

<figure><img src="./images/http-layers.png" alt=""></figure>

<br>

* 클라이언트-서버 모델
  * 클라이언트(Client): 임의의 프로세스를 요청하는 서비스 이용자.
  * 서버(Server): 요청한 프로세스를 처리하여 응답하는 서비스 제공자.
    * 서버는 어떠한 서비스를 제공해야하는지 명확하게 정의해야 하며, 클라이언트는 서버가 제공한 서비스를 이용하기 위해 리소스를 명확하게 요청해야 함. URL(Uniform Resource Locator)을 가지고 클라이언트-서버 간 통신 수행.

<br>

* 무상태(Stateless)/상태(Stateful) 프로토콜
  * 무상태(Stateless) 프로토콜: 각각의 요청이 종속되지 않고 독립적으로 움직이는 프로토콜.
  * 상태(Stateful) 프로토콜: 각각의 요청이 종속되어 움직이는 프로토콜.

<br>

* 쿠키와 세션
  * HTTP는 무상태 프로토콜이므로, 메시지를 주고받을 때 클라이언트 입장에서는 서버에게 항상 누구인지 인지시켜줘야 함.
  * 쿠키(Cookie): 상태정보에 대한 Key와 Value가 들어있는 파일. 클라이언트와 서버가 상태정보에 대한 요청과 응답을 지속적으로 주고받으며, 그 정보가 클라이언트에 저장됨.
  * 세션(Session): 쿠키를 기반으로 한 상태정보 파일. 쿠키와 다르게 상태정보가 서버에 저장됨. 일반적으로 세션에 대한 Key는 쿠키를 통해 관리함.

<br>

* HTTP 메시지 구조

<figure><img src="./images/httpmsgstructure2.png" alt=""></figure>

* 기본적으로는 사람이 인지할 수 있는 형태로 짜여있음.
* 시작 줄(Start Line) - 헤더(Headers) - 빈 줄(Empty Line) - 본문(Body) 형태로 구성됨.
* HTTP 요청(Request)과 응답(Response) 모두 동일한 구조로 되어있음.
  * 시작 줄(Start Line): 실행해야 할 요청 혹은 수행한 요청에 대한 응답코드가 기록되어 있음. 반드시 한 줄로 끝나야 함.
    * 요청: (HTTP Method) (요청 URL) (HTTP 버전)
    * 응답: (HTTP 버전) (응답 상태코드)
  * 헤더(Headers): 클라이언트가 서버에 요청할 때, 부가적인 정보(ex. Host명, 사용자 에이전트, 데이터 형식 등등)를 전송하기 위해 사용.
  * 빈 줄(Empty Line): 요청에 대한 모든 메타정보가 전송되었음을 알릴 때 사용.
  * 본문(Body): 요청의 가장 마지막 부분에 들어감. 크기를 예측하기 어려우므로, 헤더에 Content-Length를 활용함. 텍스트뿐만 아니라 바이너리 형태 데이터도 넣을 수 있음. 하나가 아니라 여러 데이터를 넣을 수 있음.

<br>

* HTTP 메서드(HTTP Method)
  * GET
    * 서버에 있는 리소스를 클라이언트로 가져오기(Read) 위한 동작.
    * 동일한 요청을 여러차례 보내도 같은 결과가 나오는 멱등성(Idempotent)을 보장하며, 캐싱(Caching) 가능함.
    * URL에 데이터가 노출되기 때문에, 보안이 필요한 민감한 데이터를 교환할때는 사용하지 않아야 함.
    * 북마크하여 사용 가능.
  * HEAD
    * GET 요청과 유사하지만, 응답 본문을 받아오지 않고 응답 헤더만 받아옴.
  * POST
    * 클라이언트가 서버에 특정 데이터를 추가(Create)하기 위한 동작.
    * 서버로 보낼 데이터를 본문에 담아서 통신 진행.
    * 멱등성을 보장하지 않으며, 캐싱 불가능함.
    * 북마크 사용 불가능함.
  * PUT
    * 클라이언트가 서버에 기존 데이터를 수정(Update)하기 위한 동작.
  * PATCH
    * PUT 요청과 유사하지만, 기존 데이터와 새로운 데이터와의 차이가 있는 부분만 전송함.
    * 멱등성을 보장하지 않음.
  * DELETE
    * 서버에 특정 데이터를 삭제(Delete)하기 위한 동작.
  * OPTIONS
    * 사용할 수 있는 HTTP Method를 확인할 때 사용.
    * 서버 또는 특정 리소스가 제공하는 기능을 확인할 수 있음.

<br>

* HTTP 상태코드(HTTP status code)
  * HTTP를 이용하여 통신을 진행할 때, 서버는 클라이언트 요청에 대한 응답을 전달해줘야 한다.
  * HTTP 상태코드를 통해 응답에 대한 분류가 가능하다.
    * 1xx - 정보 응답
      * 직접 쓰는 일은 거의 없음.
    * 2xx - 성공 응답
      * 200 OK: 요청을 성공적으로 수행했을 때 반환하며, GET 요청에서 주로 사용함.
      * 201 Created: 요청을 성공적으로 수행했으며, 그에 따른 결과로 새로운 리소스가 만들어졌을 때 반환함. 주로 POST 요청에서 사용함.
      * 204 No Content: 요청은 성공적으로 끝났지만, 제공할 수 있는 콘텐츠가 없을 때 반환함.
    * 3xx - 리다이렉션
      * 304 Not Modified: GET 요청을 보낸 응답 데이터가 클라이언트에 캐시된 데이터와 동일할 때 반환함. 만약 응답할 데이터가 변경되면, 변경된 데이터를 보내게 됨.
    * 4xx - 클라이언트쪽 문제
      * 400 Bad Request: 클라이언트가 서버에서 제공하지 않는 요청을 보냈을 때 반환함.
      * 401 Unautorized: 리소스 요청 시 그에 따른 인증자격이 없을 때 반환함.
      * 403 Forbidden: GET 요청 시 응답 데이터에 접근할 권한이 없거나 POST/PUT 요청 시 데이터 생성 혹은 수정할 권한이 없을 때 반환함.
      * 404 Not Found: 존재하지 않는 리소스를 요청했을 때 반환함.
      * 405 Method Not Allowed: 서버에서 지원하지 않는 HTTP 메서드를 가지고 요청했을 때 반환함.
    * 5xx - 서버쪽 문제
      * 500 Internal Server Error: 서버가 클라이언트 요청을 처리할 수 없을 때 반환함.
      * 503 Service Unavailable: 서버에 발생한 과부하 혹은 동작오류로 인해 클라이언트 요청을 처리할 준비가 되지 않았을 때 반환함.

<br>

* 참고
  * https://developer.mozilla.org/ko/docs/Web/HTTP
