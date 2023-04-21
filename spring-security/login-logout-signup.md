# 로그인 & 로그아웃(Login & Logout), 회원가입(Sign-Up)

### CSRF(Cross-Site Request Forgery)
* 인증이 이루어진 사용자가 사용자의 의도와는 무관하게 특정 요청을 보내도록 유도하는 것으로써, 사이트 간 요청 위조를 의미함.
* Spring Security에서는 CSRF를 방지하기 위해 의도한 사용자만 리소스를 변경할 수 있도록 허용하는 CsrfFilter를 제공함.

<br>

### Java Date, Calendar vs LocalDateTime, ZonedDateTime
* JDK 1.7 이전까지는 날짜/시간 관련 인스턴스 생성 시, java.util 패키지에 있는 Date와 Calendar 클래스를 사용했었음.
  * 그러나 인스턴스 값이 변할 수 있을뿐더러, 애매하게 명명된 메서드(ex. getTime())와 애매하게 구성된 상수(Calendar.JANUARY ~ Calendar.DECEMBER -> 0 ~ 11)로 인한 혼선 때문에 지금은 거의 사용하지 않음.
* JDK 1.8 이후 버전부터 java.time 패키지에 LocalDateTime 클래스가 추가되면서 날짜와 시간을 모두 컨트롤할 수 있게 되었음.
  * 기본적으로 YYYY-MM-DDTHH:mm:ss 형태를 띄며, DateTimeFormatter의 ofPattern 메서드를 이용하여 원하는 형태의 문자열 구성이 가능함.
  * 제공해주는 여러 메서드들을 통해 현재시각과 이전/다음 일/월에 대한 계산을 편리하게 도와줌.
  * Timezone 개념이 포함된 ZonedDateTime도 이용 가능하며, 여러 국가의 시각을 표현할 때 사용함.

<br>

### Epoch Time
* 1970년 1월 1일 0시(UTC)부터 경과시간을 초(Secound) 단위로 환산하여 나타낸 정수를 의미하며, Unix/Linux 기반 OS에 있는 터미널 상에서 date +%s 명령으로 확인 가능함.
* UNIX Time, POSIX Time이라고도 함.
* 주기적으로 백업 파일을 Export할 때, 해당 파일에 대한 명명 목적으로 많이 사용함.
* Java의 경우, Timestamp 클래스를 이용하여 활용 가능.

<br>

### Spring Security PasswordEncoder
* 임의의 DB에 비밀번호를 단방향 암호화 알고리즘으로 인코딩하여 안전하게 저장하기 위한 역할을 수행하는 인터페이스.
* 다양한 해싱 전략의 패스워드를 지원하며, 아래와 같이 Bean을 만들어야 함.
```
// BcryptPasswordEncoder를 사용하는 경우
@Bean 
public PasswordEncoder passwordEncoder() { 
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}

// Argon2PasswordEncoder를 사용하는 경우(build.gradle에 관련모듈 의존성 추가 필요)
@Bean
public PasswordEncoder passwordEncoder() {
    return Argon2PasswordEncoder.defaultsForSpringSecurity_v5_8();
}

implementation 'org.bouncycastle:bcprov-jdk15on:1.70'
```

<br>

### Argon2
* 2015년 암호 해싱대회(Password Hashing Competition)에서 우승작으로 선정된 단방향 암호화 알고리즘.
* 과거에는 Bcrypt를 많이 사용했으나, 최근들어 사용빈도가 지속적으로 높아지고 있음.

<br>

#### 참고
* https://madplay.github.io/post/java8-date-and-time


#### 배워가는 것들
* Backend Module과 Frontend Module을 분리하여 구축할 때 Backend 단에서 처리하는 로그인/로그아웃에 대한 프로세스를 익힐 수 있었다.
* 사실 기계적으로 Bcrypt를 사용했었는데, Argon2가 있다는 것도 새롭게 알게 되었다. 이에 대한 이론 학습과 병행하여 확실하게 나의 것으로 만들어서 단련하는 작업을 잘 진행하려 한다. 
