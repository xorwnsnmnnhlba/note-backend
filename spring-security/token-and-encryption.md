# 세션 토큰과 양방향 암호화

### 불투명 토큰(Opaque Token)과 JWT
- 로그인에 성공한 사용자에게 발급하는 토큰은 크게 두 가지 방식으로 나눌 수 있음
  - 불투명 토큰: 의미 없는 무작위 문자열. 서버가 DB나 Redis에 저장해두고, 요청마다 조회하여 사용자를 확인함
  - JWT: 사용자 정보와 만료 시각을 담고 서명한 토큰. 서버가 저장하지 않고 서명만 검증함. 관련 내용은 [JWT & Authority](/spring-security/jwt-authority.md) 참고
- 불투명 토큰이 적합한 경우
  - 로그아웃, 비밀번호 변경, 사용자 삭제 시 토큰을 즉시 무효화해야 하는 경우. JWT는 만료 전까지 유효하므로 별도의 차단 목록이 필요함
  - 서버가 하나라서 요청마다 저장소를 조회하는 비용이 문제되지 않는 경우
- JWT가 적합한 경우
  - 여러 서버나 서비스가 같은 토큰을 공유 저장소 없이 검증해야 하는 경우

<br>

### 안전한 토큰 만들기
- 추측할 수 없도록 암호학적으로 안전한 난수 생성기(SecureRandom)로 충분한 길이(32바이트 = 256비트 이상)를 만듦
  - java.util.Random은 다음 값을 예측할 수 있으므로 토큰 생성에 사용하면 안 됨
- URL이나 쿠키에 그대로 넣을 수 있도록 URL-safe Base64로 인코딩함
```
private static final SecureRandom RANDOM = new SecureRandom();

static String newToken() {
    byte[] bytes = new byte[32];
    RANDOM.nextBytes(bytes);
    return Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
}
```
- 토큰 원문은 저장하지 않고, 해시 값만 저장함
  - DB가 유출되어도 해시 값으로는 로그인할 수 없음
  - 원문은 발급 응답에서 한 번만 전달하며, 이후에는 서버도 알 수 없음
- 서비스 고유의 접두어(예: GitHub의 ghp_)를 붙이면, 코드나 로그에 실수로 노출된 토큰을 비밀 탐지 도구가 찾아낼 수 있고 토큰 종류를 빠르게 구분할 수 있음

<br>

### 토큰은 SHA-256, 비밀번호는 BCrypt인 이유
- 비밀번호는 사람이 만든 값이라 경우의 수가 적음
  - 해시 값이 유출되면 자주 쓰이는 비밀번호를 대입해보는 사전 공격(Dictionary Attack)이 가능함
  - 그래서 일부러 느리게 동작하고 Salt를 포함하는 BCrypt, Argon2 같은 알고리즘을 사용함
  - 관련 내용은 [로그인 & 로그아웃, 회원가입](/spring-security/login-logout-signup.md) 참고
- 무작위로 만든 256비트 토큰은 대입해볼 후보가 사실상 무한하므로 사전 공격 대상이 아님
  - 빠른 해시 함수인 SHA-256으로 충분하며, 요청마다 해시를 계산해도 부담이 없음
  - 같은 값은 항상 같은 해시가 나오므로, 해시 값으로 DB를 바로 조회할 수 있음
```
static String hash(String token) {
    MessageDigest digest = MessageDigest.getInstance("SHA-256");
    return HexFormat.of().formatHex(digest.digest(token.getBytes(StandardCharsets.UTF_8)));
}
```

<br>

### 세션과 일회용 링크 관리
- 만료 방식
  - 절대 만료(Absolute Timeout): 로그인 시점을 기준으로 고정된 시간이 지나면 만료됨
  - 유휴 만료(Idle Timeout, Sliding): 마지막 사용 시점부터 일정 시간이 지나면 만료되며, 사용할 때마다 연장됨
- 비밀번호를 변경하거나 재설정하면 해당 사용자의 모든 세션을 종료하여, 탈취된 세션이 계속 쓰이지 않도록 함
- 초대 링크, 비밀번호 재설정 링크에 사용하는 토큰
  - 세션 토큰과 같은 방식으로 만들고, 해시로 저장함
  - 한 번 사용하면 무효화하고(일회용), 짧은 만료 시간을 둠
  - 만료, 이미 사용, 존재하지 않음을 모두 같은 응답(예: 404)으로 처리하여, 어떤 토큰이 유효했는지 알 수 없도록 함
- 자동화(CI 등)에 사용하는 API 토큰은 로그인 세션과 분리함
  - 필요한 기능만 수행할 수 있도록 권한 범위를 좁힘
  - 만료 기한이 없는 토큰은 잊힌 채 계속 유효하므로, 발급 시 만료 기한을 반드시 지정하게 함
- 남아 있는 자격증명 정리하기
  - 비밀번호 재설정은 계정 복구에 해당하므로, 세션뿐 아니라 그 사용자가 발급한 API 토큰도 폐기함. 탈취된 계정을 되찾았는데 공격자가 만든 토큰이 남아 있으면 안 됨
  - 초대를 다시 보냈다면, 수락 시 같은 이메일로 발급된 다른 초대 링크를 모두 무효화함. 예전 링크가 남아 있으면 계정 삭제 뒤 그 링크로 다시 가입할 수 있음
  - 사용자를 삭제하면 그 사용자에게 보낸 대기 중인 초대도 함께 삭제함
  - 관리자가 대기 중인 초대를 확인하고 취소할 수 있어야, 잘못 보낸 링크가 만료될 때까지 살아 있지 않음
  - 만료된 세션과 링크 행은 정리 배치로 주기적으로 삭제함. 지우지 않으면 브라우저를 닫고 돌아오지 않은 세션의 해시가 계속 쌓임
- 같은 링크를 동시에 사용하는 경합은 [동시 수정과 낙관적 잠금](/jpa/optimistic-locking.md) 참고

<br>

### 계정 존재 여부 노출 막기
- 로그인 실패 시 "없는 계정"과 "비밀번호 불일치"를 구분해서 알려주면, 공격자가 가입된 이메일 목록을 수집할 수 있음(User Enumeration)
- 응답 문구뿐 아니라 응답 시간도 같아야 함
  - 없는 계정은 BCrypt 비교를 건너뛰어 응답이 훨씬 빠르므로, 시간 차이로 계정 존재 여부가 드러남(Timing Attack)
  - 없는 계정일 때도 더미 해시와 비교를 수행하여 응답 시간을 맞춤
  - 더미 해시는 형식이 올바른 실제 BCrypt 해시여야 함. 길이나 형식이 틀린 문자열은 비교 연산 없이 즉시 실패하므로, 응답 시간 차이가 그대로 남음
  - 기동 시 무작위 값으로 해시를 하나 만들어 사용하면 형식 실수를 피할 수 있음
```
User user = users.findByEmail(email).orElse(null);
if (user == null || !passwordEncoder.matches(password, user.getPasswordHash())) {
    if (user == null) {
        passwordEncoder.matches(password, dummyHash);    // dummyHash = passwordEncoder.encode(무작위 값)
    }
    throw new InvalidCredentialsException();
}
```
- 비밀 값을 직접 비교할 때는 equals() 대신 MessageDigest.isEqual()처럼 비교 시간이 일정한 방법을 사용함

<br>

### 로그인 시도 제한(Login Throttling)
- 제한이 없으면 비밀번호를 계속 대입해보는 무차별 대입 공격(Brute Force Attack)이 가능함
- 일정 시간 안에 정해진 횟수 이상 실패하면, 일정 시간 동안 로그인을 거부함
  - 거부 응답은 429 Too Many Requests와 Retry-After 헤더로 다시 시도할 수 있는 시점을 안내함
  - 없는 계정도 같은 방식으로 세어야, 잠김 여부로 계정 존재가 드러나지 않음
- 고려해야 할 점
  - 기준이 계정이면, 다른 사람이 특정 계정을 일부러 잠글 수 있음
  - 기준이 IP면, 프록시나 BFF 뒤에 있는 서버에는 모든 요청이 같은 IP로 보여 사용자를 구분할 수 없음
  - 메모리에 보관하면 재시작 시 초기화되고 인스턴스 간에 공유되지 않으므로, 여러 인스턴스 환경에서는 Redis 같은 공유 저장소가 필요함

<br>

### AES-GCM 양방향 암호화
- 외부 API 토큰처럼 저장했다가 다시 꺼내서 사용해야 하는 값은 해시가 아닌 양방향 암호화로 저장해야 함
- AES(Advanced Encryption Standard): 대칭키 블록 암호. 키 길이는 128, 192, 256비트 중 선택함
- 운영 모드(Mode of Operation): 블록 암호로 긴 데이터를 암호화하는 방식
  - ECB: 같은 평문 블록이 같은 암호문이 되어 패턴이 드러나므로 사용하면 안 됨
  - CBC: 암호화만 제공하며, 암호문이 변조되었는지는 알 수 없음
  - GCM(Galois/Counter Mode): 암호화와 함께 인증 태그(Authentication Tag)를 만들어, 복호화 시 변조 여부를 검증함(AEAD, Authenticated Encryption with Associated Data)
- GCM 사용 시 지켜야 할 점
  - IV(Nonce)는 12바이트를 사용하며, 같은 키로 같은 IV를 두 번 사용하면 평문이 드러날 수 있으므로 암호화할 때마다 새로 생성함
  - IV는 비밀이 아니므로 암호문 앞에 붙여서 함께 저장함
  - 인증 태그는 128비트를 사용함
- 저장 형식에 버전 접두어(v1:)를 붙여두면, 나중에 키나 알고리즘을 바꿀 때 기존 값과 새 값을 구분할 수 있음
```
public String encrypt(String plainText) {
    byte[] iv = new byte[12];
    random.nextBytes(iv);
    Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
    cipher.init(Cipher.ENCRYPT_MODE, key, new GCMParameterSpec(128, iv));
    byte[] encrypted = cipher.doFinal(plainText.getBytes(StandardCharsets.UTF_8));

    byte[] combined = new byte[iv.length + encrypted.length];
    System.arraycopy(iv, 0, combined, 0, iv.length);
    System.arraycopy(encrypted, 0, combined, iv.length, encrypted.length);
    return "v1:" + Base64.getEncoder().encodeToString(combined);
}
```
- 키 관리
  - 키는 DB와 분리하여 환경변수 등으로 애플리케이션에만 둠. DB만 유출되어서는 복호화할 수 없어야 함
  - 키가 설정되지 않았다면 평문으로 저장하지 말고, 저장 자체를 거부해야 함
  - 키를 바꾸면 기존 값을 복호화할 수 없으므로, 아래의 키 교체 절차가 필요함
  - 키 생성 예: openssl rand -base64 32
- 암호화한 값은 조회 API와 로그에 노출하지 않고, 이름이나 설정 여부만 보여줌
- 저장된 비밀 값을 실행 기록 등에서 가릴(Masking) 때는 긴 값부터 가려야 함
  - 한 비밀 값이 다른 비밀 값의 일부라면, 짧은 값을 먼저 가렸을 때 긴 값의 나머지 부분이 그대로 드러남
- 암호화 위치(애플리케이션, DBMS)에 따른 방식 비교는 [애플리케이션 수준의 보안](/spring-security/application-level-security.md) 참고

<br>

### 암호화 키 교체(Key Rotation)
- 키가 유출되었다고 의심되거나, 키를 알고 있던 사람이 바뀌었을 때 새 키로 바꾸는 작업
  - 키만 바꾸면 기존 암호문을 풀 수 없으므로, 기존 값을 이전 키로 풀어서 새 키로 다시 암호화해야 함
- 구현 방식 예시
  - 현재 키와 이전 키를 함께 설정하고 애플리케이션을 기동함
  - 복호화할 때는 현재 키로 먼저 시도하고, 실패하면 이전 키로 시도함
  - 이전 키로 풀린 값은 현재 키로 다시 암호화하여 저장함
```
SENTINEL_SECRET_KEY=<새 키>
SENTINEL_SECRET_KEY_PREVIOUS=<지금까지의 키>
```
- 어떤 키로 암호화했는지 구분하는 방법
  - 저장 형식에 키 ID를 넣는 방식(v1:key-2:...): 어떤 키를 써야 하는지 바로 알 수 있음
  - 키를 차례로 시도하는 방식: AES-GCM은 키가 다르면 인증 태그 검증이 확실히 실패하므로, 잘못된 키로 엉뚱한 평문이 나올 걱정 없이 차례로 시도할 수 있음. 기존 저장 형식을 바꾸지 않아도 됨
- 재암호화를 언제 할 것인가
  - 별도의 관리 API나 명령으로 실행할 수도 있지만, 키가 환경변수로만 주어진다면 기동 시점에 처리하는 것이 간단함
  - 요청을 받거나 스케줄을 시작하기 전에 끝내야, 화면에서의 수정과 겹쳐 값이 덮어쓰이지 않음
  - Spring에서는 모든 싱글턴 Bean이 생성된 직후에 호출되는 SmartInitializingSingleton을 사용할 수 있음. 관련 내용은 [Spring Boot](/spring-framework/spring-boot.md) 참고
```
@Component
class SecretKeyRotation implements SmartInitializingSingleton {

    @Override
    public void afterSingletonsInstantiated() {
        Result result = rotate();    // 이전 키로 풀리는 값을 현재 키로 다시 암호화
        log.info("Re-encrypted {} secret value(s)", result.reEncrypted());
    }

}
```
- 어느 키로도 풀 수 없는 값은 지우지 않고 위치만 경고로 남김
  - 이전 키를 잘못 넣은 경우에도, 올바른 키로 다시 기동하면 복구할 수 있음
- 교체가 끝난 뒤의 처리
  - 이전 키 설정은 제거하되, 이전 키 자체는 교체 전에 만든 DB 백업이 모두 사라질 때까지 보관해야 함. 백업으로 복원하면 그 안의 값은 이전 키로 암호화되어 있음
  - 유출 때문에 교체했다면, 키뿐 아니라 그 키로 풀 수 있었던 실제 비밀 값(외부 API 토큰, Webhook 주소)도 발급처에서 바꿔야 함
- 대규모 시스템에서는 데이터를 데이터 키로 암호화하고, 데이터 키를 다시 마스터 키로 암호화하는 봉투 암호화(Envelope Encryption)와 AWS KMS 같은 키 관리 서비스를 사용하여, 마스터 키 교체 시 데이터 키만 다시 암호화하도록 함

<br>

#### 참고
- OWASP Cheat Sheet Series <Session Management> - https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html
- OWASP Cheat Sheet Series <Authentication> - https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- OWASP Cheat Sheet Series <Cryptographic Storage> - https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html
- NIST SP 800-38D <Galois/Counter Mode> - https://csrc.nist.gov/pubs/sp/800/38/d/final

#### 배워가는 것들
- 토큰과 비밀번호의 해시 방식이 다른 이유를 이해할 수 있었다. 값의 경우의 수가 충분히 크다면 느린 해시가 필요 없고, 사람이 만든 값이라면 반드시 느린 해시를 써야 한다.
- 로그인 실패 응답은 문구뿐 아니라 응답 시간까지 같아야 한다는 점이 인상적이었다. 보안은 눈에 보이는 결과만이 아니라 부수적인 신호까지 고려해야 한다.
- 양방향 암호화는 알고리즘보다 IV와 키를 어떻게 다루느냐가 더 중요하다는 것을 알게 되었다. 버전 접두어처럼 나중에 바꿀 것을 미리 대비해두는 습관도 들여야겠다.
- 더미 해시처럼 보안을 위해 넣어둔 장치도 형식이 틀리면 조용히 효과가 사라진다는 것을 알게 되었다. 보안 장치는 "있는지"가 아니라 "동작하는지"를 테스트로 확인해야 한다.
