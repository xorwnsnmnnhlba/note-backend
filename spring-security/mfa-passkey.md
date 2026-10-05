# 다중 인증과 패스키

### Spring Security 7의 주요 변화
- Spring Boot 4.x와 함께 Spring Security 7이 사용됨
- 주요 변화
  - 람다 DSL만 지원: http.csrf().disable().and()처럼 and()로 이어 쓰던 방식이 제거되고, 람다로 설정하는 방식만 남음
  - Authorization Server 통합: 별도 프로젝트였던 Spring Authorization Server가 Spring Security에 포함됨
  - 다중 인증(MFA) 지원: 여러 인증 수단을 조합하여 요구하는 기능이 기본으로 제공됨
  - Null 안전성: JSpecify 애노테이션이 적용됨. 관련 내용은 [Null 안전성(JSpecify)](/java/null-safety.md) 참고
- 인증의 기본 구조(SecurityFilterChain, SecurityContextHolder)는 [Authentication](/spring-security/authentication.md) 참고
```
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
            .csrf(csrf -> csrf.ignoringRequestMatchers("/api/webhooks/**"))
            .authorizeHttpRequests(authorize -> authorize
                    .requestMatchers("/admin/**").hasRole("ADMIN")
                    .anyRequest().authenticated())
            .formLogin(Customizer.withDefaults());
    return http.build();
}
```

<br>

### 다중 인증(MFA, Multi-Factor Authentication)
- 서로 다른 종류의 인증 수단을 두 가지 이상 요구하여, 비밀번호가 유출되어도 계정을 지킬 수 있게 하는 방식
  - 아는 것(비밀번호), 가진 것(휴대폰, 보안 키), 고유한 것(지문, 얼굴)
- Spring Security 7에서는 인증할 때마다 어떤 수단으로 인증했는지가 권한(FactorGrantedAuthority)으로 기록됨
  - PASSWORD_AUTHORITY: 아이디와 비밀번호
  - OTT_AUTHORITY: 일회용 토큰(One-Time Token). 이메일이나 문자로 받은 링크와 코드
  - WEBAUTHN_AUTHORITY: 패스키
  - X509_AUTHORITY, AUTHORIZATION_CODE_AUTHORITY: 인증서, OAuth 2.0 로그인
- @EnableMultiFactorAuthentication으로 앱 전체에 요구할 인증 수단의 조합을 지정함
  - 아래 설정은 비밀번호로 로그인한 뒤 일회용 토큰 인증까지 마쳐야 접근할 수 있게 함
```
@Configuration
@EnableMultiFactorAuthentication(authorities = {
        FactorGrantedAuthority.PASSWORD_AUTHORITY,
        FactorGrantedAuthority.OTT_AUTHORITY })
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/admin/**").hasRole("ADMIN")
                        .anyRequest().authenticated())
                .formLogin(Customizer.withDefaults())
                .oneTimeTokenLogin(Customizer.withDefaults());
        return http.build();
    }

}
```
- 경로마다 다르게 요구하려면 AuthorizationManagerFactories.multiFactor()로 만든 규칙을 access에 지정함
  - 관리자 페이지와 계정 설정 변경처럼 민감한 기능에만 추가 인증을 요구하는 방식(Step-up Authentication)으로 사용함
```
var mfa = AuthorizationManagerFactories.multiFactor()
        .requireFactors(
                FactorGrantedAuthority.PASSWORD_AUTHORITY,
                FactorGrantedAuthority.OTT_AUTHORITY)
        .build();

http.authorizeHttpRequests(authorize -> authorize
        .requestMatchers("/admin/**").access(mfa.hasRole("ADMIN"))
        .requestMatchers("/user/settings/**").access(mfa.authenticated())
        .anyRequest().authenticated());
```

<br>

### 일회용 토큰 로그인(One-Time Token Login)
- 비밀번호 대신 이메일이나 문자로 보낸 일회용 링크나 코드로 로그인하는 방식(Magic Link)
- oneTimeTokenLogin을 설정하면 토큰 생성과 검증은 Spring Security가 처리하며, 토큰을 사용자에게 전달하는 부분은 OneTimeTokenGenerationSuccessHandler로 직접 구현함
- 토큰은 짧은 시간만 유효하고 한 번 사용하면 만료되어야 하며, 기본 저장소는 메모리이므로 서버가 여러 대이면 JDBC 저장소를 사용함
- 토큰 생성과 해시 저장의 원리는 [세션 토큰과 양방향 암호화](/spring-security/token-and-encryption.md) 참고

<br>

### 패스키(Passkey)
- 비밀번호 대신 기기에 저장된 개인 키로 인증하는 방식으로, FIDO Alliance와 W3C의 WebAuthn 표준을 기반으로 함
  - 가입할 때 기기에서 공개 키와 개인 키 쌍을 만들고, 서버에는 공개 키만 저장함
  - 로그인할 때 서버가 보낸 무작위 값(Challenge)에 기기가 개인 키로 서명하고, 서버는 공개 키로 서명을 검증함
  - 사용자는 지문, 얼굴 인식, 기기 PIN으로 개인 키 사용을 승인함
- 장점
  - 서버에 비밀번호가 없으므로 유출될 비밀번호 자체가 없음
  - 서명이 사이트의 도메인(RP ID)에 묶여 있어, 비슷하게 만든 피싱 사이트에서는 사용할 수 없음
  - 여러 기기 사이에 동기화되어, 휴대폰을 바꿔도 계속 사용할 수 있음
- Spring Security는 webAuthn DSL로 패스키 등록과 로그인을 지원함
  - rpId: 패스키가 묶일 도메인
  - allowedOrigins: 인증 요청을 허용할 출처
  - 사용자 정보와 등록된 자격 증명을 저장할 PublicKeyCredentialUserEntityRepository, UserCredentialRepository가 필요하며, 운영 환경에서는 JDBC 구현체를 사용함
```
http
        .formLogin(Customizer.withDefaults())
        .webAuthn(webAuthn -> webAuthn
                .rpName("Example Shop")
                .rpId("example.com")
                .allowedOrigins("https://example.com"));
```
- 패스키를 등록한 사용자에게만 비밀번호와 패스키를 함께 요구하려면 @EnableMultiFactorAuthentication의 when 속성에 MultiFactorCondition.WEBAUTHN_REGISTERED를 지정함

<br>

### Authorization Server
- OAuth 2.0과 OpenID Connect의 인가 서버(Authorization Server)를 직접 구축하는 기능
  - 여러 서비스가 하나의 로그인을 공유하는 SSO, 외부 파트너에게 API 접근 권한을 발급하는 경우에 사용함
- 2022년부터 Spring Authorization Server라는 별도 프로젝트로 개발되다가, Spring Security 7에 포함됨
  - OAuth2 Client, Resource Server, Authorization Server를 하나의 프로젝트와 문서에서 다룰 수 있게 됨
- 직접 구축하는 것은 운영 부담이 크므로, 자체 사용자 관리가 꼭 필요한 경우가 아니라면 외부 인증 서비스(Keycloak, Auth0, Cognito 등)도 함께 검토함
- OAuth 2.0과 JWT의 기본 개념은 [Authentication](/spring-security/authentication.md), [JWT, Authority](/spring-security/jwt-authority.md) 참고

<br>

#### 참고
- Spring Security Reference Documentation <What's New in Spring Security 7.0> - https://docs.spring.io/spring-security/reference/whats-new.html
- Spring Security Reference Documentation <Multi-Factor Authentication> - https://docs.spring.io/spring-security/reference/servlet/authentication/mfa.html
- Spring Security Reference Documentation <Passkeys> - https://docs.spring.io/spring-security/reference/servlet/authentication/passkeys.html
- Spring 공식블로그 <Spring Authorization Server moving to Spring Security 7.0> - https://spring.io/blog/2025/09/11/spring-authorization-server-moving-to-spring-security-7-0
- FIDO Alliance <Passkeys> - https://fidoalliance.org/passkeys/

#### 배워가는 것들
- Spring Security 7에서 인증 수단 자체가 권한으로 기록된다는 설계가 인상 깊었다. 그래서 다중 인증도 기존의 권한 검사와 같은 방식으로 경로마다 요구할 수 있다.
- 패스키는 서버에 비밀번호가 없고 서명이 도메인에 묶여 있어서, 비밀번호 유출과 피싱을 함께 막을 수 있다는 것을 알게 되었다.
- and()로 이어 쓰던 설정 방식이 제거되고 람다 DSL만 남았다는 것을 알게 되었다. 예전 자료의 설정을 그대로 가져오면 컴파일되지 않으므로 주의해야 한다.
