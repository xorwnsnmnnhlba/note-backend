# URI와 웹 브라우저 요청 흐름

### URI, URL, URN의 관계
- URI(Uniform Resource Identifier)가 가장 큰 개념이며, URL과 URN은 URI의 하위 개념임
- RFC 3986 기준으로 URI는 로케이터(locator), 이름(name), 또는 둘 다로 분류될 수 있음
```
                    URI
        ┌────────────┴────────────┐
       URL                       URN
   (Resource Locator)      (Resource Name)
```
- 단어의 뜻
  - Uniform: 리소스를 식별하는 통일된 방식
  - Resource: 자원. URI로 식별할 수 있는 모든 것을 의미하며, 제한이 없음
  - Identifier: 다른 항목과 구분하는 데 필요한 정보
  - URL(Locator): 리소스가 있는 위치를 지정함
  - URN(Name): 리소스에 이름을 부여함
- 위치는 변할 수 있으나 이름은 변하지 않는다는 점이 URL과 URN의 근본적인 차이임
  - URN 예시: urn:isbn:8960777331 (특정 도서의 ISBN URN)
- URN만으로 실제 리소스를 찾는 방법이 보편화되지 않았기 때문에, 실무에서는 URI를 URL과 거의 같은 의미로 통용함

<br>

### URL 전체 문법
```
scheme://[userinfo@]host[:port][/path][?query][#fragment]

https://www.google.com:443/search?q=hello&hl=ko
```
- scheme: 주로 프로토콜이 들어가며, 어떤 방식으로 자원에 접근할지에 대한 약속을 의미함(ex. http, https, ftp)
  - https는 http에 보안을 추가한 것임(HTTP Secure)
- userinfo: URL에 사용자 정보를 포함하여 인증할 때 사용하며, 거의 사용하지 않음
- host: 호스트명. 도메인명 또는 IP 주소를 직접 사용할 수 있음
- port: 접속 포트. 일반적으로 생략하며, 생략 시 http는 80, https는 443이 적용됨
- path: 리소스 경로이며, 계층적 구조를 가짐(ex. /members/100)
- query: Key-Value 형태로 들어가며, ?로 시작하여 &로 추가함. 문자 형태로 웹 서버에 전달되며, 쿼리 파라미터(query parameter) 혹은 쿼리 스트링(query string)이라고도 함
- fragment: HTML 내부 북마크 등에 사용하며, 서버로 전송되지 않는 정보임

<br>

### 웹 브라우저 요청 흐름
- https://www.google.com/search?q=hello&hl=ko 를 입력했을 때의 전체 흐름
  1. DNS 조회를 통해 www.google.com의 IP 주소를 획득함(ex. 200.200.200.2)
  2. HTTPS이므로 생략된 PORT는 443으로 결정됨
  3. HTTP 요청 메시지를 생성함
```http
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```
  4. SOCKET 라이브러리를 통해 OS로 전달함(TCP/IP 연결 -> 데이터 전달)
  5. TCP/IP 패킷을 생성하고, HTTP 메시지를 데이터 영역에 포함시킴
```
TCP/IP 패킷
 ├─ 출발지 IP, PORT
 ├─ 목적지 IP, PORT
 └─ 전송 데이터: [HTTP 메시지]
```
  6. 인터넷 망을 통해 요청 패킷이 전달되어 구글 서버에 도착함
  7. 서버가 HTTP 응답 메시지를 생성함
```http
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 3423

<html>
  <body>...</body>
</html>
```
  8. 응답 패킷이 클라이언트에 도착함
  9. 웹 브라우저가 전달받은 HTML을 렌더링함

<br>

#### 참고
- 인프런 <모든 개발자를 위한 HTTP 웹 기본 지식> - 김영한
- RFC 3986(URI): https://www.ietf.org/rfc/rfc3986.txt
- https://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn

<br>

#### 배워가는 것들
- 평소 혼용해서 쓰던 URI와 URL의 관계를 명확하게 정리할 수 있었다. 실무에서는 사실상 같은 의미로 쓰이지만, 개념적으로는 URI가 상위 개념이라는 점을 기억해야 한다.
- 브라우저에 주소를 입력한 순간부터 화면이 그려지기까지의 흐름을 DNS 조회, TCP 연결, 메시지 생성, 렌더링 단계로 나누어 볼 수 있게 되었다. 문제가 생겼을 때 어느 단계에서 끊겼는지 짚어보는 기준으로 활용할 수 있을 것이다.
