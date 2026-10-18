# CI와 GitHub Actions

### CI/CD
- CI(Continuous Integration, 지속적 통합)
  - 코드 변경을 자주 합치고, 합칠 때마다 빌드와 테스트를 자동으로 수행하여 문제를 일찍 발견하는 방식
  - Pull Request 단계에서 테스트가 실패하면 병합 전에 막을 수 있음
- CD(Continuous Delivery / Continuous Deployment, 지속적 전달 / 지속적 배포)
  - 통합된 코드를 언제든 배포할 수 있는 상태로 유지하거나(Delivery), 자동으로 배포까지 수행하는 것(Deployment)
- CI 도구: GitHub Actions, GitLab CI, Jenkins 등

<br>

### GitHub Actions의 구성 요소
- Workflow: 자동화할 작업 전체. .github/workflows 디렉터리의 YAML 파일 하나가 Workflow 하나임
- Event(on): Workflow를 실행할 시점. push, pull_request, schedule(cron), workflow_dispatch(수동 실행) 등
- Job: 하나의 실행 환경(Runner)에서 수행되는 작업 묶음. 기본적으로 Job끼리는 병렬로 실행되며, needs로 순서를 지정할 수 있음
- Step: Job 안에서 순서대로 실행되는 단계
  - uses: 미리 만들어진 Action을 사용함
  - run: 셸 명령을 실행함
- Runner: Job을 실행하는 머신. GitHub가 제공하는 ubuntu-latest 등에는 Docker가 설치되어 있어 Testcontainers를 바로 사용할 수 있음

<br>

### Workflow 예시
```
name: CI

on:
  pull_request:
  push:
    branches: [main]

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: read

jobs:
  api:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: api
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-java@v6
        with:
          distribution: zulu
          java-version: 25
      - uses: gradle/actions/setup-gradle@v6
      - run: ./gradlew test
      - name: 실패 시 테스트 리포트 보관
        if: failure()
        uses: actions/upload-artifact@v7
        with:
          name: api-test-report
          path: api/build/reports/tests/test
          retention-days: 7
```
- concurrency: 같은 그룹의 Workflow가 새로 시작되면 진행 중이던 이전 실행을 취소함. 같은 브랜치에 커밋을 연달아 올릴 때 불필요한 실행을 줄여줌
- permissions: Workflow가 사용하는 GITHUB_TOKEN의 권한. 필요한 만큼만 부여함(최소 권한 원칙)
- defaults.run.working-directory: 하나의 Repository에 여러 모듈이 있을 때 명령을 실행할 디렉터리를 지정함
- setup-java, setup-gradle, setup-node 등은 실행 환경 설치와 함께 의존성 캐시를 제공하여 실행 시간을 줄여줌

<br>

### 조건부 Step과 산출물 보관
- if 조건으로 Step 실행 여부를 정할 수 있음
  - success(): 앞의 Step이 모두 성공했을 때(기본값)
  - failure(): 앞의 Step 중 하나라도 실패했을 때. 실패 원인을 확인할 리포트나 로그를 보관할 때 사용함
  - always(): 결과와 관계없이 항상. 테스트용으로 띄운 컨테이너를 정리할 때 사용함
- upload-artifact로 테스트 리포트, 스크린샷, 로그 등을 실행 결과에 첨부하여 내려받을 수 있음
```
- name: E2E 스택 기동
  run: docker compose -f docker-compose.e2e.yml up -d --build --wait
- run: npm run e2e
- name: 실패 시 스택 로그 보관
  if: failure()
  run: docker compose -f docker-compose.e2e.yml logs --no-color > stack.log
- name: E2E 스택 정리
  if: always()
  run: docker compose -f docker-compose.e2e.yml down -v
```
- E2E 테스트는 운영과 같은 구성의 스택을 Docker Compose로 띄우되, 포트와 데이터를 분리하여 개발 환경과 겹치지 않게 함

<br>

### 종료 코드(Exit Code)로 단계 통과 여부 정하기
- CI는 각 명령의 종료 코드로 성공과 실패를 판단함
  - 0이면 성공, 0이 아니면 실패로 보고 Job을 중단함
- 배포 후 검증처럼 외부 시스템의 결과를 CI 단계의 통과 여부로 연결하려면, 결과를 종료 코드로 바꿔주는 스크립트를 만들면 됨
  - 실패 종류에 따라 종료 코드를 나누면 원인을 구분할 수 있음. 예: 0 통과, 1 검증 실패, 2 실행 불가·시간 초과
  - 특정 CI 도구에 묶이지 않도록 bash, curl, jq 같은 기본 도구로 작성하면 어떤 CI에서도 사용할 수 있음
- 셸 스크립트 작성 시 유의사항
  - set -euo pipefail: 명령 실패(-e), 정의되지 않은 변수 사용(-u), 파이프 중간의 실패(pipefail) 시 즉시 중단함
  - shellcheck로 따옴표 누락 같은 흔한 실수를 정적 분석할 수 있으며, CI에서 함께 검사하면 좋음
  - 토큰 같은 비밀 값은 출력하지 않으며, CI의 Secrets 기능으로 주입함
- GitHub Actions의 run 단계 기본 셸에 주의해야 함
  - shell을 지정하지 않으면 Linux Runner에서는 bash -e로 실행되어 pipefail이 없음
  - 예: 판정 스크립트 | tee summary.md처럼 결과를 요약에 남기면, 판정이 실패해도 마지막 명령(tee)이 성공하여 단계가 통과해버림
  - shell: bash를 명시하면 bash --noprofile --norc -eo pipefail로 실행되므로, Job이나 Workflow의 defaults에 지정해두는 것이 안전함
```
defaults:
  run:
    shell: bash
```
- 실패해야 하는 상황에서 정말 실패하는지도 테스트해야 함. 통과만 확인해서는 "항상 통과하는 검사"를 발견할 수 없음

<br>

### 정기 실행(schedule)과 알림
- on.schedule에 cron을 지정하면 정기적으로 Workflow를 실행할 수 있음(UTC 기준). 의존성 취약점 점검처럼 코드 변경이 없어도 결과가 바뀌는 작업에 사용함
```
on:
  schedule:
    - cron: "0 0 * * 1"    # 매주 월요일 00:00 UTC
  workflow_dispatch:
```
- 정기 실행의 함정
  - 실패 알림 메일은 cron을 마지막으로 수정한 사람에게만 가므로, 다른 사람은 실패를 모르고 지나칠 수 있음
  - 공개 저장소는 60일 동안 활동이 없으면 GitHub가 정기 실행을 자동으로 멈춤
- 실패를 사람에게 전달하는 방법
  - 실패하면 gh issue create로 이슈를 열고(이미 열려 있으면 댓글), 다시 통과하면 이슈를 닫음
  - 저장소를 지켜보는(Watch) 사람 모두에게 알림이 가고, 해결 여부가 이슈 상태로 남음
  - Job에 permissions: issues: write를 부여해야 함
  - 공개 저장소의 이슈에는 취약점 위치처럼 공격에 도움이 되는 내용은 적지 않음
- 취약점 점검 관련 내용은 [소프트웨어 공급망 보안](/infra/supply-chain-security.md) 참고

<br>

### Docker 이미지 빌드 캐시
- Runner는 매번 새로운 환경이므로, docker compose build로 이미지를 만들면 의존성 다운로드부터 다시 수행함
  - 시간이 오래 걸리고, Maven Central이나 플러그인 저장소의 일시 장애에 그대로 노출됨
- docker/build-push-action에 GitHub Actions 캐시(type=gha)를 지정하면 레이어 캐시를 실행 간에 재사용할 수 있음
  - 빌드 파일(build.gradle, package-lock.json)이 그대로면 의존성 단계를 다시 실행하지 않음
  - load: true로 Runner의 Docker에 이미지를 올려두면, 이후 docker compose up은 빌드 없이 그 이미지를 사용함
```
- uses: docker/build-push-action@<SHA> # v7.4.0
  with:
    context: api
    load: true
    tags: sentinel-e2e-api
    cache-from: type=gha,scope=e2e-api
    cache-to: type=gha,scope=e2e-api,mode=max
```

<br>

### JUnit XML 리포트
- JUnit XML은 테스트 결과를 표현하는 XML 형식으로써, 대부분의 CI 도구가 읽어서 테스트 결과 화면으로 보여줌
  - JUnit에서 시작되었지만, 언어와 관계없이 사실상의 표준으로 쓰임
  - testsuite 안에 testcase가 있고, 실패한 케이스에는 failure, 오류가 난 케이스에는 error 요소가 들어감
```
<testsuite name="결제 스위트" tests="3" failures="1" errors="0" time="1.234">
  <testcase name="결제 요청" classname="결제 스위트" time="0.421"/>
  <testcase name="결제 이력 조회" classname="결제 스위트" time="0.512">
    <failure message="status in [2xx]: was 500"/>
  </testcase>
  <testcase name="환불 요청" classname="결제 스위트" time="0.301"/>
</testsuite>
```
- 직접 만든 검증 도구의 결과도 이 형식으로 내보내면, CI 화면에서 어떤 케이스가 왜 실패했는지 표로 확인할 수 있음
- API에서 같은 리소스를 JSON과 XML로 함께 제공하는 방법은 [직렬화, Marshalling, JSON](/dto-json-cors/serialization-marshalling-JSON.md)의 Content Negotiation 참고
- CI 로그와 산출물은 볼 수 있는 사람이 많으므로, 요청·응답 원문처럼 민감한 정보는 넣지 않는 것이 좋음

<br>

#### 참고
- GitHub Docs <Understanding GitHub Actions> - https://docs.github.com/en/actions/get-started/understand-github-actions
- GitHub Docs <Workflow syntax for GitHub Actions> - https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
- ShellCheck - https://www.shellcheck.net/
- GitHub Docs <Workflow syntax - defaults.run.shell> - https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax#defaultsrunshell
- Docker Docs <GitHub Actions cache> - https://docs.docker.com/build/cache/backends/gha/

#### 배워가는 것들
- CI는 결국 명령의 종료 코드를 보고 판단한다는 것을 이해하게 되었다. 어떤 검증이든 종료 코드로 결과를 돌려주면 파이프라인의 한 단계로 붙일 수 있다.
- failure()와 always() 조건으로 실패했을 때의 리포트 보관과 뒷정리를 분리할 수 있다는 점이 유용했다. CI가 실패했을 때 원인을 바로 볼 수 있어야 고치는 시간이 줄어든다.
- concurrency와 permissions처럼 짧은 설정 몇 줄로 불필요한 실행과 과도한 권한을 줄일 수 있다는 것을 알게 되었다.
- 셸을 지정하지 않으면 pipefail이 빠져 판정이 실패해도 단계가 통과할 수 있다는 점이 인상적이었다. 검사는 통과하는 경우뿐 아니라 실패해야 하는 경우에도 실제로 실패하는지 확인해야 한다.
