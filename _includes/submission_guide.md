### 대회 제출 안내: GitHub release와 GHCR 이미지

참가 팀은 **등록을 한 번** 마치면, 그 뒤로는 **직접 빌드한 이미지를 등록한 이름으로 `docker push`할 때마다** 정식 release가 자동으로 만들어지고 제출 알림이 운영팀에 전달됩니다. 등록할 때는 이미지 이름의 **마지막 부분만** 적습니다. 같은 이름의 패키지가 없으면 등록 중에 저장소에 설치되는 workflow가 `hello-world` 자리표시 이미지로 패키지를 만들어 저장소에 연결하고, 시스템이 연결을 확인한 뒤에 등록이 끝납니다. 이 문서는 참가자가 준비할 것과 시스템이 확인하고 설치하는 것을 설명합니다.

### 1. 전체 흐름

1. **등록 1회**: 대회 공지의 등록 링크에서 개인정보 수집·이용에 동의하고 **팀장 이메일을 인증**한 뒤(이메일로 받은 6자리 코드를 입력합니다), 팀 정보, GitHub 저장소, GHCR 이미지 **이름의 마지막 부분**(예: `my-server`)을 입력하고, 안내 화면대로 저장소를 설정한 뒤 자동 연결 테스트를 통과합니다. 이 테스트 중에 시스템이 저장소에 제출용 workflow 파일을 commit하고, 같은 이름의 패키지가 없으면 그 commit이 workflow의 **link job**을 실행합니다(파일이 이미 같은 내용이면 commit이 없으므로 시스템이 Run workflow와 같은 실행을 요청합니다). link job이 `hello-world` 자리표시 이미지(tag `aisc-registration`)를 push해 이미지 패키지를 만들고 저장소에 연결하고, 시스템은 패키지가 저장소에 연결된 것을 확인한 뒤에 다음 단계로 넘어갑니다. 자리표시 tag는 제출이 아니며, 보통 1분 안에 끝납니다.
2. **그 뒤 제출마다 `docker push`**: 이미지를 **직접 빌드**해 등록한 이름에 **새 tag**를 붙여 `docker push`합니다. 등록을 마친 뒤에 push하세요. 그러면 push → release 자동 생성 → 제출 메일(운영팀 알림) 순서로 진행되고, 보통 1분 안에 끝납니다. 시험용과 제출용의 구분은 없어서 `latest`와 `aisc-registration`이 아닌 새 tag는 모두 제출입니다(아래 「4. 제출하는 방법」).

Actions가 제출 이미지를 빌드하지 않고, 저장소 루트에 Dockerfile이 있을 필요도 없습니다. 이미지에 `LABEL`을 넣을 필요도 없습니다. 참가자는 로컬에서 빌드한 이미지를 `docker push`만 하고, 저장소의 workflow는 등록 때 패키지를 연결하는 일(link job)과 push된 tag의 release를 만드는 일(release job)만 합니다.

### 2. 참가자 준비물

| 준비물 | 내용 |
|---|---|
| GitHub 저장소 | commit이 하나 이상 있어야 합니다. 빈 저장소(commit 0)는 등록할 수 없습니다. 저장소의 Actions가 켜져 있어야 합니다(아래 「5. Actions 켜기와 사용 시간」). 이미지를 이 저장소에서 빌드할 필요는 없습니다. |
| Collaborator | `AISystemChallenge`를 **Write 이상** 권한으로 초대합니다. 조직 저장소는 **Collaborators and teams**에서 초대합니다. 초대는 시스템이 자동으로 수락합니다. |
| Webhook | 이벤트는 **Releases만** 선택합니다. Payload URL과 Secret은 등록 화면에서 복사합니다. |
| GHCR 이미지 이름 | **마지막 부분만** 입력합니다(예: `my-server`). 전체 이름은 시스템이 `ghcr.io/<저장소 소유 계정>/<입력한 이름>`으로 만들고, 계정은 등록 저장소를 소유한 계정입니다. tag는 붙이지 않습니다. 영문 소문자와 숫자로 쓰고, 구분 문자(`.`, `_`, `__`, `-`)는 글자 사이에만 쓸 수 있으며 100자 이하입니다(대문자는 소문자로 바뀝니다). `a/b/c` 같은 계층 이름이나 다른 계정의 이름은 쓸 수 없습니다. 권장은 저장소 이름과 같게 하는 것입니다. 등록할 때 이미지가 없어도 됩니다. |
| GitHub PAT(classic) | 이미지를 push하는 데 `write:packages` 권한이 필요합니다. `docker login ghcr.io`에만 쓰고 저장소에 commit하지 마세요(아래 「4. 제출하는 방법」). |
| Docker | 이미지를 직접 빌드하고 `docker push`할 수 있는 환경입니다. |

<h5 style="margin-top: 48px;">등록 절차</h5>
<!-- ### 등록 절차 -->

1. 등록 Form에서 개인정보 수집·이용에 동의하고 **팀장 이메일을 인증**한 뒤, 팀 정보, GitHub 저장소 주소, GHCR 이미지 이름을 입력합니다. 이미지 이름은 **마지막 부분만** 적습니다(예: `my-server`). 전체 이름은 시스템이 `ghcr.io/<owner>/my-server`로 만듭니다.
   - **팀장 이메일 인증**: 「동의하고 이메일 인증」을 누르면 팀장 이메일 주소를 묻습니다. 주소를 입력하면 그 주소로 6자리 인증 코드가 가고, 다음 화면에 그 코드를 입력합니다. 맞으면 팀 정보 입력으로 넘어가고, 등록 정보의 팀장 이메일은 인증한 주소가 됩니다. 팀 정보 화면에는 이메일 칸이 없습니다.
   - 코드는 **10분** 안에 입력합니다. 메일이 보이지 않으면 스팸함을 확인하세요. 코드를 받지 못했거나 주소가 틀렸다면 같은 화면의 「다른 동작」에서 「코드 다시 받기」 또는 「이메일 주소 고치기」를 고릅니다.
   - 틀린 코드는 **5번**까지 입력할 수 있고, 인증 코드 메일은 한 번의 신청에서 **4통**까지 보냅니다(다시 받기와 주소 고치기 포함). 한도에 닿으면 인증이 끝나고 처음부터 다시 신청해야 합니다.
2. 안내 화면에서 아래를 합니다.
   - **Collaborator 초대**: 저장소에 `AISystemChallenge`를 Write 이상으로 초대합니다.
   - **Webhook 추가**: 저장소 **Settings → Webhooks → Add webhook**에서 아래처럼 설정합니다.
     - **Payload URL**과 **Secret**: 안내 화면의 값을 그대로 복사해 붙여넣습니다.
     - **Content type**: `application/json`
     - **SSL verification**: 활성(Enable SSL verification)
     - **이벤트**: `Let me select individual events`를 선택하고 **Releases만** 체크합니다. 기본으로 체크된 **Pushes는 해제**합니다.
     - **Active**: 체크
   - **확인할 것**: 저장소의 Actions가 켜져 있는지(아래 「5. Actions 켜기와 사용 시간」), 같은 이름의 패키지가 이미 있다면 저장소에 연결돼 있는지(아래 「6. 이미지를 저장소에 연결하기」)를 봅니다.
3. 「설정 완료 · 자동 연결 테스트 시작」을 누릅니다. 테스트 중에는 새로고침하거나 창을 닫지 마세요.
4. 시스템이 아래를 차례로 자동 진행합니다.
   1. **초대 수락**: `AISystemChallenge`가 저장소 초대를 수락합니다.
   2. **쓰기 권한 확인**: 저장소에 Write 이상 권한이 있는지 확인합니다.
   3. **이미지 이름 확인**: 이름이 규칙에 맞는지 확인하고, 같은 이름의 패키지가 없으면 통과합니다(패키지는 아래 단계에서 workflow의 link job이 만들어 저장소에 연결합니다. 이미지의 `LABEL`은 필요 없습니다). 이 저장소에 연결된 패키지가 있으면 그 패키지를 그대로 씁니다. 같은 이름의 패키지가 있는데 이 저장소에 연결돼 있지 않거나 다른 저장소에 연결돼 있으면 실패합니다.
   4. **제출 workflow commit**: `AISystemChallenge`가 기본 branch에 `.github/workflows/aisc-submission.yml`을 commit합니다. 이미 같은 내용이면 건너뛰고, 내용이 다르면 덮어씁니다. commit이 있으면 그 commit(push)이 workflow의 link job을 실행합니다(자세한 내용은 아래 「3. 등록 때 제출 workflow가 자동으로 설치됩니다」).
   5. **패키지 연결 확인**: 이미 이 저장소에 연결된 패키지가 있으면 건너뜁니다. 없으면 link job이 `hello-world` 자리표시 이미지를 push해 패키지를 만들고 저장소에 연결할 때까지 기다리며 확인합니다. 4단계에서 commit이 없었다면(파일이 이미 같은 내용) 시스템이 먼저 Run workflow와 같은 실행을 요청합니다. 몇 분 안에 연결이 확인되지 않으면 실패하고, 안내 화면이 원인과 할 일을 알려 줍니다(아래 「6. 이미지를 저장소에 연결하기」).
   6. **임시 draft release 생성**: 공개되지 않고, tag를 만들지 않으며, Actions도 실행하지 않습니다.
   7. **webhook 도착 확인**: 그 release의 webhook이 도착하는지 확인합니다.
   8. **임시 draft release 삭제**
5. 모두 통과하면 등록이 완료됩니다. 완료 화면은 패키지가 어떻게 연결됐는지(등록 중에 만들었는지, 이미 연결돼 있었는지)와 이어서 할 일(이미지 빌드와 `docker push`)을 안내합니다.

Payload URL과 Secret은 **저장소마다 고정**입니다. 같은 저장소를 다시 등록해도 값이 같고, 이미 대회 webhook이 있다면 그대로 두면 됩니다. webhook은 **하나면 충분**합니다. 여러 개를 만들면 같은 알림이 그 수만큼 전달됩니다.

##### 원클릭 복사 화면

등록 화면의 「원클릭 복사 화면 열기」 버튼을 누르면 `https://aisystemchallenge.github.io/submission-guide/copy/`가 새 탭으로 열립니다. 이 화면에서 Collaborator 사용자명, Payload URL, Secret을 버튼 하나로 복사할 수 있습니다.

- 값은 주소의 `#` 뒤에 담겨 전달되며, `#` 뒤쪽은 서버로 전송되지 않습니다.
- 이 화면은 대회 공식 형식의 값만 표시합니다. 경고가 보이면 그 값을 GitHub에 입력하지 말고, 등록 화면의 버튼으로 다시 여세요.
- 등록 화면의 버튼으로만 여세요. 주소만 직접 열면 값이 비어 있습니다.

<a id="workflow-install"></a>

### 3. 등록 때 제출 workflow가 자동으로 설치됩니다

자동 연결 테스트 중에 `AISystemChallenge`가 등록한 저장소의 기본 branch에 `.github/workflows/aisc-submission.yml`을 commit합니다. 모든 팀에 같은 내용이고 `AISC_IMAGE` 값(등록한 이미지 이름)만 다릅니다. 같은 내용을 [aisc-submission.yml](examples/aisc-submission.yml)에서 미리 볼 수 있습니다. 예시의 `AISC_IMAGE`는 `ghcr.io/<owner>/<name>` 자리표시이고, 등록할 때 시스템이 등록한 이름으로 채웁니다.

commit message는 `chore: add AI System Challenge submission workflow (image ghcr.io/…)`입니다.

이전 안내(release를 발행하면 workflow가 이미지를 빌드하던 방식, 또는 이미지에 `LABEL`을 넣으면 패키지가 연결된다고 안내했던 방식)로 등록한 팀의 저장소에는 이전 파일이 남아 있습니다. **다시 등록하면** 자동 연결 테스트가 그 파일을 이 문서의 새 내용으로 갱신하고(commit message는 `add` 대신 `update`로 시작합니다), 그 commit이 link job을 실행합니다.

##### workflow가 하는 일

이 workflow에는 job이 둘 있습니다. 어느 job이 실행되는지는 무엇이 workflow를 시작했는지에 따라 정해집니다. 제출 이미지는 빌드하지 않습니다. 빌드하는 것은 link job이 push하는 내용 없는 자리표시 이미지 하나뿐입니다.

| job | 언제 실행되나 | 하는 일 |
|---|---|---|
| **link** | 이 파일이 commit(push)될 때(등록 때 시스템의 commit, 나중에 파일을 고쳐 push할 때)와 Actions 탭에서 **Run workflow**를 누를 때. 파일이 이미 있는 저장소를 등록할 때는 시스템이 같은 실행(`workflow_dispatch`)을 요청합니다. 패키지 이벤트로는 실행되지 않습니다. | 자리표시 이미지 `aisc-registration`을 push해 이미지 패키지를 만들고 이 저장소에 연결합니다. 제출이 아닙니다. |
| **release** | 패키지 이벤트(`registry_package`)가 올 때 | push된 tag 이름의 정식 release를 만듭니다. 이것이 제출입니다. |

<h5 style="margin-top: 48px;">link job: 등록 때 패키지를 저장소에 연결합니다</h5>
<!-- ### link job: 등록 때 패키지를 저장소에 연결합니다 -->

이미지 패키지를 이 저장소에 연결하는 것은 이미지의 `LABEL`이 아니라 이 job입니다. 아래를 차례로 합니다.

1. 저장소의 `GITHUB_TOKEN`으로 `ghcr.io`에 로그인합니다. 토큰은 명령줄이 아니라 입력으로 넘깁니다.
2. 자리표시 이미지(`hello-world` 바탕, 이미지 소스 label은 이 저장소)를 빌드해 등록한 이름에 `aisc-registration` tag로 push합니다. 이 push가 이미지 패키지를 만들고 이 저장소에 연결합니다. `aisc-registration`은 자리표시 tag이며 제출이 아닙니다.
3. push가 실패하면 Actions 로그에 `::error::` 안내를 남기고 멈춥니다. 원인은 둘 중 하나입니다.
   - 같은 이름의 패키지가 이미 있는데 이 저장소에 연결돼 있지 않거나 이 저장소의 Actions가 쓸 수 없는 경우입니다(예: 등록 전에 직접 push해 만든 패키지). 그 패키지 페이지의 **Package settings → Connect repository**에서 이 저장소를 연결하세요. 이미 이 저장소에 연결된 패키지라면 이 실패는 무시해도 되고, 새 tag를 push하면 그대로 제출됩니다.
   - 조직 정책이 Actions의 `GITHUB_TOKEN`으로 패키지를 만들거나 쓰는 것을 막고 있는 경우입니다. 정책을 고친 뒤 **Run workflow**로 다시 실행하세요.
4. 성공하면 저장소 첫 화면 오른쪽 **Packages** 목록에 패키지가 보이면 연결된 것이라는 안내를 남깁니다. 이제 새 tag로 push하면 제출됩니다.

등록 중에는 시스템이 이 job의 결과(패키지가 이 저장소에 연결됐는지)를 기다리며 확인하므로, 등록이 통과했다면 패키지는 연결돼 있습니다. 나중에 패키지를 지웠다면 **Run workflow**로 이 job을 다시 실행해 만드세요(아래 「6. 이미지를 저장소에 연결하기」).

##### release job: push된 tag를 제출로 접수합니다

패키지 이벤트(`registry_package`)로 실행되고, 아래를 파일에 적힌 순서대로 확인합니다.

1. tag가 없는 이벤트는 건너뜁니다. 한 번의 push가 이벤트를 여러 개 만들 수 있습니다(같은 push의 플랫폼별 manifest 등). tag 없는 이벤트는 job을 시작하지 않고, tag가 붙은 이벤트만 release로 이어집니다.
2. 등록한 이름이 아닌 패키지는 건너뜁니다. 저장소를 소유한 계정과 패키지 이름을 소문자로 이어 붙인 `ghcr.io/<계정>/<패키지>`를 `AISC_IMAGE`와 비교합니다.
3. tag가 docker tag 규칙에 맞지 않으면 오류로 멈추고 release를 만들지 않습니다. 규칙은 이렇습니다. tag는 영문, 숫자, `_`, `.`, `-`만 쓰고 128자 이하여야 하며, 첫 글자는 영문, 숫자, `_` 중 하나입니다(예: `v1.0.0`).
4. `latest` tag는 제출하지 않습니다. `docker push`에 tag를 붙이지 않으면 `latest`가 올라가는데, 그것이 실수로 제출이 되지 않게 하려는 규칙입니다. Actions 로그에 안내만 남기고 건너뜁니다.
5. `aisc-registration` tag는 제출하지 않습니다. link job이 패키지를 연결하려고 push하는 자리표시 tag입니다. Actions 로그에 안내만 남기고 건너뜁니다.
6. 같은 tag의 release가 이미 있으면 건너뜁니다. 같은 tag를 다시 push해도 다시 제출되지 않습니다.
7. 그 밖의 모든 경우에 push된 tag 이름의 **정식 release**(pre-release도 draft도 아님)를 저장소 기본 branch에 만듭니다. 제목은 tag이고, 본문은 `제출 이미지 <AISC_IMAGE>:<tag> (<digest>) — GHCR push로 자동 생성`입니다.

workflow에 주어진 권한은 `contents: write`(release를 만들 때)와 `packages: write`(link job이 자리표시 이미지를 push할 때)뿐이고, 둘 다 저장소의 `GITHUB_TOKEN`으로 씁니다. 이 방식으로 만든 release는 다른 workflow를 실행하지 않지만 webhook 알림은 보내므로, 대회 시스템이 그 release를 받습니다. link job의 push도 다른 workflow를 실행하지 않습니다.

##### workflow 파일의 값

| 값 | 내용 |
|---|---|
| `AISC_IMAGE` | 등록한 이름입니다. 바꾸면 제출 이미지를 찾지 못하니 바꾸지 마세요. |

파일에서 팀마다 다른 값은 이것 하나뿐입니다. 다시 등록하면 시스템이 파일을 공통 내용으로 되돌립니다(고친 값은 덮어씁니다). 파일을 고쳐 push하면 link job이 다시 실행되어 자리표시 이미지를 다시 push합니다(제출은 아닙니다).

<a id="branch-protection"></a>

##### branch 보호 규칙이 commit을 막을 때

기본 branch에 branch 보호 규칙이나 ruleset이 있어 commit이 막히면 둘 중 하나를 한 뒤 다시 테스트하세요.

1. `AISystemChallenge`를 허용(bypass)합니다.
2. 공개 템플릿 [aisc-submission.yml](examples/aisc-submission.yml)을 저장소의 `.github/workflows/aisc-submission.yml`로 직접 추가합니다. 이때 `AISC_IMAGE` 줄**만** 등록한 이름으로 바꿉니다. 같은 내용이면 시스템이 commit을 건너뜁니다. 다른 줄까지 고쳐서 내용이 달라지면 시스템이 덮어쓰려고 하므로 다시 막힐 수 있습니다. 직접 추가한 파일도 그 commit(push)에서 link job을 실행하므로, 자리표시 이미지가 push됩니다(제출은 아닙니다).

##### 등록을 끝내지 못해도

workflow 파일은 자동 연결 테스트 중에 commit됩니다. 그래서 이후 단계가 실패하거나 등록을 끝내지 못해도 파일과, link job이 만든 자리표시 패키지는 저장소에 남습니다. 등록하지 않을 거라면 지워도 됩니다. 등록 commit이 저장소의 push 기반 CI를 실행할 수도 있습니다.

<a id="how-to-submit"></a>

### 4. 제출하는 방법

등록을 마친 뒤 제출은 직접 빌드한 이미지를 `docker push`하는 것이 전부입니다. 등록이 통과했다면 패키지는 이미 이 저장소에 연결돼 있으니 바로 push하면 됩니다(패키지를 지웠다면 아래 「6. 이미지를 저장소에 연결하기」).

##### ghcr.io에 로그인하기

GitHub의 **Settings → Developer settings → Personal access tokens → Tokens (classic)**에서 `write:packages` 권한의 PAT(classic)를 만들고, 아래처럼 로그인합니다.

```
echo <PAT> | docker login ghcr.io -u <GitHub 사용자명> --password-stdin
```

PAT는 비밀번호와 같습니다. 저장소, 이미지, 공개 문서에 넣거나 commit하지 마세요.

##### 이미지를 빌드하고 push하기

1. 이미지를 직접 빌드합니다. LABEL은 필요 없습니다. 넣어도 괜찮지만 `LABEL`만으로는 패키지가 저장소에 연결되지 않습니다. 패키지는 등록 때 workflow의 link job이 연결합니다(위 「3. 등록 때 제출 workflow가 자동으로 설치됩니다」).
2. **등록을 마친 뒤에**, 등록한 이름에 **새 tag**를 붙여 빌드하고 push합니다. 이름은 모두 소문자여야 합니다.

   ```
   docker build -t ghcr.io/<owner>/<name>:v1.0.0 .
   docker push ghcr.io/<owner>/<name>:v1.0.0
   ```

   직접 push해 만든 패키지(등록 전이나, 패키지를 지운 뒤)는 저장소에 연결되지 않아 `registry_package` 이벤트가 이 저장소에 오지 않고, release도 제출도 생기지 않습니다(아래 「6. 이미지를 저장소에 연결하기」).
3. 잠시 뒤 저장소의 **Releases**에 tag와 같은 이름의 release가 생기고, 운영팀에 제출 알림이 전달됩니다. push부터 보통 1분 안에 끝납니다.

##### 제출 규칙

- **새 tag를 push하면 제출입니다.** 시험용과 제출용의 구분은 없어서 `latest`와 `aisc-registration`이 아닌 새 tag는 모두 제출입니다. workflow는 pre-release나 draft를 만들지 않고 항상 정식 release를 만듭니다.
- **`latest` tag는 제출하지 않습니다.** `docker push`에 tag를 붙이지 않으면 `latest`가 올라가는데, 그것이 실수로 제출이 되지 않게 하려는 규칙입니다. 항상 버전 tag(예: `v1.0.0`)를 붙이세요.
- **`aisc-registration` tag는 제출하지 않습니다.** `aisc-registration`은 자리표시 tag이며 제출이 아닙니다. 등록 때 link job이 패키지를 연결하려고 push하는 tag이니, 제출에는 다른 버전 tag를 쓰세요.
- **같은 tag를 다시 push해도 다시 제출되지 않습니다.** 그 tag의 release가 이미 있으면 workflow가 건너뜁니다. 다시 제출하려면 새 tag(예: `v1.0.1`)로 push하세요.
- **tag 규칙**: tag는 영문, 숫자, `_`, `.`, `-`만 쓰고 128자 이하여야 하며, 첫 글자는 영문, 숫자, `_` 중 하나입니다(예: `v1.0.0`). 규칙에 맞지 않으면 workflow가 오류로 멈추고 release를 만들지 않습니다.

<a id="actions-usage"></a>

### 5. Actions 켜기와 사용 시간

- 저장소의 Actions가 켜져 있어야 합니다(**Settings → Actions → General**). 꺼져 있으면 link job도 release job도 실행되지 않아 패키지가 연결되지 않고 release도 제출도 생기지 않습니다.
- 조직 정책이 Actions를 막고 있을 수 있습니다. Actions의 `GITHUB_TOKEN`으로 패키지를 만들거나 쓰는 것을 막는 조직 정책도 있습니다. 그러면 link job이 실패하고, 정책을 고친 뒤 **Run workflow**로 다시 실행해야 합니다.
- 시스템은 위 두 가지를 직접 감지하지는 못합니다. 다만 그 결과로 link job이 실행되지 않거나 실패해 패키지가 몇 분 안에 연결되지 않으면 등록이 실패하고, 안내 화면이 Actions 설정을 먼저 확인하라고 알려 줍니다. 그럴 때 Actions를 켜거나 정책을 고친 뒤 다시 테스트하면 link job이 다시 실행됩니다.
- private 저장소는 그 계정의 Actions 무료 사용량(분)을 씁니다. 이 workflow는 자리표시 이미지를 push하고 release를 만드는 짧은 작업입니다. 한도는 GitHub 요금제에 따라 다르고 바뀔 수 있어서 이 문서에는 숫자를 적지 않습니다. GitHub의 안내를 확인하세요.
- 등록 중 패키지가 연결되지 않았거나 push했는데 release가 생기지 않으면 저장소의 Actions 탭에서 `AI System Challenge submission`의 실행과 로그를 먼저 확인하세요. link job의 `::error::` 안내, 그리고 건너뛴 이유(latest, aisc-registration, 이미 있는 tag, 등록하지 않은 이름)가 로그에 남습니다.

<a id="link-package"></a>

### 6. 이미지를 저장소에 연결하기

push가 이 저장소의 workflow를 실행하려면 패키지가 이 저장소에 **연결**돼 있어야 합니다. 연결되지 않은 패키지에 push하면 `registry_package` 이벤트가 이 저장소에 오지 않아 workflow도, release도, 제출도 없습니다.

##### 등록 때 자동으로 연결됩니다

같은 이름의 패키지가 없으면 등록 중에 commit된 workflow의 link job이 저장소의 `GITHUB_TOKEN`으로 `hello-world` 자리표시 이미지(`aisc-registration`)를 push하고, 그때 만들어지는 패키지가 이 저장소에 연결됩니다. 보통 1분 안에 끝납니다(위 「3. 등록 때 제출 workflow가 자동으로 설치됩니다」). 시스템은 연결이 확인될 때까지 기다린 뒤 다음 단계로 넘어가므로, 등록이 통과했다면 패키지는 만들어져 이 저장소에 연결돼 있습니다. 파일이 이미 같은 내용이어서 commit이 없었다면 시스템이 Run workflow와 같은 실행을 대신 요청합니다. LABEL은 필요 없습니다. 있어도 괜찮지만 `LABEL`만으로는 연결되지 않습니다. 직접 push해 만든 패키지는 이미지에 `LABEL`이 있어도 연결되지 않은 채로 남습니다.

##### 연결됐는지 확인하기

- 저장소 첫 화면 오른쪽의 **Packages** 목록에 그 패키지가 보이면 연결된 것입니다.
- 패키지 페이지에 이 저장소 이름이 보이면 연결된 것입니다.

##### 연결되지 않았을 때

1. 등록 중에 '이미지 패키지가 저장소에 연결되지 않았습니다'가 나오면, 몇 분 기다려도 패키지가 이 저장소에 연결되지 않은 것입니다. 저장소의 **Actions** 탭에서 `AI System Challenge submission`의 마지막 실행을 열어 link job의 로그를 보세요. 원인은 Actions가 꺼져 있거나(위 「5. Actions 켜기와 사용 시간」), 조직 정책이 Actions의 `GITHUB_TOKEN`으로 패키지를 쓰는 것을 막고 있거나, 같은 이름의 연결되지 않은 패키지가 있는 경우입니다. 고친 뒤 안내 화면의 「다시 테스트」를 누르면 link job이 다시 실행됩니다.
2. 등록을 마친 뒤 패키지를 지웠거나 연결이 풀렸다면, 저장소의 **Actions** 탭에서 `AI System Challenge submission`을 열고 **Run workflow**를 눌러 link job을 다시 실행합니다. Actions가 켜져 있어야 합니다.
3. 실행이 실패하면 로그의 `::error::` 안내를 따릅니다. 원인은 같은 이름의 패키지(아래 「직접 연결하기」)이거나 조직 정책입니다. 정책을 고친 뒤에는 **Run workflow**로 다시 실행하세요.
4. 연결된 뒤에는 **새 tag**로 push합니다. 연결하기 전에 push한 tag는 이 저장소에 이벤트를 만들지 못했습니다.

##### 직접 연결하기

1. 패키지 페이지의 **Package settings → Connect repository**에서 이 저장소를 연결합니다.
2. 연결한 뒤 **새 tag**로 다시 push합니다. 연결하기 전에 push한 tag는 이 저장소에 이벤트를 만들지 못했습니다.

##### 같은 이름의 패키지가 이미 있을 때

같은 이름의 패키지가 이미 있으면(예: 등록 전에 직접 push한 이미지) 위 「직접 연결하기」로 이 저장소에 연결하세요. 연결하지 않을 거라면 다른 이름으로 등록하세요.

- 같은 이름의 패키지가 있는데 이 저장소에 연결돼 있지 않거나 다른 저장소에 연결돼 있으면 자동 연결 테스트가 실패합니다.
- 이 저장소에 연결되지 않은 **비공개** 패키지는 대회 계정(`AISystemChallenge`)에게 보이지 않아 이름 확인은 통과합니다. 하지만 link job의 push가 그 패키지에서 거부되어(`::error::`) 패키지가 연결되지 않으므로, 시스템이 몇 분 기다린 뒤 등록이 '이미지 패키지가 저장소에 연결되지 않았습니다'로 실패합니다. 바로 거절되지는 않습니다. 같은 이름의 패키지가 이미 있다면 먼저 연결하거나 다른 이름으로 등록하세요.
- 이미 이 저장소에 연결된 패키지라면 link job이 실패해도 무시해도 됩니다. 새 tag를 push하면 그대로 제출됩니다.

### 7. push 때 일어나는 일

1. 참가자가 등록한 이름에 새 tag를 `docker push`합니다(패키지가 이 저장소에 연결돼 있어야 합니다).
2. GitHub가 패키지 이벤트(`registry_package`)를 보내 저장소의 제출 workflow(Actions)의 release job을 실행하고, release job이 그 tag 이름의 정식 release를 만듭니다.
3. 시스템이 release의 webhook을 받고, 서명과 저장소를 확인합니다.
4. 저장소에 연결된 GHCR 패키지 중 **release tag와 같은 tag**의 버전을 GitHub를 통해 조회합니다. release 직후에 찾고, 이미지가 아직 없으면 약 5분 뒤 한 번 더 찾습니다. release 본문에 이미지 주소를 적지 않아도 됩니다.
5. release 본문에 `ghcr.io/…` 주소가 있으면(자동으로 만든 release의 본문에도 있습니다) 그 주소도 같은 규칙으로 확인합니다. 거부된 주소는 운영팀 알림에 **경고로만** 표시됩니다.
6. 운영팀에 알림을 보냅니다. 알림에는 pull 주소와 digest가 들어갑니다.

두 번째 확인 때도 패키지 목록에서 tag를 찾지 못하면 알림에서 이미지가 빠질 수 있습니다. 그런 경우에는 운영팀에 문의하세요.

##### 이미지 규칙

**같은 계정 + 저장소에 연결 + tag 존재**, 세 가지를 모두 만족해야 합니다.

- 같은 계정은 push한 개인이 아니라 **저장소를 소유한 계정**을 기준으로 합니다. 그래서 팀원 누구나 push할 수 있습니다.
- 연결은 이미지가 그 저장소의 패키지로 묶여 있다는 뜻입니다. 등록 때 workflow의 link job이 자리표시 이미지를 push해 연결하고, 연결되지 않았으면 패키지 페이지에서 직접 연결합니다(위 「6. 이미지를 저장소에 연결하기」). 이미지의 `LABEL`은 연결에 쓰이지 않습니다.
- tag는 release tag와 같아야 합니다. 자동으로 만든 release는 push한 tag의 이름으로 만들어지므로 항상 같습니다.

### 8. digest란?

digest는 이미지 manifest의 `sha256` 지문(content address)입니다. tag(`v1.0.0` 같은 이름)는 나중에 다른 빌드로 옮겨갈 수 있지만, digest는 그 빌드 하나에 고정됩니다. 그래서 알림에는 digest가 함께 들어가고, 아래처럼 정확히 그 빌드를 받을 수 있습니다.

```
docker pull ghcr.io/<owner>/<image>@sha256:…
```

multi-arch 이미지는 index의 digest를 씁니다.

### 9. 거부되는 경우

release 본문에 적힌 주소가 아래 경우에 해당하면 이미지가 거부됩니다. 거부된 주소는 운영팀 알림에 **경고로만** 표시되고 pull 대상에서 빠집니다.

- 다른 계정의 이미지
- 저장소에 연결되지 않은 패키지. 개인에게 직접 읽기 권한을 준 것만으로는 연결로 인정하지 않습니다.
- tag가 없는 경우. 5분 뒤 재확인한 뒤에도 없을 때입니다.
- 조회에 실패한 경우

### 10. FAQ

**등록할 때 이미지가 있어야 하나요?**
아니요. 이름의 마지막 부분만 등록합니다. 같은 이름의 패키지가 없으면 통과하고, 등록 중에 commit되는 workflow의 link job이 `hello-world` 자리표시 이미지(`aisc-registration`)를 push해 패키지를 만들고 저장소에 연결합니다. 시스템이 그 연결을 확인한 뒤에 등록이 끝납니다. 제출할 이미지는 등록을 마친 뒤에 직접 빌드하고 push하세요.

**이미지 이름에는 무엇을 적나요?**
마지막 부분만 적습니다(예: `my-server`). 전체 이름은 시스템이 `ghcr.io/<저장소 소유 계정>/my-server`로 만듭니다. 영문 소문자와 숫자로 쓰고, 구분 문자(`.`, `_`, `__`, `-`)는 글자 사이에만 쓸 수 있으며 100자 이하입니다. 대문자는 소문자로 바뀌고, tag(`:v1.0.0`)와 digest, `a/b/c` 같은 계층 이름, 다른 계정의 이름은 쓸 수 없습니다. `ghcr.io/<소유 계정>/my-server`처럼 전체 이름을 적어도 마지막 부분만 사용합니다.

**인증 코드 메일이 오지 않아요.**
먼저 스팸함을 확인하세요. 메일 제목은 「이메일 인증 코드」로 끝나고, 본문에 6자리 코드와 유효 시간이 있습니다. 코드 입력 화면에 보이는 주소가 맞는지도 확인하세요. 오지 않으면 같은 화면의 「다른 동작」에서 「코드 다시 받기」를 고르세요(새 코드가 가고 이전 코드는 쓸 수 없습니다). 주소가 틀렸다면 「이메일 주소 고치기」를 고릅니다. 인증 코드 메일은 한 번의 신청에서 4통까지만 보냅니다.

**인증 코드를 틀리면 어떻게 되나요?**
틀린 코드는 5번까지 입력할 수 있고, 남은 횟수가 화면에 보입니다. 5번째로 틀리면 인증이 끝나고 처음부터 다시 신청해야 합니다. 6자리 숫자가 아닌 값(빈 칸 포함)은 횟수에 들지 않습니다. 코드는 10분 뒤에 만료되고, 만료된 코드는 「코드 다시 받기」로 새 코드를 받아야 합니다.

**`latest` tag로 push하면 제출되나요?**
아니요. `latest`는 제출하지 않습니다. tag 없이 `docker push`하면 `latest`가 올라가는데, 그것이 실수로 제출이 되지 않게 하려는 규칙입니다. 버전 tag(예: `v1.0.0`)를 붙여 push하세요.

**같은 tag를 다시 push하면 다시 제출되나요?**
아니요. 그 tag의 release가 이미 있으면 workflow가 건너뜁니다. 새로 제출하려면 새 tag(예: `v1.0.1`)로 push하세요.

**시험용 push와 제출용 push가 따로 있나요?**
아니요. 구분이 없습니다. `latest`와 `aisc-registration`이 아닌 새 tag를 push하면 모두 제출로 접수됩니다.

**`aisc-registration` tag는 무엇인가요?**
등록 때 workflow의 link job이 push하는 **자리표시** tag입니다. 이 push가 이미지 패키지를 만들고 저장소에 연결합니다. `aisc-registration`은 자리표시 tag이며 제출이 아닙니다. 이 이름으로 push해도 release job이 건너뛰므로 release도 알림도 만들어지지 않습니다. 제출에는 다른 버전 tag(예: `v1.0.0`)를 쓰세요.

**LABEL을 넣어야 하나요?**
아니요. LABEL은 필요 없습니다. 있어도 괜찮지만, 패키지를 저장소에 연결하는 것은 `LABEL`이 아니라 workflow의 link job입니다. 직접 push해 만든 패키지는 이미지에 `LABEL`이 있어도 연결되지 않은 채로 남습니다.

**패키지가 저장소에 연결됐는지 어떻게 확인하나요?**
등록이 통과했다면 연결돼 있습니다. 직접 보려면 저장소 첫 화면 오른쪽 **Packages** 목록에 그 패키지가 보이거나, 패키지 페이지에 이 저장소 이름이 보이는지 확인하세요. 패키지를 지웠거나 연결돼 있지 않으면 저장소의 **Actions** 탭에서 `AI System Challenge submission`을 열고 **Run workflow**를 눌러 link job을 다시 실행하세요. 그래도 연결되지 않으면 패키지 페이지의 **Package settings → Connect repository**에서 연결한 뒤 새 tag로 다시 push하세요. 자세한 내용은 위 「6. 이미지를 저장소에 연결하기」에 있습니다.

**등록 때 실행된 link job이 실패했어요.**
Actions 탭의 실행 로그에서 `::error::` 안내를 보세요. 원인은 둘 중 하나입니다. 같은 이름의 패키지가 이미 있는데 이 저장소에 연결돼 있지 않은 경우(직접 push해 만든 패키지 등)는 그 패키지 페이지의 **Package settings → Connect repository**에서 이 저장소를 연결하세요. 이미 이 저장소에 연결된 패키지라면 실패를 무시해도 되고, 새 tag를 push하면 제출됩니다. 조직 정책이 Actions의 `GITHUB_TOKEN`으로 패키지를 쓰는 것을 막는 경우는 정책을 고친 뒤 **Run workflow**로 다시 실행하세요. 등록 중이었다면 안내 화면의 「다시 테스트」로도 link job이 다시 실행됩니다.

**등록 중에 '이미지 패키지가 저장소에 연결되지 않았습니다'가 나와요.**
시스템이 몇 분 기다렸는데도 패키지가 이 저장소에 연결되지 않았다는 뜻입니다. 저장소의 Actions가 꺼져 있거나, 조직 정책이 Actions의 `GITHUB_TOKEN`으로 패키지를 쓰는 것을 막고 있거나, 같은 이름의 연결되지 않은 패키지(직접 push해 만든 패키지 등)가 있어 link job의 push가 거부됐을 수 있습니다. Actions 탭의 `AI System Challenge submission` 실행 로그에서 link job의 `::error::` 안내를 확인하고, 고친 뒤 「다시 테스트」를 누르세요. 같은 이름의 패키지가 있으면 **Package settings → Connect repository**에서 연결하거나 다른 이름으로 처음부터 다시 등록하세요.

**push했는데 release가 생기지 않아요.**
저장소의 Actions 탭에서 `AI System Challenge submission` 실행을 먼저 보세요. 실행이 없으면 패키지가 저장소에 연결돼 있지 않거나(등록 전에 직접 push해 만든 패키지 등) Actions가 꺼져 있을 가능성이 큽니다. 실행이 있다면 로그에 건너뛴 이유(`latest`, `aisc-registration`, 이미 release가 있는 tag, 등록하지 않은 이름)나 tag 규칙 오류가 남아 있습니다.

**Actions가 꺼져 있으면 어떻게 되나요?**
시스템은 Actions가 켜져 있는지 직접 감지하지는 못하지만, 꺼져 있으면 link job이 실행되지 않아 등록 중 패키지가 연결되지 않으므로 몇 분 뒤 등록이 실패하고 안내 화면이 원인을 알려 줍니다. 꺼진 채로 둘 수는 없습니다. release job도 실행되지 않아 release도 제출도 생기지 않습니다. **Settings → Actions → General**에서 켠 뒤 「다시 테스트」를 누르세요. 패키지를 지운 뒤라면 **Run workflow**로 link job을 실행하세요.

**이미지를 public으로 해야 하나요?**
아니요. public 여부보다 저장소에 연결돼 있는지가 중요합니다.

**release 본문에 pull 주소를 꼭 적어야 하나요?**
아니요. release는 release job이 자동으로 만들고, 시스템이 저장소에 연결된 패키지에서 release tag와 같은 tag를 찾습니다.

**push하는 tag에 규칙이 있나요?**
docker tag 규칙을 따릅니다. tag는 영문, 숫자, `_`, `.`, `-`만 쓰고 128자 이하여야 하며, 첫 글자는 영문, 숫자, `_` 중 하나입니다. 예: `v1.0.0`. 맞지 않으면 workflow가 오류로 멈추고 release를 만들지 않습니다.

**workflow 파일을 고쳐도 되나요?**
고치지 않는 것을 권장합니다. `AISC_IMAGE`는 바꾸지 마세요. 바꾸면 제출 이미지를 찾지 못합니다. 그 밖의 부분을 고치면 release가 제출 규칙대로 만들어지지 않을 수 있습니다. 파일을 고쳐 push하면 link job이 다시 실행됩니다(자리표시 이미지만 push하며 제출은 아닙니다). 다시 등록하면 파일은 공통 내용으로 되돌아갑니다.

**제출 workflow만 직접 추가하면 되나요?**
아니요. 등록 1회(Collaborator 초대, webhook, 자동 연결 테스트)는 따로 해야 합니다. 직접 추가하는 방법은 branch 보호 규칙이 commit을 막을 때만 씁니다.

**branch 보호 규칙이 있으면 어떻게 하나요?**
`AISystemChallenge`를 허용(bypass)하거나, 공개 템플릿을 직접 추가하세요. 자세한 방법은 위 「branch 보호 규칙이 commit을 막을 때」에 있습니다.

**이전에 등록한 팀은 어떻게 하나요?**
이전 안내(release를 발행하면 workflow가 이미지를 빌드하던 방식, 또는 이미지에 `LABEL`을 넣으면 패키지가 연결된다고 안내했던 방식)로 등록한 팀의 저장소에는 이전 workflow 파일이 남아 있습니다. 같은 저장소와 이미지 이름으로 **다시 등록**하면 자동 연결 테스트가 파일을 새 내용으로 갱신하고, 그 commit이 link job을 실행해 패키지를 저장소에 연결합니다. 새 파일은 제출 이미지를 빌드하지 않으므로, 이미지는 직접 빌드한 뒤 `docker push`하세요. 이미 push한 패키지가 있다면 Packages 목록에서 연결됐는지 확인하고, 연결되지 않았으면 위 「6. 이미지를 저장소에 연결하기」를 따르세요.
