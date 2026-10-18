# REST 리소스 설계

### 동작을 리소스로 바라보기
- REST에서는 URL에 동사를 넣지 않고, HTTP 메서드로 행위를 표현함
  - 기본 규칙은 [Collection Pattern](/rest-api/collection-pattern.md), [CQS가 적용된 HTTP Method에서의 CRUD 활용](/rest-api/crud-http-method-using-cqs.md) 참고
- 로그인, 초대, 비밀번호 재설정, 실행처럼 CRUD로 보이지 않는 동작도 "무엇이 만들어지거나 바뀌는가"를 기준으로 보면 리소스로 표현할 수 있음
  - 로그인: 세션을 만드는 것 → POST /api/sessions, 로그아웃은 DELETE /api/sessions/current
  - 초대: 초대 링크를 발급하는 것 → POST /api/invites, 링크 확인은 GET /api/invites/{token}
  - 초대 수락: 계정을 만드는 것 → POST /api/users (본문에 초대 토큰)
  - 비밀번호 변경: 사용자의 하위 리소스를 바꾸는 것 → PUT /api/users/me/password
  - 실행 시작: 실행 기록을 만드는 것 → POST /api/runs, 실행 중지는 상태를 바꾸는 것 → PATCH /api/runs/{id} {"status": "CANCELED"}
  - 테스트 알림 보내기: 발송 이력을 만드는 것 → POST /api/notification-deliveries
- 동사형 주소(POST /api/users/{id}/reset-password, POST /api/runs/{id}/cancel)에 비해 얻는 것
  - 같은 개념이 여러 주소로 흩어지지 않고 한 리소스 아래 모임
  - 생성 결과를 201 Created와 Location으로, 조회를 GET으로 일관되게 다룰 수 있음
- 하나의 요청이 서로 다른 리소스 두 개를 만들지 않도록 함
  - 예: 초대 수락 응답에 로그인 세션까지 섞어서 주지 않고, 계정 생성(POST /api/users)과 로그인(POST /api/sessions)을 따로 요청함
  - 두 요청을 이어서 처리하는 것은 화면(또는 BFF)의 몫임

<br>

### 관계는 중첩 경로 대신 조회 조건으로
- 모든 리소스를 최상위 컬렉션으로 두고, 부모와의 관계는 쿼리 파라미터로 좁힘
```
GET /api/suites?projectId=1
GET /api/runs?suiteId=3&environmentId=2&status=RUNNING
```
- 중첩 경로(/projects/1/suites/3/runs)보다 유리한 경우
  - 부모가 둘 이상인 리소스(실행은 스위트와 환경 모두에 속함)도 어색하지 않게 표현할 수 있음
  - 조회 조건이 늘어나도 주소 구조를 바꾸지 않아도 됨
- 존재하지 않는 부모 ID로 목록을 조회하면 404가 아니라 빈 목록(200)으로 응답함. 404는 단건 조회(/{id})에서만 사용함

<br>

### 수정·삭제할 수 있는 리소스는 조회도 가능하게
- PATCH, PUT, DELETE /api/xxx/{id}가 있다면 GET /api/xxx/{id}도 있어야 함
  - 생성 응답의 Location이 실제로 조회되는 주소가 됨
  - Client가 수정 전에 현재 상태를 확인하고, 수정 후 결과를 다시 읽을 수 있음
- 같은 경로를 서로 다른 의미로 쓰지 않음
  - 예: /api/invites/{token}이 로그인 없이 링크를 확인하는 주소라면, 관리자가 대기 중인 초대를 숫자 ID로 조회·취소하는 주소는 /api/pending-invites/{id}로 분리함
  - 인증 없이 열린 경로와 관리용 경로가 섞이면, 인증 규칙을 경로로 판단하기 어려워짐

<br>

### 컬렉션에 대한 일괄 변경
- 여러 항목을 한 번에 바꿔야 할 때, 항목마다 요청을 보내는 대신 컬렉션에 PATCH를 보낼 수 있음
  - 동사형 주소(POST /api/schedules/bulk-enable)를 만들지 않고도 표현할 수 있음
```
PATCH /api/schedules
{
  "scheduleIds": [3, 5, 8],
  "enabled": true
}
```
- 일괄 변경의 원칙
  - 하나의 트랜잭션으로 처리하여, 하나라도 없거나 실패하면 전부 변경하지 않음(All or Nothing)
  - 이미 원하는 상태인 항목은 건드리지 않음. 예: 이미 켜진 스케줄을 다시 켜면 다음 실행 시각이 다시 계산되어 곧 돌 실행을 건너뛸 수 있음
- 순서처럼 목록 전체가 하나의 값인 경우는 전체 목록을 보내고, 서버가 알고 있는 목록과 다르면 409로 거부함
  - 다른 사람이 그사이 추가·삭제한 항목을 모르고 덮어쓰는 것을 방지함

<br>

### 응답에서 다른 리소스를 가리키는 방법
- 숫자 ID만 주는 방식(suiteId), ID와 이름을 따로 주는 방식(suiteId, suiteName)이 섞여 있으면 Client가 리소스마다 다르게 처리해야 함
- {id, name} 객체로 통일하면
  - 화면이 이름을 보여주기 위해 목록을 따로 받아와 이어 붙일 필요가 없음
  - 어떤 필드가 다른 리소스를 가리키는지 형태만 보고 알 수 있음
```
{
  "id": 42,
  "status": "COMPLETED",
  "project": { "id": 1, "name": "결제" },
  "suite": { "id": 3, "name": "smoke" },
  "environment": { "id": 2, "name": "dev" }
}
```
- 요청 본문과 조회 조건에는 그대로 ID만 받음
- 이름을 채우기 위해 항목마다 조회하면 N+1 문제가 생기므로, 목록이 가리키는 리소스를 한 번에 읽어둔 뒤 채워야 함
  - 쿼리 횟수가 항목 수에 비례하지 않는지 테스트로 확인해두면 좋음

<br>

### 설계 규칙을 테스트로 고정하기
- 문서로만 남긴 규칙은 새 API를 만들 때 쉽게 잊힘
- RequestMappingHandlerMapping에서 등록된 모든 경로와 메서드를 꺼내 검사하면, 규칙을 어긴 API가 추가될 때 빌드가 실패하도록 할 수 있음
  - 경로 규칙: /api/ 아래 소문자와 하이픈만 사용
  - 수정·삭제 가능한 경로는 GET도 있는지
  - 응답 record에 suiteId, suiteName 같은 필드가 없는지
```
@Autowired
@Qualifier("requestMappingHandlerMapping")
private RequestMappingHandlerMapping mappings;

@Test
void everyApiPathIsUnderApiAndUsesLowerCaseWordsJoinedByHyphens() {
    assertThat(routes().keySet()).allSatisfy(path -> assertThat(path)
            .matches("/api(/([a-z]+(-[a-z]+)*|\\{[a-zA-Z]+}))+"));
}
```
- 라이브러리가 등록하는 경로(API 문서 등)는 검사 대상에서 제외해야 함
- 패키지 의존 방향 같은 구조 규칙은 ArchUnit이나 Spring Modulith로 검사할 수 있으며, 관련 내용은 [Spring Modulith](/domain-driven-design/spring-modulith.md) 참고

<br>

#### 참고
- Microsoft Learn <Web API design best practices> - https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design
- Google Cloud <API Design Guide - Custom Methods> - https://cloud.google.com/apis/design/custom_methods
- Zalando <RESTful API Guidelines> - https://opensource.zalando.com/restful-api-guidelines/

#### 배워가는 것들
- 동사처럼 보이는 동작도 "무엇이 만들어지는가"로 바라보면 리소스로 표현할 수 있다는 것을 알게 되었다. 초대 수락이 계정 생성이라는 관점이 인상적이었다.
- 응답에서 다른 리소스를 가리키는 형태를 하나로 통일하면 화면 코드가 단순해진다는 점을 배웠다. 일관성은 API를 쓰는 쪽의 비용을 줄여준다.
- 설계 규칙은 문서보다 테스트로 고정해야 지켜진다는 것을 알게 되었다. 새 API가 규칙을 어기면 빌드가 실패하도록 해두는 습관을 들여야겠다.
