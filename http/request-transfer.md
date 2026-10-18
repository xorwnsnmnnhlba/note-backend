# HTTP 요청 전송의 세부 동작

### 같은 요청인데 결과가 다른 이유
- curl이나 브라우저로는 정상인 API가, 애플리케이션의 HTTP Client로 호출하면 실패하는 경우가 있음
- 대부분 요청 줄과 헤더는 같지만, 본문을 보내는 방식, 기본 헤더, 연결 방식처럼 눈에 잘 띄지 않는 부분이 다르기 때문임
- 이런 차이를 확인하려면 받는 쪽(Mock Server, 네트워크 캡처)에서 실제로 도착한 요청 원문을 나란히 비교해보는 것이 가장 확실함

<br>

### 본문 길이를 알리는 두 가지 방법
- HTTP/1.1에서 받는 쪽은 본문이 어디서 끝나는지 알아야 다음 요청을 읽을 수 있으므로, 보내는 쪽이 길이를 알려야 함
- Content-Length
  - 본문의 전체 길이(바이트)를 미리 알려줌
  - 보내기 전에 본문 전체 크기를 알아야 함
- Transfer-Encoding: chunked
  - 본문을 여러 조각(Chunk)으로 나누어, 각 조각 앞에 길이를 붙여 보내고 길이 0인 조각으로 끝을 알림
  - 전체 길이를 미리 몰라도 되므로, 스트리밍처럼 만들면서 보내는 응답에 주로 쓰임
  - 두 헤더를 함께 쓰면 안 되며, 관련 내용은 [HTTP 헤더](/http/http-header.md) 참고
```
POST /users HTTP/1.1
Transfer-Encoding: chunked

7\r\n
{"name"\r\n
9\r\n
:"kim"}\r\n
0\r\n
\r\n
```
- 요청 본문을 chunked로 보내면 생길 수 있는 문제
  - 일부 서버, 게이트웨이, 프록시는 길이 없는 요청을 거부하거나(411 Length Required) 다르게 처리함
  - 그래서 보통의 Client처럼 동작하려면, 본문을 미리 바이트 배열로 만들어 Content-Length와 함께 보내는 것이 안전함
  - HTTP Client 라이브러리는 본문을 스트림(InputStream)으로 받으면 길이를 모르므로 chunked로 보내는 경우가 많음
- 요청에 적은 Content-Length가 실제 길이와 다르면
  - 실제보다 크면 서버는 나머지 본문을 기다리다 시간 초과가 남
  - 사용자가 직접 헤더를 지정할 수 있는 도구라면, Content-Length는 실제 길이로 다시 계산해서 보내야 함
  - 길이 정보가 서로 다르게 해석되는 점을 악용하는 공격을 HTTP Request Smuggling이라고 함
- HTTP/2, HTTP/3은 프레임 단위로 데이터를 보내므로 chunked 전송 코딩을 사용하지 않음

<br>

### User-Agent
- 요청을 보낸 Client 프로그램을 알려주는 요청 헤더
- HTTP Client 라이브러리의 기본값은 라이브러리와 런타임 버전을 담고 있음
  - 예: Apache-HttpClient/5.6.4 (Java/25.0.4.1), Java-http-client/25
  - 보안 장비나 WAF(Web Application Firewall)가 자동화 도구로 보고 막는 경우가 있음
  - 상대 서버에 사용 중인 라이브러리와 Java 버전을 알려주게 됨
- 서버 운영자가 로그에서 알아볼 수 있도록, 제품 이름과 버전으로 지정하는 것이 좋음
  - 예: Sentinel/1.0.0
  - 버전은 빌드 정보(build-info.properties)에서 읽으면 릴리스와 자동으로 맞춰짐. 관련 내용은 [릴리스와 배포](/infra/release.md) 참고
- 사용자가 요청 헤더를 직접 지정할 수 있다면, 지정한 값이 기본값보다 우선하도록 함
- Accept 헤더를 보내지 않으면 HTTP 규칙상 "어떤 형식이든 받는다"는 의미이므로, 기본값을 억지로 넣을 필요는 없음

<br>

### 여러 IP 주소를 가진 호스트
- 하나의 도메인은 여러 IP 주소를 가질 수 있음
  - IPv4 주소(A 레코드)와 IPv6 주소(AAAA 레코드)를 함께 가지거나, 부하 분산을 위해 여러 주소를 가짐
  - localhost도 127.0.0.1과 ::1 두 주소로 해석됨
- HTTP Client는 첫 주소로 연결에 실패하면 다음 주소로 다시 시도함
  - IPv6와 IPv4를 동시에 시도하여 먼저 연결되는 쪽을 쓰는 방식을 Happy Eyeballs(RFC 8305)라고 함
- 이때 주의할 점: 마지막으로 시도한 주소의 실패만 예외로 남아 진짜 원인이 가려질 수 있음
  - 예: 첫 주소에서 인증서 오류로 실패 → 다음 주소는 연결 자체가 안 됨 → 최종 오류는 "연결 실패"
  - 실패 원인을 정확히 보고해야 한다면, 주소별 시도 결과를 기록해두고 최종 실패 시 함께 확인해야 함
- DNS 해석 결과를 검사하여 위험한 주소를 막는 방법은 [웹 취약점과 방어](/spring-security/web-vulnerability.md) 참고

<br>

### TLS 연결 과정에서의 실패
- HTTPS 요청은 TCP 연결 후 TLS Handshake를 거쳐야 HTTP 요청을 보낼 수 있음
  - SNI(Server Name Indication): Client가 Handshake 시작 시 접속할 호스트 이름을 알려, 하나의 IP에서 여러 인증서를 쓰는 서버가 알맞은 인증서를 고를 수 있도록 함
- 자주 발생하는 TLS 실패
  - 인증서 만료
  - 인증서의 이름과 접속한 호스트 이름이 다름
  - 신뢰하지 않는 인증 기관이 발급한 인증서(사내 인증 기관, 자체 서명 인증서)
  - 중간 인증서 누락, 서로 지원하는 TLS 버전이나 암호 방식이 없음
- Java에서는 SSLHandshakeException, SSLPeerUnverifiedException 등으로 나타남
- TLS 실패는 일반적인 연결 끊김과 구분해서 다루는 것이 좋음
  - "인증서 만료"처럼 원인이 분명하고 흔한 장애라서, 따로 분류하면 바로 알아볼 수 있음
  - 다시 보내도 같은 결과이므로 재시도 대상이 아님. 재시도 판단은 [타임아웃과 재시도](/spring-framework/resilience.md) 참고
- 사내 인증 기관의 인증서를 신뢰하게 하려면, JVM의 trust store에 루트 인증서를 추가하거나 SSLContext를 직접 구성함
- 인증서의 종류는 [Reverse Proxy와 HTTPS](/infra/reverse-proxy.md) 참고

<br>

### 응답을 무한정 받지 않기
- 응답 타임아웃(Read Timeout)은 "한 번 읽을 때 기다리는 시간"이라서, 상대가 아주 조금씩 계속 보내면 타임아웃 없이 오래 붙잡힐 수 있음
  - 1바이트씩 느리게 보내 연결을 오래 점유하는 공격을 Slowloris(요청 측)나 Slow Read류 공격이라고 함
- 응답 본문 크기에 제한이 없으면, 매우 큰 응답 하나로 메모리가 가득 찰 수 있음
- 대응 방법
  - 응답 전체에 대한 시간 제한(Deadline)을 따로 두고, 넘으면 연결을 끊고 타임아웃으로 처리함
  - 본문을 스트림으로 읽으면서 최대 크기를 넘으면 더 받지 않고 연결을 끊음
  - 남은 본문을 끝까지 읽어야 연결을 재사용할 수 있으므로, 중간에 그만둘 때는 연결을 풀에 돌려주지 않고 버려야 함
```
long deadline = startedAt + totalTimeout.toNanos();
ByteArrayOutputStream out = new ByteArrayOutputStream();
byte[] buffer = new byte[8192];
int read;
while ((read = body.read(buffer)) != -1) {
    if (System.nanoTime() - deadline > 0) {
        abort(body);
        throw new SocketTimeoutException("response not completed within " + totalTimeout);
    }
    int room = maxResponseBytes - out.size();
    if (read > room) {
        out.write(buffer, 0, room);
        abort(body);
        break;
    }
    out.write(buffer, 0, read);
}
```
- 사용자가 지정한 주소로 요청을 보내는 기능(Webhook, URL 미리보기 등)에서는 상대 서버를 신뢰할 수 없으므로 특히 필요함

<br>

#### 참고
- RFC 9112 <HTTP/1.1 - Message Body Length> - https://www.rfc-editor.org/rfc/rfc9112#name-message-body-length
- RFC 9110 <HTTP Semantics - User-Agent> - https://www.rfc-editor.org/rfc/rfc9110#name-user-agent
- RFC 8305 <Happy Eyeballs Version 2> - https://www.rfc-editor.org/rfc/rfc8305
- PortSwigger Web Security Academy <HTTP request smuggling> - https://portswigger.net/web-security/request-smuggling

#### 배워가는 것들
- 같은 요청이라도 본문을 chunked로 보내느냐 Content-Length로 보내느냐에 따라 서버의 반응이 달라질 수 있다는 것을 알게 되었다. 다른 Client와 결과가 다르면 요청 원문부터 비교해봐야 한다.
- 라이브러리의 기본 User-Agent가 버전 정보를 노출하고 차단의 원인이 될 수 있다는 점이 인상적이었다.
- 여러 주소로 재시도하는 과정에서 진짜 실패 원인이 가려질 수 있다는 것을 배웠다. 오류를 분류할 때는 마지막 예외만 보지 말고 그 과정을 함께 봐야 한다.
