# Polling과 Webhook

### 변경 사항을 알아내는 두 가지 방향
- HTTP는 Client가 요청해야 Server가 응답하는 구조이므로, Server 쪽에서 생긴 변화를 상대에게 알리려면 방법이 필요함
  - Polling: 알고 싶은 쪽이 주기적으로 물어봄(Pull)
  - Webhook: 변화가 생긴 쪽이 상대의 주소로 직접 알려줌(Push)
- 브라우저와 Server 사이에서 실시간으로 주고받아야 한다면 SSE(Server-Sent Events)나 WebSocket을 사용하기도 함

<br>

### Polling
- 일정한 간격으로 상태를 반복 조회하는 방식
  - 구현이 단순하고, 기존 조회 API를 그대로 사용할 수 있음
  - 변화가 없어도 계속 요청하므로 낭비가 생기고, 간격만큼 늦게 알게 됨
- Long Polling: Server가 변화가 생길 때까지 응답을 미루었다가 응답하는 방식. 요청 수는 줄지만 연결을 오래 유지해야 함
- 오래 걸리는 작업의 진행 상황 확인
  1. Client가 작업을 요청하면, Server는 작업을 접수만 하고 202 Accepted와 작업 ID(또는 Location 헤더)를 응답함
  2. Client는 작업 조회 API를 일정 간격으로 호출하여 진행률과 완료 여부를 확인함
  3. 완료되면 조회를 멈춤
```
POST /api/runs            → 202 Accepted, {"id": 42, "status": "RUNNING"}
GET  /api/runs/42         → 200 OK, {"status": "RUNNING", "plannedCount": 12, "finished": 5}
GET  /api/runs/42         → 200 OK, {"status": "COMPLETED", ...}
```
- Polling 간격을 정할 때 고려할 점
  - 진행 중인 작업이 없으면 조회를 멈춰서 불필요한 요청을 줄임
  - 화면이 보이지 않는(비활성 탭) 동안에는 간격을 늘리거나 멈춤
- 202 Accepted는 [HTTP 상태코드](/http/http-status-code.md), 백그라운드 실행은 [가상 스레드와 동시성 제어](/java/virtual-thread.md) 참고
- Server 내부에서도 Polling을 활용할 수 있으며, DB에 저장된 실행 예정 시각을 주기적으로 확인하는 스케줄러가 그 예임. 관련 내용은 [스케줄링](/spring-framework/scheduling.md) 참고

<br>

### Webhook
- 특정 이벤트가 발생했을 때, 미리 등록해둔 URL로 HTTP 요청(주로 POST)을 보내 알리는 방식
  - "역방향 API" 또는 "HTTP Callback"이라고도 부름
  - 받는 쪽은 Polling 없이 이벤트가 생긴 즉시 알 수 있음
- 사용 예
  - GitHub: Push, Pull Request 이벤트를 CI 서버로 전달함
  - 결제 서비스: 결제 완료, 환불 결과를 쇼핑몰 서버로 전달함
  - 메신저 수신 Webhook(Incoming Webhook): 외부 시스템이 Slack, Teams 채널에 메시지를 보낼 수 있도록 발급해주는 주소
```
POST https://hooks.slack.com/services/T000/B000/XXXX
Content-Type: application/json

{"text": "[Sentinel] 결제 스위트 - 12건 중 2건 실패"}
```
- 수신 Webhook의 메시지 형식은 서비스마다 다름
  - Slack: text 필드 또는 Block Kit(blocks)
  - Teams: 새 워크플로 기반 Webhook은 Adaptive Card 형식을 요구함
  - 여러 서비스를 지원해야 한다면, 메시지 내용(제목, 본문, 링크)과 형식 변환을 분리해두면 좋음

<br>

### Webhook을 보내는 쪽의 유의사항
- Webhook 주소 자체가 자격증명임
  - 주소만 알면 누구나 그 채널에 메시지를 보낼 수 있으므로, 환경변수로 주입하거나 암호화하여 저장함
  - 로그, 예외 메시지, 조회 API 응답에 주소가 남지 않도록 해야 함. HTTP Client의 예외 메시지에는 요청 URL이 포함되는 경우가 많음
- 타임아웃을 짧게 두어, 상대가 느려도 본래 작업이 지연되지 않도록 함
- 발송 실패가 본래 작업을 실패시키지 않도록, 예외를 잡아서 기록만 남김
- 사용자가 Webhook 주소를 등록할 수 있다면, 서버가 내부 주소로 요청을 보내는 통로가 될 수 있으므로 SSRF 방어가 필요함. 관련 내용은 [웹 취약점과 방어](/spring-security/web-vulnerability.md) 참고
- 발송 이력(시각, 대상, 결과 상태 코드)을 남겨두면 "알림이 오지 않았다"는 문의에 바로 답할 수 있음
  - 응답 상태 코드별로 원인을 짐작할 수 있음: 404, 410은 주소가 삭제·변경됨, 401, 403은 권한 거부, 429는 발송 한도 초과, 5xx는 상대 서비스 오류
- 등록 직후 연결 확인 메시지를 보내주면, 주소 오타나 권한 문제를 실제 이벤트가 생기기 전에 발견할 수 있음

<br>

### Webhook을 받는 쪽의 유의사항
- 주소가 공개되어 있으면 누구나 가짜 이벤트를 보낼 수 있으므로, 서명을 검증해야 함
  - 보내는 쪽이 공유 비밀키로 본문의 HMAC 서명을 만들어 헤더에 넣고, 받는 쪽이 같은 방식으로 계산하여 비교함
  - 재전송 공격을 막기 위해 서명에 타임스탬프를 포함하기도 함
- 같은 이벤트가 여러 번 올 수 있으므로(재시도), 이벤트 ID로 중복을 걸러 멱등하게 처리함
- 빠르게 2xx로 응답하고, 오래 걸리는 처리는 비동기로 넘김. 응답이 늦으면 보내는 쪽이 실패로 보고 재시도함

<br>

### Heartbeat
- 자신이 정상적으로 동작하고 있음을 알리기 위해, 일정한 간격으로 외부에 보내는 신호
- 장애가 발생하면 장애를 알리는 기능도 함께 멈추므로, "아무 알림도 오지 않는 상태"가 정상처럼 보이는 문제가 있음
  - 이를 해결하기 위해, 신호가 정해진 시간 동안 오지 않으면 외부 감시 서비스가 대신 알려주는 방식을 사용함(Dead Man's Switch)
  - healthchecks.io, Uptime Kuma(Push 방식), Better Stack 등은 발급받은 주소로 GET 요청을 보내는 것만으로 동작함
- 신호를 보내는 위치가 중요함
  - 별도의 Thread에서 신호만 보내면, 본래 작업이 멈춰도 신호는 계속 가므로 장애를 감지하지 못함
  - 본래 작업(예: 스케줄 폴링)의 흐름 안에서, DB 조회 등 핵심 의존성이 정상일 때만 보내야 의미가 있음
  - 오래 걸리는 작업 뒤에 보내면 그동안 신호가 끊겨 오탐이 생기므로, 작업을 시작하기 전에 보냄
- 외부에서 상태를 묻는 방식(health Endpoint)은 [Actuator](/spring-framework/actuator.md) 참고

<br>

#### 참고
- Slack API <Sending messages using incoming webhooks> - https://docs.slack.dev/messaging/sending-messages-using-incoming-webhooks
- Microsoft Learn <Create incoming webhooks with Workflows for Microsoft Teams> - https://support.microsoft.com/en-us/office/create-incoming-webhooks-with-workflows-for-microsoft-teams-8ae491c7-0394-4861-ba59-055e33f75498
- GitHub Docs <Validating webhook deliveries> - https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries
- Healthchecks.io Documentation - https://healthchecks.io/docs/

#### 배워가는 것들
- Polling과 Webhook은 누가 먼저 말을 거느냐의 차이라는 것을 정리할 수 있었다. 간단한 진행 상황 확인은 Polling으로 충분하고, 즉시 알려야 하는 이벤트는 Webhook이 적합하다.
- Webhook 주소가 그 자체로 자격증명이라 로그와 예외 메시지까지 신경 써야 한다는 점이 인상적이었다.
- 장애 알림 시스템이 멈추면 아무 알림도 오지 않아 정상처럼 보인다는 문제를 알게 되었다. 스스로 알릴 수 없는 상황은 외부의 감시에 맡겨야 한다.
