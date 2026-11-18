### 대회 제출 안내: GitHub release와 GHCR 이미지

참가 팀은 **등록을 한 번** 마치면, 그 뒤로는 **직접 빌드한 이미지를 등록한 이름으로 `docker push`할 때마다** 정식 release가 자동으로 만들어지고 제출 알림이 운영팀에 전달됩니다. 이 문서는 참가자가 준비할 것과 시스템이 확인하고 설치하는 것을 설명합니다.

### 1. 전체 흐름

1. **등록 1회**: 대회 공지의 등록 링크에서 팀 정보, GitHub 저장소, GHCR 이미지 **이름**을 입력하고, 안내 화면대로 저장소를 설정한 뒤 자동 연결 테스트를 통과합니다. 이 테스트 중에 시스템이 저장소에 제출용 workflow 파일을 commit합니다.
2. **제출마다 `docker push`**: 이미지를 **직접 빌드**해 등록한 이름에 **새 tag**를 붙여 `docker push`합니다. 그러면 push → release 자동 생성 → 운영팀 알림 순서로 진행되고, 보통 1분 안에 끝납니다. 시험용과 제출용의 구분은 없어서 `latest`가 아닌 새 tag는 모두 제출입니다(아래 「4. 제출하는 방법」).

Actions가 이미지를 빌드하지 않고, 저장소 루트에 Dockerfile이 있을 필요도 없습니다. 참가자는 로컬에서 빌드한 이미지를 `docker push`만 하고, 저장소의 workflow는 push된 tag의 release를 만드는 일만 합니다.

### 2. 참가자 준비물

| 준비물 | 내용 |
|---|---|
| GitHub 저장소 | commit이 하나 이상 있어야 합니다. 빈 저장소(commit 0)는 등록할 수 없습니다. 저장소의 Actions가 켜져 있어야 합니다(아래 「5. Actions 켜기와 사용 시간」). 이미지를 이 저장소에서 빌드할 필요는 없습니다. |
| Collaborator | `AISystemChallenge`를 **Write 이상** 권한으로 초대합니다. 조직 저장소는 **Collaborators and teams**에서 초대합니다. 초대는 시스템이 자동으로 수락합니다. |
| Webhook | 이벤트는 **Releases만** 선택합니다. Payload URL과 Secret은 등록 화면에서 복사합니다. |
| GHCR 이미지 이름 | **tag 없이 이름만** 등록합니다. 이름은 `ghcr.io/<저장소 소유 계정>/<이름>`이며, 계정은 등록 저장소를 소유한 계정과 같아야 합니다. 권장은 저장소 이름과 같게 하는 것입니다(`github.com/owner/repo`이면 `ghcr.io/owner/repo`). `/`로 나눈 계층 이름도 쓸 수 있고, 대문자는 소문자로 바뀝니다. 등록할 때 이미지가 없어도 됩니다. |
| GitHub PAT(classic) | 이미지를 push하는 데 `write:packages` 권한이 필요합니다. `docker login ghcr.io`에만 쓰고 저장소에 commit하지 마세요(아래 「4. 제출하는 방법」). |
| Docker | 이미지를 직접 빌드하고 `docker push`할 수 있는 환경입니다. |

<h5 style="margin-top: 48px;">등록 절차</h5>
<!-- #### 등록 절차 -->

1. 등록 Form에 팀 정보, GitHub 저장소 주소, GHCR 이미지 이름을 입력합니다. 이미지는 tag 없이 이름만 적습니다(예: `ghcr.io/<owner>/<repo>`).
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
   3. **이미지 이름 확인**: 같은 이름의 패키지가 없으면 통과합니다(첫 `docker push`로 패키지가 만들어지고, 이미지의 `LABEL`로 이 저장소에 연결됩니다). 이 저장소에 연결된 패키지가 있으면 통과합니다. 같은 이름의 패키지가 있는데 이 저장소에 연결돼 있지 않거나 다른 저장소에 연결돼 있으면 실패합니다.
   4. **제출 workflow commit**: `AISystemChallenge`가 기본 branch에 `.github/workflows/aisc-submission.yml`을 commit합니다. 이미 같은 내용이면 건너뛰고, 내용이 다르면 덮어씁니다(자세한 내용은 아래 「3. 등록 때 제출 workflow가 자동으로 설치됩니다」).
   5. **임시 draft release 생성**: 공개되지 않고, tag를 만들지 않으며, Actions도 실행하지 않습니다.
   6. **webhook 도착 확인**: 그 release의 webhook이 도착하는지 확인합니다.
   7. **임시 draft release 삭제**
5. 모두 통과하면 등록이 완료됩니다. 완료 화면은 이어서 할 일(이미지 빌드와 `docker push`)을 안내합니다.

Payload URL과 Secret은 **저장소마다 고정**입니다. 같은 저장소를 다시 등록해도 값이 같고, 이미 대회 webhook이 있다면 그대로 두면 됩니다. webhook은 **하나면 충분**합니다. 여러 개를 만들면 같은 알림이 그 수만큼 전달됩니다.

##### 원클릭 복사 화면

등록 화면의 「원클릭 복사 화면 열기」 버튼을 누르면 `https://aisystemchallenge.github.io/submission-guide/copy/`가 새 탭으로 열립니다. 이 화면에서 Collaborator 사용자명, Payload URL, Secret을 버튼 하나로 복사할 수 있습니다.

- 값은 주소의 `#` 뒤에 담겨 전달되며, `#` 뒤쪽은 서버로 전송되지 않습니다.
- 이 화면은 대회 공식 형식의 값만 표시합니다. 경고가 보이면 그 값을 GitHub에 입력하지 말고, 등록 화면의 버튼으로 다시 여세요.
- 등록 화면의 버튼으로만 여세요. 주소만 직접 열면 값이 비어 있습니다.

### 3. 등록 때 제출 workflow가 자동으로 설치됩니다

자동 연결 테스트 중에 `AISystemChallenge`가 등록한 저장소의 기본 branch에 `.github/workflows/aisc-submission.yml`을 commit합니다. 모든 팀에 같은 내용이고 `AISC_IMAGE` 값(등록한 이미지 이름)만 다릅니다. 같은 내용을 [aisc-submission.yml](examples/aisc-submission.yml)에서 미리 볼 수 있습니다. 예시의 `AISC_IMAGE`는 `ghcr.io/<owner>/<name>` 자리표시이고, 등록할 때 시스템이 등록한 이름으로 채웁니다.

commit message는 `chore: add AI System Challenge submission workflow (image ghcr.io/…)`입니다.

이전 안내(release를 발행하면 workflow가 이미지를 빌드하던 방식)로 등록한 팀의 저장소에는 이전 파일이 남아 있습니다. **다시 등록하면** 자동 연결 테스트가 그 파일을 이 문서의 새 내용으로 갱신합니다(commit message는 `add` 대신 `update`로 시작합니다).

##### workflow가 하는 일

이 workflow는 **이미지를 빌드하지 않습니다.** 패키지 이벤트(`registry_package`)로 실행되고, 아래를 파일에 적힌 순서대로 확인합니다.

1. tag가 없는 이벤트는 건너뜁니다. 한 번의 push가 이벤트를 여러 개 만들 수 있습니다(같은 push의 플랫폼별 manifest 등). tag 없는 이벤트는 job을 시작하지 않고, tag가 붙은 이벤트만 release로 이어집니다.
2. 등록한 이름이 아닌 패키지는 건너뜁니다. 저장소를 소유한 계정과 패키지 이름을 소문자로 이어 붙인 `ghcr.io/<계정>/<패키지>`를 `AISC_IMAGE`와 비교합니다.
3. tag가 docker tag 규칙에 맞지 않으면 오류로 멈추고 release를 만들지 않습니다. 규칙은 이렇습니다. tag는 영문, 숫자, `_`, `.`, `-`만 쓰고 128자 이하여야 하며, 첫 글자는 영문, 숫자, `_` 중 하나입니다(예: `v1.0.0`).
4. `latest` tag는 제출하지 않습니다. `docker push`에 tag를 붙이지 않으면 `latest`가 올라가는데, 그것이 실수로 제출이 되지 않게 하려는 규칙입니다. Actions 로그에 안내만 남기고 건너뜁니다.
5. 같은 tag의 release가 이미 있으면 건너뜁니다. 같은 tag를 다시 push해도 다시 제출되지 않습니다.
6. 그 밖의 모든 경우에 push된 tag 이름의 **정식 release**(pre-release도 draft도 아님)를 저장소 기본 branch에 만듭니다. 제목은 tag이고, 본문은 `제출 이미지 <AISC_IMAGE>:<tag> (<digest>) — GHCR push로 자동 생성`입니다.

workflow에 주어진 권한은 `contents: write`뿐이고, release는 저장소의 `GITHUB_TOKEN`으로 만듭니다. 이 방식으로 만든 release는 다른 workflow를 실행하지 않지만 webhook 알림은 보내므로, 대회 시스템이 그 release를 받습니다.

##### workflow 파일의 값

| 값 | 내용 |
|---|---|
| `AISC_IMAGE` | 등록한 이름입니다. 바꾸면 제출 이미지를 찾지 못하니 바꾸지 마세요. |

파일에서 팀마다 다른 값은 이것 하나뿐입니다. 다시 등록하면 시스템이 파일을 공통 내용으로 되돌립니다(고친 값은 덮어씁니다).

##### branch 보호 규칙이 commit을 막을 때

기본 branch에 branch 보호 규칙이나 ruleset이 있어 commit이 막히면 둘 중 하나를 한 뒤 다시 테스트하세요.

1. `AISystemChallenge`를 허용(bypass)합니다.
2. 공개 템플릿 [aisc-submission.yml](examples/aisc-submission.yml)을 저장소의 `.github/workflows/aisc-submission.yml`로 직접 추가합니다. 이때 `AISC_IMAGE` 줄**만** 등록한 이름으로 바꿉니다. 같은 내용이면 시스템이 commit을 건너뜁니다. 다른 줄까지 고쳐서 내용이 달라지면 시스템이 덮어쓰려고 하므로 다시 막힐 수 있습니다.

##### 등록을 끝내지 못해도

workflow 파일은 자동 연결 테스트 중에 commit됩니다. 그래서 이후 단계가 실패하거나 등록을 끝내지 못해도 파일은 저장소에 남습니다. 등록하지 않을 거라면 지워도 됩니다. 등록 commit이 저장소의 push 기반 CI를 실행할 수도 있습니다.

### 4. 제출하는 방법

등록을 마친 뒤 제출은 직접 빌드한 이미지를 `docker push`하는 것이 전부입니다.

##### ghcr.io에 로그인하기

GitHub의 **Settings → Developer settings → Personal access tokens → Tokens (classic)**에서 `write:packages` 권한의 PAT(classic)를 만들고, 아래처럼 로그인합니다.

```
echo <PAT> | docker login ghcr.io -u <GitHub 사용자명> --password-stdin
```

PAT는 비밀번호와 같습니다. 저장소, 이미지, 공개 문서에 넣거나 commit하지 마세요.

##### 이미지를 빌드하고 push하기

1. 이미지에 아래 `LABEL`을 넣습니다. `<owner>/<repo>`는 등록한 저장소입니다. `Dockerfile`에 한 줄을 넣거나, `docker build --label org.opencontainers.image.source=https://github.com/<owner>/<repo> …`로 붙여도 됩니다.

   ```
   LABEL org.opencontainers.image.source=https://github.com/<owner>/<repo>
   ```

   이 label이 첫 push로 만들어지는 패키지를 저장소에 연결합니다. 연결되지 않으면 `registry_package` 이벤트가 이 저장소에 오지 않아 release도 제출도 생기지 않습니다(아래 「6. 이미지를 저장소에 연결하기」).
2. 등록한 이름에 **새 tag**를 붙여 빌드하고 push합니다. 이름은 모두 소문자여야 합니다.

   ```
   docker build -t ghcr.io/<owner>/<repo>:v1.0.0 .
   docker push ghcr.io/<owner>/<repo>:v1.0.0
   ```
3. 잠시 뒤 저장소의 **Releases**에 tag와 같은 이름의 release가 생기고, 운영팀에 제출 알림이 전달됩니다. push부터 보통 1분 안에 끝납니다.

##### 제출 규칙

- **새 tag를 push하면 제출입니다.** 시험용과 제출용의 구분은 없어서 `latest`가 아닌 새 tag는 모두 제출입니다. workflow는 pre-release나 draft를 만들지 않고 항상 정식 release를 만듭니다.
- **`latest` tag는 제출하지 않습니다.** `docker push`에 tag를 붙이지 않으면 `latest`가 올라가는데, 그것이 실수로 제출이 되지 않게 하려는 규칙입니다. 항상 버전 tag(예: `v1.0.0`)를 붙이세요.
- **같은 tag를 다시 push해도 다시 제출되지 않습니다.** 그 tag의 release가 이미 있으면 workflow가 건너뜁니다. 다시 제출하려면 새 tag(예: `v1.0.1`)로 push하세요.
- **tag 규칙**: tag는 영문, 숫자, `_`, `.`, `-`만 쓰고 128자 이하여야 하며, 첫 글자는 영문, 숫자, `_` 중 하나입니다(예: `v1.0.0`). 규칙에 맞지 않으면 workflow가 오류로 멈추고 release를 만들지 않습니다.

### 5. Actions 켜기와 사용 시간

- 저장소의 Actions가 켜져 있어야 합니다(**Settings → Actions → General**). 꺼져 있으면 push해도 workflow가 실행되지 않아 release도 제출도 생기지 않습니다.
- 조직 정책이 Actions를 막고 있을 수 있습니다.
- 시스템은 위 두 가지를 감지하지 못합니다. 등록은 통과하더라도 직접 확인해 주세요.
- private 저장소는 그 계정의 Actions 무료 사용량(분)을 씁니다. 이 workflow는 release를 만드는 짧은 작업입니다. 한도는 GitHub 요금제에 따라 다르고 바뀔 수 있어서 이 문서에는 숫자를 적지 않습니다. GitHub의 안내를 확인하세요.
- push했는데 release가 생기지 않으면 저장소의 Actions 탭에서 `AI System Challenge submission`의 실행과 로그를 먼저 확인하세요. 건너뛴 이유(latest, 이미 있는 tag, 등록하지 않은 이름)가 로그에 남습니다.

### 6. 이미지를 저장소에 연결하기

push가 이 저장소의 workflow를 실행하려면 패키지가 이 저장소에 **연결**돼 있어야 합니다. 연결되지 않은 패키지에 push하면 `registry_package` 이벤트가 이 저장소에 오지 않아 workflow도, release도, 제출도 없습니다.

##### 처음 push할 때 연결하기

이미지에 `LABEL org.opencontainers.image.source=https://github.com/<owner>/<repo>`를 넣고 첫 push를 하면, 그때 만들어지는 패키지가 그 label의 저장소에 연결됩니다(위 「4. 제출하는 방법」).

##### 연결됐는지 확인하기

- 패키지 페이지에 이 저장소 이름이 보이면 연결된 것입니다.
- 저장소 첫 화면 오른쪽의 **Packages** 목록에 그 패키지가 보여도 연결된 것입니다.

##### 직접 연결하기

1. 패키지 페이지의 **Package settings → Connect repository**에서 이 저장소를 연결합니다.
2. 연결한 뒤 **새 tag**로 다시 push합니다. 연결하기 전에 push한 tag는 이 저장소에 이벤트를 만들지 못했습니다.

##### 같은 이름의 패키지가 이미 있을 때

같은 이름의 패키지가 이미 있으면(예: 이전에 직접 push한 이미지) 위 「직접 연결하기」로 이 저장소에 연결하세요. 연결하지 않을 거라면 다른 이름으로 등록하세요.

- 같은 이름의 패키지가 있는데 이 저장소에 연결돼 있지 않거나 다른 저장소에 연결돼 있으면 자동 연결 테스트가 실패합니다.
- 이 저장소에 연결되지 않은 **비공개** 패키지는 대회 계정(`AISystemChallenge`)에게 보이지 않습니다. 그래서 등록은 통과해도, 연결하기 전에는 push해도 release도 제출도 생기지 않습니다. 같은 이름의 패키지가 이미 있다면 먼저 연결하세요.

### 7. push 때 일어나는 일

1. 참가자가 등록한 이름에 새 tag를 `docker push`합니다.
2. GitHub가 패키지 이벤트(`registry_package`)를 보내 저장소의 제출 workflow(Actions)를 실행하고, workflow가 그 tag 이름의 정식 release를 만듭니다.
3. 시스템이 release의 webhook을 받고, 서명과 저장소를 확인합니다.
4. 저장소에 연결된 GHCR 패키지 중 **release tag와 같은 tag**의 버전을 GitHub를 통해 조회합니다. release 직후에 찾고, 이미지가 아직 없으면 약 5분 뒤 한 번 더 찾습니다. release 본문에 이미지 주소를 적지 않아도 됩니다.
5. release 본문에 `ghcr.io/…` 주소가 있으면(자동으로 만든 release의 본문에도 있습니다) 그 주소도 같은 규칙으로 확인합니다. 거부된 주소는 운영팀 알림에 **경고로만** 표시됩니다.
6. 운영팀에 알림을 보냅니다. 알림에는 pull 주소와 digest가 들어갑니다.

두 번째 확인 때도 패키지 목록에서 tag를 찾지 못하면 알림에서 이미지가 빠질 수 있습니다. 그런 경우에는 운영팀에 문의하세요.

##### 이미지 규칙

**같은 계정 + 저장소에 연결 + tag 존재**, 세 가지를 모두 만족해야 합니다.

- 같은 계정은 push한 개인이 아니라 **저장소를 소유한 계정**을 기준으로 합니다. 그래서 팀원 누구나 push할 수 있습니다.
- 연결은 이미지가 그 저장소의 패키지로 묶여 있다는 뜻입니다. 이미지의 `org.opencontainers.image.source` label로 연결하거나, 패키지 페이지에서 직접 연결합니다(위 「6. 이미지를 저장소에 연결하기」).
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
아니요. 이름만 등록합니다. 같은 이름의 패키지가 없으면 통과하고, 첫 `docker push`로 패키지가 만들어집니다. 이미지에 `LABEL org.opencontainers.image.source=https://github.com/<owner>/<repo>`를 넣어 두면 그 패키지가 저장소에 연결됩니다.

**`latest` tag로 push하면 제출되나요?**
아니요. `latest`는 제출하지 않습니다. tag 없이 `docker push`하면 `latest`가 올라가는데, 그것이 실수로 제출이 되지 않게 하려는 규칙입니다. 버전 tag(예: `v1.0.0`)를 붙여 push하세요.

**같은 tag를 다시 push하면 다시 제출되나요?**
아니요. 그 tag의 release가 이미 있으면 workflow가 건너뜁니다. 새로 제출하려면 새 tag(예: `v1.0.1`)로 push하세요.

**시험용 push와 제출용 push가 따로 있나요?**
아니요. 구분이 없습니다. `latest`가 아닌 새 tag를 push하면 모두 제출로 접수됩니다.

**패키지가 저장소에 연결됐는지 어떻게 확인하나요?**
패키지 페이지에 이 저장소 이름이 보이거나, 저장소 첫 화면 오른쪽 **Packages** 목록에 그 패키지가 보이면 연결된 것입니다. 연결돼 있지 않으면 패키지 페이지의 **Package settings → Connect repository**에서 연결한 뒤 새 tag로 다시 push하세요. 자세한 내용은 위 「6. 이미지를 저장소에 연결하기」에 있습니다.

**push했는데 release가 생기지 않아요.**
저장소의 Actions 탭에서 `AI System Challenge submission` 실행을 먼저 보세요. 실행이 없으면 패키지가 저장소에 연결돼 있지 않거나 Actions가 꺼져 있을 가능성이 큽니다. 실행이 있다면 로그에 건너뛴 이유(`latest`, 이미 release가 있는 tag, 등록하지 않은 이름)나 tag 규칙 오류가 남아 있습니다.

**Actions가 꺼져 있으면 어떻게 되나요?**
시스템은 Actions가 켜져 있는지 감지하지 못해서 등록은 통과할 수 있습니다. 하지만 꺼져 있으면 push해도 workflow가 실행되지 않아 release도 제출도 생기지 않습니다. **Settings → Actions → General**에서 켜세요.

**이미지를 public으로 해야 하나요?**
아니요. public 여부보다 저장소에 연결돼 있는지가 중요합니다.

**release 본문에 pull 주소를 꼭 적어야 하나요?**
아니요. release는 workflow가 자동으로 만들고, 시스템이 저장소에 연결된 패키지에서 release tag와 같은 tag를 찾습니다.

**push하는 tag에 규칙이 있나요?**
docker tag 규칙을 따릅니다. tag는 영문, 숫자, `_`, `.`, `-`만 쓰고 128자 이하여야 하며, 첫 글자는 영문, 숫자, `_` 중 하나입니다. 예: `v1.0.0`. 맞지 않으면 workflow가 오류로 멈추고 release를 만들지 않습니다.

**workflow 파일을 고쳐도 되나요?**
고치지 않는 것을 권장합니다. `AISC_IMAGE`는 바꾸지 마세요. 바꾸면 제출 이미지를 찾지 못합니다. 그 밖의 부분을 고치면 release가 제출 규칙대로 만들어지지 않을 수 있습니다. 다시 등록하면 파일은 공통 내용으로 되돌아갑니다.

**제출 workflow만 직접 추가하면 되나요?**
아니요. 등록 1회(Collaborator 초대, webhook, 자동 연결 테스트)는 따로 해야 합니다. 직접 추가하는 방법은 branch 보호 규칙이 commit을 막을 때만 씁니다.

**branch 보호 규칙이 있으면 어떻게 하나요?**
`AISystemChallenge`를 허용(bypass)하거나, 공개 템플릿을 직접 추가하세요. 자세한 방법은 위 「branch 보호 규칙이 commit을 막을 때」에 있습니다.

**이전에 등록한 팀은 어떻게 하나요?**
이전 안내(release를 발행하면 workflow가 이미지를 빌드하던 방식)로 등록한 팀의 저장소에는 이전 workflow 파일이 남아 있습니다. 같은 저장소와 이미지 이름으로 **다시 등록**하면 자동 연결 테스트가 파일을 새 내용으로 갱신합니다. 새 파일은 이미지를 빌드하지 않으므로, 이미지는 직접 빌드한 뒤 `docker push`하세요.
