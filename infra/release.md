# 릴리스와 배포

### 릴리스(Release)
- 사용자에게 전달할 특정 시점의 코드를 정하고, 버전 번호를 붙여 배포 가능한 상태로 만드는 것
- main 브랜치를 그대로 운영 서버에 배포하면
  - 머지된 모든 변경이 바로 운영에 반영되므로, 여러 변경을 모아서 확인하고 내보내기 어려움
  - 지금 운영 중인 것이 어떤 코드인지, 문제가 생겼을 때 어디로 되돌려야 하는지 불분명해짐
- 릴리스 태그를 만들고 운영 서버는 태그를 받아 배포하면, 운영 중인 버전이 항상 명확하고 되돌릴 지점도 분명해짐

<br>

### 유의적 버전(Semantic Versioning)
- MAJOR.MINOR.PATCH 형식으로 버전을 표기하고, 변경의 성격에 따라 올릴 자리를 정하는 규칙
  - MAJOR: 호환이 깨지는 변경. API 응답 형식 변경, 설정 이름 변경·삭제, 되돌릴 수 없는 DB 마이그레이션(컬럼 삭제·이름 변경) 등
  - MINOR: 호환을 유지하면서 기능을 추가함
  - PATCH: 호환을 유지하는 버그 수정
- 1.0.0 미만(0.y.z)은 초기 개발 단계로써, 언제든 호환이 깨질 수 있음을 의미함
- 1.0.0-rc.1처럼 하이픈 뒤에 붙이는 사전 릴리스(Pre-release), Maven의 1.0.1-SNAPSHOT처럼 개발 중인 버전을 표시하는 방식도 있음
- 서비스의 "호환"이 무엇인지 프로젝트마다 정의해두어야 함
  - 예: 외부 CI 스크립트가 사용하는 API 응답이 바뀌면 MAJOR

<br>

### Git 태그로 릴리스하기
- 태그(Tag)는 특정 커밋에 붙이는 이름으로써, 브랜치와 달리 움직이지 않음
  - Annotated Tag(git tag -a)는 작성자, 날짜, 메시지를 함께 저장하므로 릴리스에 적합함
```
git tag -a v1.0.1 -m "Release 1.0.1"
git push origin v1.0.1

git describe --tags                       # 지금 체크아웃한 버전
git tag --sort=-v:refname | head -n 3     # 최근 릴리스
```
- 릴리스 절차 예시
  1. 버전 파일(build.gradle의 version, package.json의 version)을 올리는 PR을 만들고 머지함
  2. 머지된 커밋에 태그를 붙여 Push함
  3. CI에서 태그 이름과 버전 파일이 일치하는지 확인함
  4. 운영 서버는 해당 태그를 체크아웃하여 배포함
- 버전이 여러 파일에 있다면(Backend, Frontend), 항상 같은 값인지 CI에서 검사하는 스크립트를 두면 실수를 막을 수 있음
- 이미 서버가 받아간 태그는 고치지 않고, 잘못된 릴리스는 새 PATCH 버전으로 바로잡음
- 릴리스 사이의 변경 목록은 git log --first-parent --oneline v1.0.0..main처럼 태그 사이의 커밋으로 확인할 수 있음

<br>

### 실행 중인 애플리케이션의 버전 확인하기
- 배포 후 "정말 새 버전이 떴는가"를 확인할 수 있어야 함
- Spring Boot Gradle 플러그인에 buildInfo()를 지정하면, 빌드 시 META-INF/build-info.properties가 만들어지고 BuildProperties Bean이 등록됨
  - 버전, 빌드 시각, 그룹, 아티팩트 정보를 담고 있음
  - Actuator의 info Endpoint에도 표시되며, 관련 내용은 [Actuator](/spring-framework/actuator.md) 참고
```
build.gradle

springBoot {
    buildInfo()
}


@RestController
class SystemStatusController {

    private final BuildProperties build;

    @GetMapping("/api/system-status")
    SystemStatus get() {
        return new SystemStatus(build.getVersion(), build.getTime());
    }

}
```
- 빌드 시각이 jar에 들어가면 코드가 같아도 매번 다른 jar가 만들어지므로, Docker 이미지 캐시를 사용하지 못한다는 점을 함께 고려해야 함
- 버전은 외부로 나가는 요청의 User-Agent에도 활용할 수 있으며, 관련 내용은 [HTTP 요청 전송의 세부 동작](/http/request-transfer.md) 참고

<br>

### 업데이트 절차
- Docker Compose로 운영하는 서버의 업데이트 예시
```
git fetch --tags
git checkout v1.0.1
docker compose pull --ignore-buildable
docker compose build --pull
docker compose up -d --wait
docker image prune -f
```
- 베이스 이미지 갱신
  - eclipse-temurin:25-jre, node:24-alpine 같은 태그는 같은 이름으로 보안 패치가 반영된 새 이미지가 올라옴
  - up --build만 사용하면 서버에 남아 있던 예전 베이스 이미지로 빌드하므로, build --pull로 새로 받아야 함
  - pull --ignore-buildable은 빌드하지 않고 받아서 쓰는 이미지(Caddy, DB 등)만 새로 받음
- DB 마이그레이션이 포함된 업데이트라면, 적용 전에 DB 백업(스냅샷)을 먼저 만듦
  - 이번 업데이트에 새 마이그레이션이 있는지는 태그 사이의 파일 변경으로 확인할 수 있음(git diff --stat HEAD v1.0.1 -- src/main/resources/db/migration)
- 배포 전 점검 스크립트(Preflight)
  - 필수 환경변수, 인증서 만료일, 포트, 디스크 등을 배포 전에 확인하여 기동 실패를 미리 막음
  - 비밀 값은 출력하지 않고, 설정 여부만 표시함
- 배포 후 확인: 컨테이너 상태(healthy), 화면이나 API로 버전과 빌드 시각, 최근 로그

<br>

### 되돌리기(Rollback)
- 이전 릴리스 태그를 체크아웃하여 다시 배포함
- DB 마이그레이션이 있었다면 되돌릴 수 있는지가 달라짐
  - 테이블·컬럼 추가만 있었다면, 이전 버전도 대부분 그대로 기동됨(ddl-auto: validate는 엔티티에 없는 컬럼을 허용함)
  - 컬럼 삭제·이름 변경이 있었다면, 이전 버전의 엔티티 검증이 실패하므로 코드만 되돌릴 수 없음. 수정 버전을 새로 배포하거나 백업으로 복원해야 함
- 따라서 컬럼 삭제나 이름 변경은 한 번에 하지 않고, "새 컬럼 추가 → 양쪽에 쓰기 → 읽기 전환 → 옛 컬럼 삭제"처럼 여러 릴리스에 나누어 진행하는 방식(Expand and Contract)을 사용하기도 함
- 마이그레이션 관련 내용은 [스키마 마이그레이션](/database/schema-migration.md) 참고

<br>

### 리허설(Rehearsal)
- 문서화한 설치·업데이트·복구 절차를 일회용 환경에서 처음부터 그대로 따라 해보는 것
  - 문서에 빠진 단계, 환경마다 다른 동작, 실제로 해본 적 없는 복구 절차의 문제를 운영 전에 발견할 수 있음
  - 백업은 복원해보기 전까지는 백업이라고 할 수 없음
- Docker in Docker(docker:dind) 같은 일회용 서버를 사용하면, 깨끗한 환경에서 반복해서 확인할 수 있음

<br>

#### 참고
- Semantic Versioning 2.0.0 - https://semver.org/
- Git Documentation <Tagging> - https://git-scm.com/book/en/v2/Git-Basics-Tagging
- Spring Boot Gradle Plugin Reference <Integrating with Actuator> - https://docs.spring.io/spring-boot/gradle-plugin/integrating-with-actuator.html
- Docker Docs <docker compose build> - https://docs.docker.com/reference/cli/docker/compose/build/

#### 배워가는 것들
- main을 그대로 운영에 반영하지 않고 태그로 릴리스하면, 운영 중인 코드와 되돌릴 지점이 명확해진다는 것을 알게 되었다.
- 같은 이미지 태그로 보안 패치가 올라오기 때문에 --pull 없이 빌드하면 패치가 반영되지 않는다는 점이 인상적이었다.
- DB 마이그레이션에 따라 되돌릴 수 있는지가 달라진다는 것을 배웠다. 컬럼 삭제처럼 되돌리기 어려운 변경은 버전 규칙과 배포 절차까지 함께 고려해야겠다.
