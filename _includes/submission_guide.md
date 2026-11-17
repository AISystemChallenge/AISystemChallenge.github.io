# 대회 제출 안내: GitHub release와 GHCR 이미지

참가 팀은 **등록을 한 번** 마치면, 그 뒤로는 **GitHub release를 publish할 때마다** 제출 알림이 운영팀에 자동으로 전달됩니다. 이 문서는 참가자가 준비할 것과 시스템이 확인하는 것을 설명합니다.

## 1. 전체 흐름

1. **등록 1회**: 대회 공지의 등록 링크에서 팀 정보를 입력하고, 안내 화면대로 저장소를 설정한 뒤 자동 연결 테스트를 통과합니다.
2. **release마다 자동 알림**: release를 publish하면 시스템이 이미지를 찾아 확인하고 운영팀에 알립니다.

## 2. 참가자 준비물

| 준비물 | 내용 |
|---|---|
| GitHub 저장소 | commit이 하나 이상 있어야 합니다. 빈 저장소(commit 0)는 등록할 수 없습니다. |
| Collaborator | `AISystemChallenge`를 **Write 이상** 권한으로 초대합니다. 조직 저장소는 **Collaborators and teams**에서 초대합니다. 초대는 시스템이 자동으로 수락합니다. |
| Webhook | 이벤트는 **Releases만** 선택합니다. Payload URL과 Secret은 등록 화면에서 복사합니다. |
| GHCR 이미지 | 등록 저장소와 **같은 계정**의 `ghcr.io/<owner>/…`에 있고, 그 저장소에 **연결**돼 있어야 합니다. |

### 등록 절차

1. 등록 Form에 팀 정보, GitHub 저장소, GHCR 이미지 주소를 입력합니다. 이미지 주소에는 tag 또는 digest가 있어야 하고(예: `ghcr.io/<owner>/<image>:v1.0.0`), 등록 저장소와 같은 계정이어야 합니다.
2. 안내 화면에서 아래 세 가지를 합니다.
   - **Collaborator 초대**: 저장소에 `AISystemChallenge`를 Write 이상으로 초대합니다.
   - **Webhook 추가**: 저장소 **Settings → Webhooks → Add webhook**에서 아래처럼 설정합니다.
     - **Payload URL**과 **Secret**: 안내 화면의 값을 그대로 복사해 붙여넣습니다.
     - **Content type**: `application/json`
     - **SSL verification**: 활성(Enable SSL verification)
     - **이벤트**: `Let me select individual events`를 선택하고 **Releases만** 체크합니다. 기본으로 체크된 **Pushes는 해제**합니다.
     - **Active**: 체크
   - **이미지 연결**: 이미지가 등록 저장소에 연결돼 있게 합니다(아래 「이미지 연결 방법」).
3. 「설정 완료 · 자동 연결 테스트 시작」을 누릅니다. 테스트 중에는 새로고침하거나 창을 닫지 마세요.
4. 시스템이 초대를 수락하고, 쓰기 권한을 확인하고, 이미지가 저장소에 연결돼 있는지와 tag가 있는지 확인합니다. 그 뒤 임시 draft release를 만들어 webhook이 도착하는지 확인하고 draft를 삭제합니다. 임시 draft release는 공개되지 않고, tag를 만들지 않으며, Actions도 실행하지 않습니다. 모두 통과하면 등록이 완료됩니다.

Payload URL과 Secret은 **저장소마다 고정**입니다. 같은 저장소를 다시 등록해도 값이 같고, 이미 대회 webhook이 있다면 그대로 두면 됩니다. webhook은 **하나면 충분**합니다. 여러 개를 만들면 같은 알림이 그 수만큼 전달됩니다.

### 원클릭 복사 화면

등록 화면의 「원클릭 복사 화면 열기」 버튼을 누르면 `https://aisystemchallenge.github.io/submission-guide/copy/`가 새 탭으로 열립니다. 이 화면에서 Collaborator 사용자명, Payload URL, Secret을 버튼 하나로 복사할 수 있습니다.

- 값은 주소의 `#` 뒤에 담겨 전달되며, `#` 뒤쪽은 서버로 전송되지 않습니다.
- 이 화면은 대회 공식 형식의 값만 표시합니다. 경고가 보이면 그 값을 GitHub에 입력하지 말고, 등록 화면의 버튼으로 다시 여세요.
- 등록 화면의 버튼으로만 여세요. 주소만 직접 열면 값이 비어 있습니다.

### 이미지 연결 방법

이미지를 등록 저장소에 연결하는 방법은 세 가지입니다. 하나만 하면 됩니다.

1. **(권장) release 때 GitHub Actions로 push**: workflow에서 `GITHUB_TOKEN`으로 `ghcr.io`에 push하면 이미지가 그 저장소에 자동으로 연결됩니다. 예시는 [release.yml](examples/release.yml)입니다.
2. **Dockerfile에 label 추가**: `LABEL org.opencontainers.image.source=https://github.com/<owner>/<repo>`를 넣고 push합니다.
3. **패키지 페이지에서 직접 연결**: 패키지 페이지의 **Package settings → Connect repository**에서 저장소를 선택합니다.

## 3. release 때 일어나는 일

1. 참가자가 release를 **publish**합니다. pre-release도 포함됩니다. draft 저장, 수정, 삭제는 알림이 없습니다.
2. 시스템이 webhook을 받고, 서명과 저장소를 확인합니다.
3. 저장소에 연결된 GHCR 패키지 중 **release tag와 같은 tag**의 버전을 GitHub를 통해 조회합니다. release 본문에 이미지 주소를 적지 않아도 됩니다.
4. release 본문에 `ghcr.io/…` 주소를 적었다면, 그 주소도 같은 규칙으로 확인합니다.
5. 운영팀에 알림을 보냅니다. 알림에는 pull 주소와 digest가 들어갑니다.

이미지가 아직 없으면 약 5분 뒤 한 번 더 확인합니다. release를 publish한 직후 이미지 push가 끝나지 않았어도 괜찮습니다.

### 이미지 규칙

**같은 계정 + 저장소에 연결 + tag 존재**, 세 가지를 모두 만족해야 합니다.

- 같은 계정은 release를 누른 개인이 아니라 **저장소를 소유한 계정**을 기준으로 합니다. 그래서 팀원 누구나 release를 할 수 있습니다.
- 연결은 이미지가 그 저장소의 패키지로 묶여 있다는 뜻입니다(위 「이미지 연결 방법」).

## 4. digest란?

digest는 이미지 manifest의 `sha256` 지문(content address)입니다. tag(`v1.0.0` 같은 이름)는 나중에 다른 빌드로 옮겨갈 수 있지만, digest는 그 빌드 하나에 고정됩니다. 그래서 알림에는 digest가 함께 들어가고, 아래처럼 정확히 그 빌드를 받을 수 있습니다.

```
docker pull ghcr.io/<owner>/<image>@sha256:…
```

multi-arch 이미지는 index의 digest를 씁니다.

## 5. 거부되는 경우

아래 경우에는 이미지가 거부됩니다. 거부된 주소는 운영팀 알림에 **경고로만** 표시되고 pull 대상에서 빠집니다.

- 다른 계정의 이미지
- 저장소에 연결되지 않은 패키지. 개인에게 직접 읽기 권한을 준 것만으로는 연결로 인정하지 않습니다.
- tag가 없는 경우. 5분 뒤 재확인한 뒤에도 없을 때입니다.
- 조회에 실패한 경우

## 6. FAQ

**release.yml만 있으면 되나요?**
아니요. 등록 1회(collaborator 초대, webhook)는 따로 해야 합니다. release.yml은 release 때 이미지 빌드, push, 저장소 자동 연결을 맡습니다.

**이미지를 public으로 해야 하나요?**
아니요. public 여부보다 저장소에 연결돼 있는지가 중요합니다.

**release 본문에 pull 주소를 꼭 적어야 하나요?**
아니요. 시스템이 저장소에 연결된 패키지에서 release tag와 같은 tag를 찾습니다.

**release tag 규칙이 있나요?**
docker tag 규칙을 따릅니다. 영문, 숫자, `_`, `.`, `-`만 쓰고 128자 이하여야 합니다. 예: `v1.0.0`
