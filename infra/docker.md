# Docker

### 컨테이너(Container)
- 애플리케이션과 실행에 필요한 라이브러리, 설정을 하나로 묶어 격리된 환경에서 실행하는 기술
  - "내 PC에서는 되는데 서버에서는 안 된다"는 문제를 줄여줌
  - 가상 머신(VM)과 달리 OS 커널을 공유하므로, 가볍고 빠르게 시작됨
- 주요 용어
  - 이미지(Image): 컨테이너를 만들기 위한 읽기 전용 템플릿. 여러 레이어(Layer)가 쌓인 구조
  - 컨테이너(Container): 이미지를 실행한 인스턴스
  - Dockerfile: 이미지를 만드는 절차를 적은 파일
  - 레지스트리(Registry): 이미지를 저장하고 배포하는 저장소(Docker Hub 등)
- 테스트에서 컨테이너를 사용하는 방법은 [외부 의존성을 격리한 테스트](/di-spring-test/test-isolation.md) 참고

<br>

### Dockerfile과 멀티 스테이지 빌드
- 빌드에 필요한 도구(JDK, Gradle)와 실행에 필요한 것(JRE, jar)을 나눠서, 최종 이미지에는 실행에 필요한 것만 담는 방식
  - 이미지 크기가 작아지고, 컴파일러 등 불필요한 도구가 운영 이미지에 포함되지 않아 공격 표면이 줄어듦
```
FROM eclipse-temurin:25-jdk AS build
WORKDIR /workspace
COPY gradlew settings.gradle build.gradle ./
COPY gradle gradle
RUN ./gradlew --no-daemon dependencies > /dev/null
COPY src src
RUN ./gradlew --no-daemon bootJar && mv build/libs/*.jar app.jar

FROM eclipse-temurin:25-jre
RUN groupadd --system app && useradd --system --gid app app
WORKDIR /app
COPY --from=build /workspace/app.jar app.jar
USER app
ENV TZ=Asia/Seoul
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```
- 레이어 캐시 활용
  - Docker는 명령마다 레이어를 만들고, 입력이 바뀌지 않은 레이어는 다시 실행하지 않고 재사용함
  - 자주 바뀌지 않는 빌드 설정을 먼저 복사하고 의존성을 내려받은 뒤에 소스를 복사하면, 소스만 바뀌었을 때 의존성 다운로드를 건너뜀
- 비root 사용자로 실행
  - 기본값인 root로 실행하면, 애플리케이션 취약점으로 컨테이너가 장악되었을 때 피해가 커짐
- ENTRYPOINT는 exec 형식(JSON 배열)으로 작성함
  - 셸 형식으로 작성하면 셸이 1번 프로세스가 되어, docker stop의 종료 신호(SIGTERM)가 Java 프로세스에 전달되지 않을 수 있음
- 컨테이너의 기본 시간대는 대부분 UTC이므로, 로그 시각을 맞추려면 TZ를 지정함
  - 다만 업무 로직(cron 해석 등)은 TZ에 의존하지 않고 애플리케이션 설정으로 시간대를 명시하는 것이 안전함

<br>

### Docker Compose
- 여러 컨테이너를 하나의 YAML 파일로 정의하고 함께 실행하는 도구
```
docker compose up -d --build --wait
docker compose logs -f sentinel-api
docker compose down
```
- up 옵션
  - -d: 백그라운드 실행
  - --build: 이미지를 다시 빌드한 후 실행
  - --wait: 모든 서비스가 healthy 상태가 될 때까지 기다린 뒤 종료하며, 기동에 실패하면 오류로 종료함
- 같은 Compose 파일의 서비스들은 하나의 네트워크에 묶이며, 서비스 이름으로 서로를 찾을 수 있음
  - 컨테이너 안에서 localhost는 컨테이너 자기 자신을 가리키므로, 다른 컨테이너는 http://sentinel-api:8080처럼 서비스 이름으로 접근해야 함
  - 호스트 PC의 서비스(로컬 DB 등)는 host.docker.internal로 접근함. Linux에서는 extra_hosts에 host-gateway 매핑을 명시해야 함

<br>

### Compose 주요 설정
```
services:
  sentinel-api:
    build: ./api
    env_file: ./sentinel.env
    ports:
      - "127.0.0.1:8080:8080"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "bash -c ': > /dev/tcp/127.0.0.1/8080'"]
      interval: 3s
      timeout: 3s
      retries: 40

  sentinel-web:
    build: ./web
    environment:
      INTERNAL_API_URL: http://sentinel-api:8080
    depends_on:
      sentinel-api:
        condition: service_healthy
```
- ports
  - "8080:8080"은 모든 네트워크 인터페이스에 포트를 공개하므로, 같은 네트워크의 다른 PC에서도 접근할 수 있음
  - "127.0.0.1:8080:8080"처럼 주소를 지정하면 같은 PC에서만 접근할 수 있음. 앞단의 Reverse Proxy만 외부에 공개할 때 사용함
- 환경변수와 비밀 값
  - environment: 값을 Compose 파일에 직접 적음
  - env_file: 별도 파일에서 읽음. 비밀 값은 이 파일에 두고 .gitignore에 등록함
  - docker run -e로 비밀 값을 직접 넘기면 셸 히스토리에 남음
  - env 파일의 값은 따옴표까지 값으로 인식되므로 따옴표를 쓰지 않음
- restart(재시작 정책)
  - no: 재시작하지 않음(기본값)
  - on-failure: 비정상 종료 시에만 재시작함
  - always: 항상 재시작함. 설정 오류로 기동이 실패하면 무한 재시작 루프에 빠질 수 있음
  - unless-stopped: always와 같지만, 사용자가 직접 중지한 컨테이너는 재시작하지 않음
- healthcheck와 depends_on
  - 컨테이너가 시작되었다는 것과 애플리케이션이 요청을 받을 준비가 되었다는 것은 다름
  - healthcheck로 준비 상태를 판정하고, depends_on의 condition: service_healthy로 앞 서비스가 준비된 뒤에 시작하도록 함
  - JRE 이미지처럼 curl, wget이 없는 경우 bash의 /dev/tcp로 포트가 열렸는지만 확인할 수 있음
  - Spring Boot Actuator의 health Endpoint를 사용하는 방법도 있으며, 관련 내용은 [Actuator](/spring-framework/actuator.md) 참고

<br>

### 로그 관리
- 기본 로그 드라이버(json-file)는 로그 크기 제한이 없으므로, 오래 실행하면 디스크를 가득 채울 수 있음
- max-size와 max-file로 파일 하나의 크기와 보관 개수를 제한함
- YAML 앵커(&)와 별칭(*)을 사용하면 같은 설정을 여러 서비스에서 재사용할 수 있으며, x-로 시작하는 최상위 키는 Compose가 무시하므로 공통 설정을 두는 용도로 사용함
```
x-logging: &logging
  driver: json-file
  options:
    max-size: 10m
    max-file: "5"

services:
  sentinel-api:
    logging: *logging
```

<br>

### Volume
- 컨테이너는 삭제되면 내부에 저장한 파일도 함께 사라지므로, 유지해야 하는 데이터는 Volume에 저장함
  - Named Volume: Docker가 관리하는 저장 공간. DB 데이터, 발급받은 인증서 등
  - Bind Mount: 호스트의 파일이나 디렉터리를 컨테이너에 연결함. 설정 파일 등. :ro를 붙이면 읽기 전용
```
volumes:
  - ./proxy/Caddyfile:/etc/caddy/Caddyfile:ro
  - caddy_data:/data
```
- docker compose down -v는 Named Volume까지 삭제하므로, 테스트 환경처럼 매번 초기화해야 하는 경우에만 사용함

<br>

#### 참고
- Docker Docs <Multi-stage builds> - https://docs.docker.com/build/building/multi-stage/
- Docker Docs <Compose file reference> - https://docs.docker.com/reference/compose-file/
- Docker Docs <Configure logging drivers> - https://docs.docker.com/engine/logging/configure/

#### 배워가는 것들
- 멀티 스테이지 빌드와 레이어 순서만 신경 써도 이미지 크기와 빌드 시간이 크게 달라진다는 것을 알게 되었다.
- 컨테이너가 시작된 것과 애플리케이션이 준비된 것은 다르다는 점이 인상적이었다. healthcheck로 준비 상태를 판정해야 서비스 간 기동 순서가 의미를 가진다.
- 포트 공개 범위, 재시작 정책, 로그 크기처럼 기본값을 그대로 두면 운영 중에 문제가 되는 설정들이 있다는 것을 배웠다. 기본값이 무엇인지 확인하고 명시적으로 지정하는 습관을 들여야겠다.
