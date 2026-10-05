# Spring AI

### Spring AI
- Spring 애플리케이션에 LLM(대규모 언어 모델) 기능을 통합하기 위한 Spring 프로젝트
  - Anthropic, OpenAI, Google, Amazon Bedrock, Mistral AI, Ollama 등 여러 모델 제공자를 같은 API로 다루므로, 의존성과 설정만 바꿔 모델을 교체할 수 있음
  - Python 없이 Java와 Spring만으로 챗봇, 요약, 문서 검색 기반 답변(RAG), 에이전트 기능을 구현할 수 있음
- 2025년 5월 1.0이 출시되었고, 2026년 6월 2.0이 출시됨
  - 2.0은 Spring Boot 4.x와 Spring Framework 7을 필수로 요구하며, Jackson 3와 JSpecify Null 안전성이 적용됨
  - 도구 호출이 Advisor로 정리되었고, MCP 서버를 애노테이션 하나로 만들 수 있게 됨
- 모델 제공자별 Starter를 추가하고 API 키를 설정함
  - API 키는 코드나 설정 파일에 직접 쓰지 않고 환경 변수로 주입함
```
build.gradle

dependencies {
    implementation 'org.springframework.ai:spring-ai-starter-model-anthropic'
}
```
```
application.yml

spring:
  ai:
    anthropic:
      api-key: ${ANTHROPIC_API_KEY}
```

<br>

### ChatClient
- 모델과 대화하는 핵심 API로, 메서드 체인 형태의 Fluent API를 제공함
  - Spring Boot가 자동 설정한 ChatClient.Builder를 주입받아 만듦
  - defaultSystem으로 모든 요청에 적용할 역할과 규칙(시스템 프롬프트)을 지정함
- prompt()로 요청을 시작하고, user()로 사용자의 메시지를 넣음
  - call(): 응답이 모두 생성될 때까지 기다림
  - stream(): 생성되는 대로 조각을 받는 Flux를 반환함. SSE로 브라우저에 바로 전달할 수 있음
  - content(): 응답을 문자열로 받음
  - entity(타입): 응답을 지정한 타입(Record 등)으로 변환함(Structured Output)
```
@RestController
public class ChatController {

    private final ChatClient chatClient;

    public ChatController(ChatClient.Builder builder) {
        this.chatClient = builder
                .defaultSystem("당신은 쇼핑몰의 상품 안내를 돕는 상담원입니다.")
                .build();
    }

    @GetMapping("/chat")
    public String chat(@RequestParam String message) {
        return chatClient.prompt()
                .user(message)
                .call()
                .content();
    }

    @GetMapping(value = "/chat/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> stream(@RequestParam String message) {
        return chatClient.prompt()
                .user(message)
                .stream()
                .content();
    }

}
```
```
public record ProductRecommendation(String name, String reason) {
}

List<ProductRecommendation> recommendations = chatClient.prompt()
        .user("20대에게 어울리는 선물 3개를 추천해줘")
        .call()
        .entity(new ParameterizedTypeReference<List<ProductRecommendation>>() {});
```

<br>

### 도구 호출(Tool Calling)
- 모델이 스스로 판단하여 정의해둔 메서드를 호출하고, 그 결과를 바탕으로 답변을 이어가게 하는 기능
  - 주문 조회, 상품 검색처럼 모델이 모르는 최신 데이터나 우리 서비스의 데이터를 활용할 수 있음
- 메서드에 @Tool로 설명을, 매개변수에 @ToolParam으로 설명을 붙이며, 모델은 이 설명을 보고 언제 어떤 인수로 호출할지 결정함
- Spring AI 2.0에서는 도구 실행이 ChatClient에 자동 등록되는 ToolCallingAdvisor로 처리됨
- 도구 메서드는 서버에서 실행되므로, 일반 API처럼 현재 사용자의 권한을 확인해야 함
  - 모델이 만든 인수는 사용자 입력과 마찬가지로 신뢰할 수 없는 값으로 다룸
```
@Component
public class OrderTools {

    @Tool(description = "주문 번호로 주문 상태를 조회합니다")
    public OrderStatusResponse findOrderStatus(@ToolParam(description = "주문 번호") String orderNumber) {
        return orderService.getStatus(orderNumber);
    }

}


String answer = chatClient.prompt()
        .user(message)
        .tools(orderTools)
        .call()
        .content();
```

<br>

### Advisor
- ChatClient의 요청과 응답 사이에 끼어들어 공통 처리를 추가하는 구성 요소로, Servlet Filter나 AOP와 비슷한 역할을 함
- 대화 기록(Chat Memory)
  - 모델은 이전 대화를 기억하지 못하므로, 매 요청마다 이전 대화를 함께 보내야 함
  - MessageChatMemoryAdvisor가 대화 ID별로 기록을 저장하고 요청에 덧붙여줌. 저장소는 메모리, JDBC 등을 선택함
- RAG(Retrieval-Augmented Generation)
  - 질문과 관련된 문서를 먼저 검색하여 프롬프트에 함께 넣고, 그 내용을 근거로 답하게 하는 방식
  - 문서를 임베딩(Embedding)으로 바꿔 Vector Store(PostgreSQL의 pgvector 등)에 저장해두고, QuestionAnswerAdvisor가 검색과 프롬프트 구성을 처리함
  - 사내 문서, 상품 설명, FAQ처럼 모델이 학습하지 않은 내용으로 답변해야 할 때 사용함
- Spring AI 2.0에는 구조화된 출력이 형식에 맞지 않으면 자동으로 다시 요청하는 StructuredOutputValidationAdvisor 등이 추가됨
```
ChatClient chatClient = builder
        .defaultAdvisors(
                MessageChatMemoryAdvisor.builder(chatMemory).build(),
                QuestionAnswerAdvisor.builder(vectorStore).build())
        .build();

chatClient.prompt()
        .user(message)
        .advisors(advisor -> advisor.param(ChatMemory.CONVERSATION_ID, conversationId))
        .call()
        .content();
```

<br>

### MCP 서버 만들기
- MCP(Model Context Protocol)는 AI 애플리케이션이 외부 도구와 데이터에 접근하는 방식을 표준화한 프로토콜
  - MCP 서버로 공개한 기능은 Claude, IDE의 AI 에이전트 등 MCP를 지원하는 여러 AI 클라이언트에서 바로 사용할 수 있음
- Spring AI 2.0부터 MCP가 핵심 기능이 되어, 메서드에 @McpTool을 선언하는 것만으로 기존 Spring 서비스를 MCP 도구로 공개할 수 있음
  - @McpResource, @McpPrompt로 데이터와 프롬프트도 공개할 수 있음
  - 기본 전송 방식은 Streamable HTTP임
```
build.gradle

dependencies {
    implementation 'org.springframework.ai:spring-ai-starter-mcp-server-webmvc'
}
```
```
@Component
public class InventoryMcpTools {

    @McpTool(name = "check-stock", description = "상품 코드로 현재 재고 수량을 조회합니다")
    public StockResponse checkStock(@McpToolParam(description = "상품 코드") String productCode) {
        return inventoryService.getStock(productCode);
    }

}
```
- MCP 서버는 외부 AI가 우리 시스템을 호출하는 통로가 되므로, 인증을 적용하고 공개할 기능을 조회 위주로 최소한으로 정해야 함

<br>

### 운영할 때 고려할 것
- 비용: 입력과 출력의 토큰 수에 따라 비용이 발생하므로, 요청 횟수 제한과 사용량 모니터링이 필요함
  - Spring AI는 모델 호출과 토큰 사용량을 Micrometer Observation으로 기록하므로, 지표와 추적에서 확인할 수 있음. 관련 내용은 [관측 가능성](/spring-framework/observability.md) 참고
- 응답 시간: 모델 응답은 수 초에서 수십 초가 걸릴 수 있으므로, 타임아웃과 재시도 정책을 정하고 사용자에게는 스트리밍으로 응답함. 관련 내용은 [타임아웃과 재시도](/spring-framework/resilience.md) 참고
- 프롬프트 인젝션: 사용자 입력이나 RAG로 가져온 문서 속 지시 때문에 모델이 의도와 다르게 동작할 수 있으므로, 도구와 권한을 최소한으로 제한함
- 개인정보: 모델 제공자에게 전송되는 데이터에 개인정보와 비밀 값이 포함되지 않도록 걸러냄
- 테스트: 같은 입력에도 응답이 달라질 수 있으므로, 도구 호출과 Advisor 같은 결정적인 부분을 분리하여 테스트하고, 모델 응답은 평가(Evaluation) 기준을 따로 둠

<br>

#### 참고
- Spring AI Reference Documentation - https://docs.spring.io/spring-ai/reference/
- Spring 공식블로그 <Spring AI 2.0.0 GA Available Now> - https://spring.io/blog/2026/06/12/spring-ai-2-0-0-GA-available-now/
- Spring AI Reference Documentation <ChatClient API> - https://docs.spring.io/spring-ai/reference/api/chatclient.html
- Model Context Protocol 공식문서 - https://modelcontextprotocol.io

#### 배워가는 것들
- LLM 기능도 ChatClient라는 Fluent API와 Advisor라는 공통 처리 구조로 다루니, 기존의 RestClient와 Filter 개념과 크게 다르지 않다는 것을 알게 되었다.
- 도구 호출로 모델이 우리 서비스의 메서드를 호출할 수 있다는 점이 강력한 만큼, 그 메서드에도 일반 API와 똑같이 권한 확인이 필요하다는 점을 기억해야겠다.
- MCP 서버를 애노테이션 하나로 만들 수 있게 되면서, 기존 Spring 서비스를 여러 AI 클라이언트가 사용할 수 있는 도구로 공개하는 일이 쉬워졌다.
