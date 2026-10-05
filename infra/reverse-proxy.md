# Reverse Proxy와 HTTPS

### Forward Proxy와 Reverse Proxy
- Proxy는 Client와 Server 사이에서 요청을 대신 전달해주는 중계 서버
- Forward Proxy
  - Client 쪽에 위치하며, Client를 대신하여 외부 Server에 요청함
  - 사내망에서 외부 접속을 통제하거나 캐시하는 용도로 사용함
- Reverse Proxy
  - Server 쪽에 위치하며, 외부 요청을 받아 내부의 애플리케이션 Server로 전달함
  - Client는 실제 애플리케이션 Server의 존재를 알지 못함
- Reverse Proxy가 담당하는 일
  - TLS 종료(TLS Termination): HTTPS 암호화를 앞단에서 처리하고, 내부로는 HTTP로 전달함. 애플리케이션은 인증서를 다루지 않아도 됨
  - 경로나 헤더에 따른 라우팅: /api는 API Server로, 나머지는 웹 Server로 보내는 식
  - 로드 밸런싱: 여러 애플리케이션 인스턴스에 요청을 나눠줌
  - 응답 압축, 보안 헤더 추가, 정적 파일 제공
- 웹 서버와 WAS의 구분은 [Spring Boot](/spring-framework/spring-boot.md) 참고

<br>

### HTTPS와 인증서
- HTTPS는 HTTP를 TLS로 암호화한 것으로써, 통신 내용의 도청과 변조를 막고 Server가 진짜인지 확인해줌
  - 관련 내용은 [HTTP의 이해](/http/understanding-http.md) 참고
- Server의 신원은 인증 기관(CA, Certificate Authority)이 서명한 인증서로 증명함
  - 브라우저와 OS는 신뢰하는 루트 인증 기관 목록을 가지고 있으며, 그 기관이 서명한 인증서만 경고 없이 받아들임
- 인증서를 얻는 방법
  - 공인 인증 기관: Let's Encrypt처럼 무료로 자동 발급해주는 곳도 있음. 도메인이 인터넷에서 접근 가능해야 함
  - 회사 인증서: 회사가 구매하거나 사내 인증 기관이 발급한 인증서
  - 내부 인증 기관(Private CA): 직접 만든 루트 인증서로 발급함. 사내 주소에 사용할 수 있으나, 루트 인증서를 사용자 PC마다 신뢰 목록에 등록해야 경고가 사라짐
- ACME(Automatic Certificate Management Environment)
  - 인증서 발급과 갱신을 자동화하는 프로토콜이며, Let's Encrypt가 사용함
  - 도메인 소유를 확인하기 위해 80 또는 443 포트로 확인 요청을 보내므로, 해당 포트가 인터넷에서 접근 가능해야 함
  - Let's Encrypt 인증서는 유효 기간이 짧으므로(90일 이하), 자동 갱신이 필수임

<br>

### Nginx와 Caddy
- Nginx
  - 가장 널리 쓰이는 웹 서버이자 Reverse Proxy
  - 설정의 자유도가 높고 자료가 많으나, 인증서 발급·갱신은 Certbot 등을 별도로 구성해야 함
- Caddy
  - Go로 작성된 웹 서버로써, HTTPS를 기본값으로 동작함(Automatic HTTPS)
  - 사이트 주소만 적으면 인증서 발급·갱신과 HTTP에서 HTTPS로의 이동을 자동으로 처리함
  - tls internal을 지정하면 Caddy가 자체 내부 인증 기관을 만들어 인증서를 발급함
  - 설정 파일(Caddyfile)이 짧고 읽기 쉬움
```
Caddyfile

{$SITE_ADDRESS} {
    tls {$TLS_MODE}

    encode zstd gzip

    header {
        X-Content-Type-Options nosniff
        Referrer-Policy strict-origin-when-cross-origin
        X-Frame-Options DENY
        -Server
    }

    @api_token {
        header Authorization "Bearer snt_*"
        path /api/runs /api/runs/*
    }
    handle @api_token {
        reverse_proxy sentinel-api:8080
    }

    handle {
        reverse_proxy sentinel-web:3000
    }
}
```
- Caddyfile 주요 문법
  - {$이름}, {$이름:기본값}: 환경변수 참조
  - @이름 { ... }: 조건(Matcher) 정의. 헤더, 경로, 호스트 등을 조건으로 사용할 수 있음
  - handle: 조건에 맞는 요청을 처리하며, 여러 handle 중 처음 맞는 하나만 실행됨
  - reverse_proxy: 지정한 주소로 요청을 전달함
  - header: 응답 헤더 추가, 이름 앞에 -를 붙이면 삭제
- 보안 헤더의 의미는 [웹 취약점과 방어](/spring-security/web-vulnerability.md) 참고

<br>

### Reverse Proxy 뒤의 애플리케이션이 알아야 할 것
- 애플리케이션이 받는 요청은 Proxy가 보낸 것이므로, 원래 Client의 정보가 바뀌어 보일 수 있음
  - Client IP: Proxy의 IP로 보임. 원래 IP는 X-Forwarded-For 헤더로 전달됨
  - 프로토콜: Proxy와 애플리케이션 사이가 HTTP라면, 원래 HTTPS였는지는 X-Forwarded-Proto 헤더로 알 수 있음
  - Host: Proxy가 Host 헤더를 바꿔서 전달하면, 애플리케이션이 만드는 절대 URL이나 출처(Origin) 비교가 어긋남
- Spring Boot에서는 server.forward-headers-strategy=native 또는 framework로 X-Forwarded-* 헤더를 반영할 수 있음
  - 이 헤더는 Client가 임의로 넣을 수도 있으므로, 신뢰할 수 있는 Proxy를 거친 요청에서만 반영해야 함
- 애플리케이션 포트를 127.0.0.1에만 공개하면, 네트워크에서는 Proxy를 거쳐서만 접근할 수 있으므로 평문 HTTP로 비밀번호나 토큰이 오가는 경로를 막을 수 있음
  - 관련 설정은 [Docker](/infra/docker.md) 참고

<br>

#### 참고
- Caddy Documentation <Automatic HTTPS> - https://caddyserver.com/docs/automatic-https
- Caddy Documentation <Caddyfile Concepts> - https://caddyserver.com/docs/caddyfile/concepts
- Let's Encrypt <How It Works> - https://letsencrypt.org/how-it-works/
- Spring Boot Reference Documentation <Running Behind a Front-end Proxy Server> - https://docs.spring.io/spring-boot/how-to/webserver.html#howto.webserver.use-behind-a-proxy-server

#### 배워가는 것들
- HTTPS 처리를 애플리케이션이 아니라 앞단의 Reverse Proxy에 맡기면, 애플리케이션은 인증서를 몰라도 된다는 것을 알게 되었다.
- 인증서의 종류에 따라 사용자 PC에 루트 인증서를 등록해야 하는지가 달라진다는 점이 인상적이었다. 사내 도구라도 어떤 인증서를 쓸지는 배포 전에 정해야 한다.
- Proxy 뒤에 있는 애플리케이션은 Client IP나 Host가 바뀌어 보일 수 있다는 것을 배웠다. 출처 확인이나 IP 기반 제한을 구현할 때는 요청이 어떤 경로로 들어오는지 먼저 확인해야겠다.
