---
title:
feature_text: |
  <!--# Samsung AI System Challenge 2026-->
feature_image: "/assets/logos/banner_AISC2026_20.png"
excerpt:
---

### 참가 등록
* ##### Samsung AI System Challenge 참가하시려면, Submssion 페이지에서 참가 신청서를 작성해주시기 바랍니다. 
  ※ 참가 접수는 운영 상황에 따라 조기 마감될 수 있습니다.
  <!-- * <a href="https://aisystemchallenge.app.n8n.cloud/form/register">Submssion 페이지로 이동</a> -->
  * <a href="https://flat-river-ab49.samsung-aisc.workers.dev/form/register">Submssion 페이지로 이동</a>
<hr>

### 참가 자격
* ##### 2026년 12월 1일 기준 국내 대학에 재학 중인 대학생 및 대학원생 (만 14세 이상)
  ※ 수상은 아래 조건을 충족해야 합니다. (‘26년 12월 1일 기준)
  * 학적 상태가 재학 상태이어야 하고, 졸업유예생은 참여불가.
  * 산업체, 연구소 등에 재직 중인 사람은 수상 불가 (학술 연수중인 사람, 산업정부출연연구소 소속 학생연구원 포함).
  * 최종 수상후보 팀은 재학증명서 또는 대체인증서류 제출 필수.
* ##### 개인 또는 팀으로 참여 가능하며, 팀 당 최대 3명까지 구성.
  ※ 개인/팀은 1개 결과물만 제출 가능 (동일인이 개인 또는 복수팀에 중복하여 참여 불가).
<hr>

### 대회 운영
* 본 대회는 각 팀이 제출한 AI Serving Framework 소스코드를 기반으로, 사무국에서 지정한 벤치마크 환경에서의 추론 처리량(QPS) 극대화를 목표로 합니다.
* 사무국이 정한 SLO와 Baseline 대비 95% 이상의 정확도를 모두 충족한 경우에 한하여 유효한 평가 결과로 인정하며, QPS가 높은 순서로 상위 8개 팀을 선발합니다.
* 각 팀의 평가 결과는 2026년 12월 10일부터 리더보드(Leaderboard)를 통해 공개되며, 주기적으로 업데이트될 예정입니다.
* 동일한 팀이 소스코드를 여러 차례 제출한 경우에는 유효한 평가 결과 중 가장 높은 QPS를 해당 팀의 기록으로 반영합니다.
※ 소스코드는 팀당 1일 최대 1회 제출할 수 있습니다.
※ 대회 평가규칙, 제약사항은 <a href="https://aisystemchallenge.github.io/Rules/">Rules</a>를 참고해주세요.
* 리더보드 운영
  * 필수 조건 : 아래 조건을 모두 충족해야 유효한 평가 결과로 인정하며, 하나라도 충족하지 못하면 FAIL 처리됩니다.
    * SLO: 사무국에서 정한 응답시간 기준(xx ms) 충족
    * 정확도: Baseline 대비 95% 이상 유지
  * 순위 산정
    * 필수 조건을 충족한 팀을 대상으로 QPS가 높은 순서대로 순위 산정
    * 리더보드에는 상위 8개 팀의 기록 표시
    * QPS가 동일한 경우 해당 기록을 먼저 제출한 팀에 우선순위 부여
* 최종 평가
  * 최종 평가 대상 선정 및 검증
    * 리더보드 상위 8개 팀을 최종 평가 대상으로 선정
    * 선정된 팀의 소스코드 및 문제풀이 설명 문서를 제출받아 구현 내용 검증 및 추론 성능 실측평가 진행
  * 오프라인 발표 및 최종 순위 결정
    * 상위 8개 팀을 대상으로 최적화 기법 및 구현 내용에 대한 오프라인 발표 진행
    * 발표 내용을 바탕으로 기술적 우수성 및 구현 완성도 등을 정성 평가하여 최종 순위 결정
<hr>

### 성능 평가 가이드
  * Private repository의 main branch에 push된 latest commit을 기준으로 <a href="https://aisystemchallenge.github.io/leaderboard/">Leaderboard</a>에 주기적으로 채점 결과가 반영됩니다.
  * Leaderboard 제출은 1일 1회까지 한정됩니다. 1회를 초과하는 경우 채점이 진행되지 않으며 에러 발생도 채점 횟수에 포함됩니다. Main branch에 commit을 push 하면 자동으로 제출로 간주됩니다.
  * Leaderboard의 제출 횟수는 0시(KST)를 기준으로 초기화 됩니다.
  * 최종 평가는 대회 종료 일시인 2027년 1월 2일(토) 23:59:59 이전 main branch의 가장 최신 commit을 기준으로 진행됩니다.
<hr>

### Registration
* ##### To participate in the Samsung AI System Challenge, please complete the registration form on the Submission page.
  ※ Registration may close early depending on operational circumstances.
  * <a href="https://flat-river-ab49.samsung-aisc.workers.dev/form/register">Go to the Submission Page</a>
<hr>

### Competition Evaluation
* The competition aims to maximize inference throughput (QPS) in the benchmark environment designated by the organizers, based on the AI Serving Framework source code submitted by each team.
* Only results that meet both the SLO requirements set by the organizers and an accuracy level of at least 95% relative to the Baseline will be considered valid. The top eight teams will be selected in descending order of QPS.
* Evaluation results will be available on the Leaderboard starting December 10, 2026, and will be updated periodically.
* If a team submits its source code multiple times, its highest QPS among valid evaluation results will be used as the team's record.
※ Each team may submit its source code at most once per day.
※ For competition evaluation rules and restrictions, please refer to <a href="https://aisystemchallenge.github.io/Rules/">Rules</a>.
* Leaderboard Evaluation
  * Requirements: All of the following requirements must be met for an evaluation result to be considered valid. Failure to meet any requirement will result in a FAIL.
    * SLO: Meet the response time requirement (xx ms) specified by the organizers.
    * Accuracy: Maintain at least 95% accuracy relative to the Baseline.
  * Ranking
    * Teams meeting all requirements will be ranked in descending order of QPS.
    * The Leaderboard will display the top eight teams.
    * In the event of a QPS tie, priority will be given to the team that submitted the result earlier.
* Final Evaluation
  * Selection and Verification
    * The top eight teams on the Leaderboard will be selected for the final evaluation.
    * Selected teams will be required to submit their source code and a document explaining their solution for implementation verification and actual inference performance measurement.
  * Offline Presentations and Final Ranking
    * The top eight teams will give offline presentations on their optimization techniques and implementation details.
    * Final rankings will be determined through a qualitative evaluation of technical excellence and implementation quality based on the presentations.
<hr>

### Eligibility
* ##### Undergraduate and graduate students currently enrolled at universities in South Korea as of December 1, 2026 (aged 14 or older)
  ※ To be eligible for an award, participants must meet the following requirements (as of Dec. 1, 2026).
  * Must be currently enrolled; students on graduation deferment are not eligible to participate.
  * Those currently employed by an industry or research institute are not eligible for awards (including those on academic training and student researchers affiliated with industry-government-funded research institutes).
  * Finalist teams must submit proof of enrollment or alternative certification documents.
* ##### Individual participation or teams of up to three members are allowed.
  ※ Individuals/teams may submit only one entry (the same person cannot participate individually or as a member of multiple teams).

<hr>

### Evaluation Submission Guidelines
  * Leaderboard results are periodically updated based on the latest commit pushed to the main branch of the private repository. Results are available on the <a href="https://aisystemchallenge.github.io/leaderboard/">Leaderboard</a>.
  * Leaderboard submissions are limited to once per day. Submissions exceeding this limit will not be evaluated, and errors also count toward the submission limit. Pushing a commit to the main branch is automatically considered a submission.
  * The Leaderboard submission count resets daily at 00:00 KST.
  * The final evaluation will be based on the latest commit on the main branch before the competition ends on January 2, 2027 (Sat), at 23:59:59 KST.
