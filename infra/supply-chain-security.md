# 소프트웨어 공급망 보안

### 소프트웨어 공급망(Software Supply Chain)
- 애플리케이션이 만들어져 배포되기까지 거치는 모든 외부 요소
  - 라이브러리(직접 추가한 의존성과 그 의존성이 끌어오는 전이 의존성)
  - 빌드 도구(Gradle, npm)와 그 플러그인
  - 베이스 Docker 이미지와 그 안의 OS 패키지
  - CI에서 실행하는 외부 Action
- 내가 작성한 코드가 안전해도, 이 중 하나에 취약점이 있거나 악성 코드가 섞이면 애플리케이션 전체가 위험해짐
  - 예: Log4Shell(Log4j 원격 코드 실행), 유지보수자 계정 탈취로 악성 버전이 배포된 npm 패키지들
- 대응은 크게 세 가지로 나눌 수 있음
  - 알려진 취약점을 찾아서 고침(취약점 점검)
  - 받는 것이 의도한 그것인지 확인함(버전·체크섬 고정)
  - 내 코드의 취약한 패턴을 찾음(정적 분석)

<br>

### 취약점 식별 체계
- CVE(Common Vulnerabilities and Exposures): 공개된 취약점마다 부여되는 고유 번호. 예: CVE-2021-44228(Log4Shell)
- CVSS(Common Vulnerability Scoring System): 취약점의 심각도를 0.0~10.0 점수로 표현함
  - 낮음(0.1~3.9), 중간(4.0~6.9), 높음(7.0~8.9), 치명(9.0~10.0)
- GHSA(GitHub Security Advisory), OSV(Open Source Vulnerabilities) 같은 데이터베이스가 패키지 이름과 버전 범위별로 취약점을 정리해둠
- 점수만으로 판단하지 말고, 해당 기능을 실제로 사용하는지도 확인해야 함
  - 예: Tomcat 자체 인증(FORM, DIGEST) 관련 취약점은, 인증을 애플리케이션 필터로 직접 구현한 서비스에는 영향이 없음
  - 다만 영향이 없더라도 수정 버전이 있으면 올려두는 편이 관리가 쉬움

<br>

### 의존성 취약점 점검 도구
- Dependabot(GitHub)
  - 취약점 경고(Alerts): 의존성 그래프를 기준으로, 알려진 취약점이 있는 의존성을 알려줌
  - 보안 업데이트(Security Updates): 취약점이 수정된 버전으로 올리는 PR을 자동으로 만듦
  - 버전 업데이트(Version Updates): .github/dependabot.yml에 지정한 주기로 의존성을 최신 버전으로 올리는 PR을 만듦
```
version: 2
updates:
  - package-ecosystem: gradle
    directory: /api
    schedule:
      interval: weekly
    groups:
      api:
        update-types: [minor, patch]
```
  - groups로 여러 의존성을 한 PR로 묶으면 PR 수를 줄일 수 있으며, MAJOR 업데이트처럼 직접 판단해야 하는 것은 ignore로 제외할 수 있음
- 의존성 그래프(Dependency Graph)
  - npm처럼 Lock 파일이 있는 생태계는 GitHub가 저장소 파일만 읽어 그래프를 만들지만, Gradle은 빌드를 실행해야 실제 의존성이 결정됨
  - 따라서 Gradle 프로젝트는 CI에서 해석한 의존성을 제출(gradle/actions/dependency-submission)해야 Dependabot 경고가 동작함
  - 이때 빌드 도구 자체의 클래스패스는 제외해야, 애플리케이션에 포함되지 않는 의존성의 경고가 생기지 않음
- 의존성 검토(Dependency Review)
  - PR에서 새로 추가되거나 바뀐 의존성만 검사하여, 취약한 버전이 들어오는 것을 병합 전에 막음
- OSV-Scanner, Trivy, Grype 등
  - Lock 파일, 디렉터리, Docker 이미지를 검사하여 취약점을 찾는 도구
  - 이미지 검사는 실제 배포물 안의 OS 패키지까지 볼 수 있다는 장점이 있음
  - 다만 jar 안의 정보만으로 Maven 좌표(groupId:artifactId)를 추정하면 잘못 읽는 경우가 있어, Java 의존성은 빌드 도구가 해석한 결과(Gradle Lock 파일 등)로 검사하는 것이 정확함
- 점검이 아무것도 찾지 못했을 때, 정말 안전한 것인지 도구가 패키지를 읽지 못한 것인지 구분해야 함
  - 검사한 패키지 수나 반드시 있어야 할 패키지(예: org.springframework:spring-core)가 결과에 포함되었는지 함께 확인하면 "조용한 통과"를 막을 수 있음

<br>

### 언제 막을 것인가
- 새 취약점은 코드를 바꾸지 않아도 어느 날 공개되므로, 모든 PR을 막으면 무관한 작업까지 멈춤
- 정책 예시
  - 의존성을 바꾼 PR(build.gradle, package-lock.json, Dockerfile 변경)만 높음 이상 취약점이 있으면 실패시킴
  - 정기 점검(매주)에서 발견되면 실패로 표시하고 이슈를 열어 알림
  - 수정 버전이 없는 취약점(대부분의 OS 패키지)은 보고만 함
- Spring Boot가 관리하는 의존성 버전에 취약점이 있고 Boot 새 버전이 아직 없다면, 관리 버전 속성으로 같은 MINOR의 수정 버전만 덮어쓸 수 있음
  - Boot가 수정 버전을 관리하게 되면 덮어쓴 설정을 제거함
```
build.gradle

ext['tomcat.version'] = '11.0.27'
ext['jackson-bom.version'] = '3.1.7'
```

<br>

### 받는 것이 의도한 것인지 확인하기
- GitHub Actions를 커밋 SHA로 고정함
  - actions/checkout@v7 같은 태그는 옮길 수 있으므로, 같은 이름으로 다른 코드가 실행될 수 있음
  - SHA로 고정하고 버전은 주석으로 남기면, Dependabot이 SHA와 주석을 함께 갱신해줌
```
- uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
```
- Gradle 배포판의 체크섬을 고정함
  - gradle-wrapper.properties의 distributionSha256Sum을 지정하면, 내려받은 Gradle이 다르면 실행을 거부함
- 외부 도구를 CI에서 내려받아 실행할 때는 버전과 체크섬을 고정함
- Lock 파일(package-lock.json, gradle.lockfile)로 전이 의존성까지 정확한 버전을 고정함
- 외부 저장소 장애 대비
  - 플러그인 저장소(Gradle Plugin Portal)나 Maven Central이 일시적으로 실패해도 빌드가 멈추지 않도록 저장소 순서를 정하고 캐시를 활용함
  - Docker 빌드에서 의존성 단계에 jar까지 모두 받아두면, 이후 컴파일은 네트워크 없이 진행되고 레이어 캐시도 재사용됨

<br>

### 정적 분석(SAST)과 CodeQL
- SAST(Static Application Security Testing): 코드를 실행하지 않고 분석하여 취약한 패턴을 찾는 것
- CodeQL(GitHub)
  - 코드를 데이터베이스로 만들고 쿼리로 취약한 데이터 흐름을 찾음. 예: 사용자 입력이 검증 없이 요청 URL에 들어가는 SSRF
  - security-extended 규칙 묶음을 사용하면 더 많은 규칙으로 검사함
  - 공개 저장소에서는 무료로 사용할 수 있음
- 경고의 처리
  - 실제 문제라면 고치고, 오탐(False Positive)이라면 왜 안전한지 근거를 남기고 닫음
  - 자동 수정 제안(Copilot Autofix 등)은 경고를 없애는 데 집중하므로, 정상 입력에서 예외가 나는 등 동작이 바뀌지 않는지 확인해야 함
- 정기 분석 결과는 보안 탭에만 쌓이면 아무도 모르고 지나치기 쉬우므로, 경고가 있으면 이슈를 열어 알리는 식으로 사람에게 전달해야 함
  - 공개 저장소의 이슈에는 취약점 위치나 규칙을 적지 않고, 권한이 있는 사람만 보는 곳에서 확인하도록 함

<br>

#### 참고
- GitHub Docs <Dependabot> - https://docs.github.com/en/code-security/dependabot
- GitHub Docs <Security hardening for GitHub Actions> - https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions
- OSV-Scanner - https://google.github.io/osv-scanner/
- CodeQL Documentation - https://codeql.github.com/docs/
- Gradle Documentation <Verifying the Gradle Wrapper> - https://docs.gradle.org/current/userguide/gradle_wrapper.html#sec:verification

#### 배워가는 것들
- 내가 작성하지 않은 코드가 애플리케이션의 대부분을 차지한다는 것을 다시 느꼈다. 의존성도 내 코드처럼 꾸준히 점검하고 관리해야 한다.
- 점검 도구가 0건을 보고해도, 도구가 제대로 읽지 못한 결과일 수 있다는 점이 인상적이었다. 검사 대상이 실제로 포함되었는지 확인하는 장치가 필요하다.
- 태그는 움직일 수 있어서 Action을 SHA로 고정해야 한다는 것을 알게 되었다. 받는 것이 의도한 것인지 확인하는 일도 보안의 일부다.
