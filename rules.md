---
title:
feature_text: |
  <!--# Samsung AI System Challenge 2026-->
feature_image: "/assets/logos/banner_AISC2026_13b.png"
excerpt:
---

### 평가 기준
사무국 평가 클라이언트가 사전 정의된 시나리오에 따라 참가팀 서버에 요청을 전송하고, **처리량(QPS), 응답 지연시간(SLO), 정확도(Accuracy)** 를 측정합니다.<br>
정확도와 응답 지연시간에 대한 최소 품질 기준을 충족한 참가팀을 대상으로 측정한 처리량(QPS) 을 기준으로 최종 순위를 결정합니다.
  * ##### 최종 순위 산정 기준
    * **핵심 평가 원칙:** 정확도와 응답 지연시간에 대한 최소 품질 기준을 보장하면서, 해당 조건을 충족하는 시스템 중 비공개 LLM serving 시나리오에서 가장 높은 처리량(QPS)을 달성한 참가팀을 우선 평가함.
    * 정확도 (Accuracy)
      * 각 Query의 정답 여부를 Pass / Fail로 판정
        * 객관식: 정답 일치 여부에 따라 Pass / Fail 판정
        * 서술형: F1 score 기준으로 사무국에서 설정한 임계값 이상인 경우 Pass 판정
      * 전체 Query 중 Pass로 판정 된 비율을 Accuracy Score로 산정
      * Accuracy Score가 95% 이상인 경우 정확도 기준 충족
    * 서비스 수준 목표 (SLO)
      * 응답 지연시간은 TTFT(Time To First Token)와 TPOT(Time Per Output Token)를 기준으로 평가
      * TTFT p95 및 TPOT p95가 각각 사전에 정의된 임계값 이하인 경우 SLO 기준 충족
      * 구체적인 임계값은 추후 확정 예정
    * 처리량 (QPS) 기반 Ranking
      * <u>Accuracy 및 SLO 기준을 모두 충족한 참가팀을 대상</u>으로 최종 순위 산정
      * QPS가 높은 순서대로 순위 부여
    * 순위 인정 조건 (모두 충족)
      * 오류 0건: HTTP 오류, 연결 끊김, Timeout, OOM(Out of Memory) 등 모든 오류가 발생하지 않아야 함.
      * 정확도 기준 충족: Accuracy Score ≥ 95%
      * SLO 기준 충족: TTFT p95 및 TPOT p95가 각각 설정된 임계값 이하
  * 모든 시간은 클라이언트 측 wall-clock으로 측정하며, 출력 토큰 수는 클라이언트 토크나이저로 계산합니다 (서버 usage 값 미사용).

<br>

### 평가 환경
참가팀은 서빙 서버를 Docker 이미지로 제출하고, 사무국 평가 서버가 아래 격리 조건에서 실행합니다. 서빙 프레임워크는 자유입니다 (vLLM, SGLang, llama.cpp, 직접 구현 등).
  * ##### 하드웨어 / 격리
    * GPU 1장: NVIDIA H100 80GB (TBD)
    * **GPU 메모리 사용 상한: XX GiB (모델 weight 포함, TBD)**
    * CPU 8코어 고정 (cpuset), Host 메모리 128 GiB, shm 16 GiB
    * 인터넷 차단, 평가 클라이언트 접속만 허용, 컨테이너 권한 제한 (--cap-drop ALL)
    * 모델: Qwen3-8B (BF16). /model 에 읽기 전용으로 마운트
    * 기동 제한 15분: /v1/models 응답과 1토큰 생성이 모두 성공해야 준비 완료로 판정

  * ##### 인터페이스 규약
    * 이미지 ENTRYPOINT로 서버 기동 (추가 인자 없음), 0.0.0.0:8000
    * POST /v1/completions : stream=true (SSE) 필수
    * GET /v1/models, 모델 이름 submission
    * 환경변수 MODEL_PATH, SERVED_MODEL_NAME, PORT 제공
    * 요청은 채팅 템플릿이 이미 적용된 raw prompt이며, temperature=0, max_tokens가 지정되어 들어옵니다.

<br>
   
### 벤치마크
평가 클라이언트(bench.py)와 예제 시나리오(scenarios/en_sum_sub.json)는 GitHub repository로 제공됩니다.<br>
**※ 본 평가는 비공개 seed·시나리오를 사용하므로, 공개 시나리오로 측정한 점수는 리더보드 점수와 다를 수 있습니다.**<br><br>
제공되는 샘플 시나리오는 scenarios/en_sum_sub.json입니다. InfiniteBench En.Sum 데이터(ids로 고정)로 ~수십만 토큰 길이의 요청을 만들어 보냅니다.
  * ##### 태스크
    * en_sum
      * 요약, 출력 ≤ 1,200 토큰
      * ROUGE-Lsum F1이 임계값 이상이면 정답
    * en_sum_next
      * 인용 구절 뒤에 이어지는 문장을 보기 4개(A~D)에서 고르기, 출력 ≤ 16 토큰
      * 보기 글자 일치로 채점
    * en_sum_blank
      * 인용 문장에서 ____로 가려진 고유명사 한 단어, 출력 ≤ 16 토큰
      * 단어 일치로 채점
  * ##### 부하 스케줄
    * 요청 방식 (Open-loop): 이전 요청의 응답 완료 여부와 관계없이, 사전에 정의된 시각에 따라 독립적으로 요청을 전송하는 Open-loop 방식 적용.

<br>

### 제한 사항
**아래의 제한 사항을 위반하면 실격처리 됩니다.**
  * ##### 허용
    * 서빙 프레임워크의 선택·수정 (스케줄러, 메모리 관리자, attention 백엔드), 커스텀 CUDA/Triton 커널
    * KV cache 관리 기법 일체: paging, prefix caching, CPU offloading, KV 양자화, eviction, chunked prefill, admission control 등 (단, 정확도 조건 충족 필요)
    * 오픈소스 라이브러리 사용 (LMCache, FlashInfer 등). 출처와 라이선스 명기 필요
  * ##### 금지
    * 지정 모델 교체, 추가 학습·증류, 별도 draft/proxy 모델 도입
    * **모델 weight 정밀도 변경 (BF16 고정)**
    * 평가 문제에 의존한 사전 계산: 응답 캐싱, 사전 계산한 KV를 이미지에 포함 (요청 처리 중의 prefix caching은 허용)
    * 요청 파라미터(prompt, max_tokens, temperature) 무시·변경, 스트리밍 타이밍 위조 (더미 첫 토큰 등)
    * GPU 메모리 상한 우회: UVM/managed memory, 별도 프로세스에 메모리 은닉 등
    * 다중 GPU·외부 노드·네트워크 사용, 평가 클라이언트·하네스 조작

<br>

### 대회 규칙
**이번 대회에서는 공정한 평가 및 재현 가능한 결과를 보장하는 것을 목표로 합니다. 이를 위해 다음 사항을 준수해야 합니다.**

##### 1. 평가 환경 및 재현성
제출물은 주최 측이 지정한 하드웨어 및 소프트웨어 환경에서 정상적으로 실행되고 평가 결과가 재현 가능해야 합니다. 재현에 실패한 제출물의 평가 결과는 무효 처리됩니다.

##### 2. 평가 자료 제출
참가팀은 평가 및 검증을 위해 주최 측이 요청하는 소스 코드, 실행 방법, 환경 설정 등 필요한 자료를 제출해야 합니다. 요청된 자료를 제출하지 않을 경우 해당 제출물의 평가 결과는 무효 처리됩니다.

##### 3. 제출물 비공개 및 활용
주최 측은 참가팀의 제출물 및 소스 코드를 외부에 공개하지 않으며, 대회 평가, 결과 재현 및 부정행위 검증 목적으로만 활용합니다. 참가팀의 별도 동의 없이 상업적 목적으로 사용하지 않으며, 평가 및 검증이 완료된 후 삭제합니다.

##### 4. 외부 오픈소스 및 지식재산권
공개된 오픈소스, 라이브러리 및 외부 구현은 관련 라이선스와 이용 조건을 준수하는 범위에서 사용할 수 있으며, 사용한 외부 구현의 출처를 명시해야 합니다. 참가팀은 제출물에 사용된 자료 및 기술에 대해 적법한 사용 권한을 확보해야 하며, 제출물로 인해 제3자의 지식재산권 침해가 발생하는 경우 이에 따른 모든 법적 책임은 해당 참가팀에 있으며, 주최 측은 이에 대한 책임을 지지 않습니다.

##### 5. 참가팀 간 코드 및 구현 공유 금지
참가팀은 대회 기간 중 자신의 소스 코드, 구현 내용 및 제출물을 다른 참가팀에 공개하거나 공유할 수 없으며, 다른 참가팀의 구현을 무단 복제하는 행위도 금지됩니다. 단, 공개된 논문, 오픈소스 및 기술 자료의 활용은 허용됩니다.

##### 6. 참가 자격 및 팀 구성
참가팀은 최대 3명으로 구성할 수 있으며, 참가자는 하나의 팀에만 소속될 수 있습니다. 동일인이 복수의 팀에 중복 참가하는 것은 허용되지 않습니다.

##### 7. 대회 관계자 참가 제한
대회 운영 및 평가에 직접 관여하는 주최 측 관계자는 참가할 수 없습니다.

##### 8. 평가 시스템 우회 및 부정행위 금지
평가 프로그램 또는 성능 측정 결과를 무단으로 변경하거나, 실제 추론을 대체하는 등 평가 결과를 부당하게 왜곡하는 행위를 금지합니다. <br>
- **특히 다음 행위는 허용되지 않습니다.**
  - 평가 입력에 대한 출력값을 사전에 준비하거나 하드코딩하여 실제 추론을 대체하는 행위
  - 평가 시간, GPU 메모리 사용량 등 성능 측정값을 조작하는 행위
  - 평가 프로그램 또는 측정 로직을 무단으로 변경하거나 우회하는 행위
  - 허용되지 않은 외부 GPU, API 등 연산 자원을 사용하는 행위

##### 9. 부정행위 및 실격
상기 규칙을 위반하거나, 명시된 규칙 외에도 평가 결과 조작, 부정한 이익 취득 등 대회의 공정성과 신뢰성을 훼손하는 행위가 확인된 경우 실격 처리될 수 있습니다.

##### 10. 리더보드 및 최종 평가
리더보드에 표시된 점수와 순위는 잠정 결과이며, 최종 평가 결과는 제출물의 재현성 및 규정 준수 여부를 검증한 후 확정됩니다.

##### 11. 대회 규칙 변경 및 공지
주최 측은 공정한 대회 운영을 위해 필요한 경우 규칙 및 평가 절차를 변경할 수 있으며, 변경 사항과 적용 시점은 대회 공식 홈페이지를 통해 공지합니다.

<hr />

### Evaluation Criteria
The organizing committee's evaluation client sends requests to participating teams' servers according to predefined scenarios and measures **throughput (QPS), response latency (SLO), and accuracy**.<br>
Final rankings are determined by the measured throughput (QPS) among participating teams that meet the minimum quality requirements for accuracy and response latency.
  * ##### Final Ranking Criteria
    * **Core Evaluation Principle:** Priority is given to participating teams that achieve the highest throughput (QPS) under undisclosed LLM serving scenarios while satisfying the minimum quality requirements for accuracy and response latency.
    * Accuracy
      * Each query is evaluated as Pass or Fail based on the correctness of its response.
        * Multiple-choice questions: Pass or Fail is determined by whether the response matches the correct answer.
        * Open-ended questions: A response is considered Pass if its F1 score meets or exceeds the threshold defined by the organizing committee.
      * The Accuracy Score is calculated as the percentage of all queries evaluated as Pass.
      * The accuracy requirement is satisfied when the Accuracy Score is at least 95%.
    * Service Level Objectives (SLO)
      * Response latency is evaluated using TTFT (Time to First Token) and TPOT (Time per Output Token).
      * The SLO requirement is satisfied when both TTFT p95 and TPOT p95 are at or below their respective predefined thresholds.
      * Specific threshold values will be determined later.
    * Throughput (QPS) Based Ranking
      * Final rankings are determined only among participating <u>teams that satisfy both the Accuracy and SLO requirements</u>.
      * Teams are ranked in descending order of QPS.
    * Ranking Eligibility Requirements (all must be satisfied)
      * Zero errors: No errors of any kind, including HTTP errors, connection failures, timeouts, or OOM (Out of Memory), are permitted.
      * Accuracy requirement: Accuracy Score ≥ 95%.
      * SLO requirement: Both TTFT p95 and TPOT p95 must be at or below their respective predefined thresholds.
  * All timing measurements are based on client-side wall-clock time, and output token counts are calculated using the client-side tokenizer (server-reported usage values are not used).
<br>

### Evaluation Environment
Participating teams submit their serving servers as Docker-images, which are executed by the organizing committee's evaluation server under the following isolated conditions. Teams may freely choose their serving framework (vLLM, SGLang, llama.cpp, custom implementations, etc.).
  * ##### Hardware / Isolation
    * One GPU: NVIDIA H100 80GB (TBD)
    * **GPU memory usage limit: XX GiB (including model weights, TBD)**
    * 8 dedicated CPU cores (cpuset), 128 GiB host memory, 16 GiB shm
    * Internet access disabled, only evaluation-client connections permitted, restricted container privileges (--cap-drop ALL)
    * Model: Qwen3-8B (BF16), mounted read-only at /model
    * 15-minute startup limit: both a successful /v1/models response and successful one-token generation are required for readiness

  * ##### Interface Requirements
    * Start the server using the image ENTRYPOINT (no additional arguments), listening on 0.0.0.0:8000
    * POST /v1/completions : stream=true (SSE) required
    * GET /v1/models, model name submission
    * Environment variables MODEL_PATH, SERVED_MODEL_NAME, PORT are provided
    * Requests contain raw prompts with the chat template already applied, with temperature=0 and max_tokens specified.

   
### Benchmark
The evaluation client (bench.py) and example scenario (scenarios/en_sum_sub.json) are provided through a GitHub repository.<br>
**※ This evaluation uses private seeds and scenarios, so scores measured using public scenarios may differ from leaderboard scores.**<br><br>
The provided sample scenario is scenarios/en_sum_sub.json. It generates and sends requests with lengths of approximately up to hundreds of thousands of tokens using the InfiniteBench En.Sum dataset (fixed ids).
  * ##### Tasks
    * en_sum
      * Summarization, output ≤ 1,200 tokens
      * A response is considered correct if its ROUGE-Lsum F1 score meets or exceeds the threshold.
    * en_sum_next
      * Select the sentence following a quoted passage from four options (A–D), output ≤ 16 tokens
      * Evaluated by matching the option letter.
    * en_sum_blank
      * A single-word proper noun masked with ____ in a quoted sentence, output ≤ 16 tokens
      * Evaluated by word matching.
  * ##### Load Schedule
    * Request Method (Open-loop): Requests are sent independently at predefined times, regardless of whether previous requests have completed.


### Restrictions
**Violations of the following restrictions will result in disqualification.**
  * ##### Permitted
    * Selection and modification of serving frameworks (schedulers, memory managers, attention backends), custom CUDA/Triton kernels
    * All KV cache management techniques: paging, prefix caching, CPU offloading, KV quantization, eviction, chunked prefill, admission control, etc. (accuracy requirements must be satisfied)
    * Use of open-source libraries (LMCache, FlashInfer, etc.). Sources and licenses must be acknowledged
  * ##### Prohibited
    * Replacing the designated model, additional training or distillation, introducing separate draft/proxy models
    * **Changing model weight precision (BF16 must be maintained)**
    * Precomputation based on evaluation problems: response caching or including precomputed KV caches in the image (prefix caching during request processing is permitted)
    * Ignoring or modifying request parameters (prompt, max_tokens, temperature), falsifying streaming timing (dummy first tokens, etc.)
    * Circumventing GPU memory limits: UVM/managed memory, concealing memory usage in separate processes, etc.
    * Using multiple GPUs, external nodes, or networks; manipulating the evaluation client or harness
<br>

### Competition Rules
**This competition aims to ensure fair evaluation and reproducible results. All participants must comply with the following rules.**

##### 1. Evaluation Environment and Reproducibility
Submissions must run properly in the hardware and software environment specified by the organizer, and their evaluation results must be reproducible. Evaluation results that cannot be reproduced will be invalidated.

##### 2. Submission of Evaluation Materials
Participating teams must provide all materials requested by the organizer for evaluation and verification, including source code, execution instructions, and environment configurations. Failure to provide the requested materials will result in the invalidation of the corresponding submission's evaluation results.

##### 3. Confidentiality and Use of Submissions
The organizer will not disclose participating teams' submissions or source code to third parties and will use them solely for competition evaluation, result reproduction, and misconduct verification. Submissions will not be used for commercial purposes without the participating team's separate consent and will be deleted after evaluation and verification are completed.

##### 4. External Open Source and Intellectual Property Rights
Publicly available open-source software, libraries, and third-party implementations may be used in compliance with their applicable licenses and terms of use. Teams must properly acknowledge the sources of any external implementations used. Participating teams are responsible for obtaining all necessary rights and permissions for materials and technologies included in their submissions. If a submission infringes upon a third party's intellectual property rights, the participating team shall bear full legal responsibility for the infringement, and the organizer shall not be held liable.

##### 5. Prohibition of Code and Implementation Sharing Between Teams
During the competition, participating teams must not disclose or share their source code, implementations, or submissions with other participating teams. Unauthorized copying of another team's implementation is also prohibited. However, the use of publicly available research papers, open-source software, and technical resources is permitted.

##### 6. Eligibility and Team Composition
Each team may consist of up to three members, and each participant may belong to only one team. Participation in multiple teams by the same individual is not permitted.

##### 7. Restrictions on Participation by Competition Personnel
Individuals representing the organizer who are directly involved in competition operations or evaluation are not eligible to participate.

##### 8. Prohibition of Evaluation System Circumvention and Misconduct
Unauthorized modification of evaluation programs or performance measurements, substitution of actual inference, or any other actions that improperly distort evaluation results are prohibited. <br>
- **In particular, the following actions are not permitted.**
  - Precomputing or hardcoding outputs for evaluation inputs to replace actual inference
  - Manipulating performance measurements, including evaluation time and GPU memory usage
  - Unauthorized modification or circumvention of evaluation programs or measurement logic
  - Using unauthorized computational resources, including external GPUs or APIs

##### 9. Misconduct and Disqualification
Participants may be disqualified for violating the above rules or engaging in any other conduct that undermines the fairness and integrity of the competition, including manipulation of evaluation results or obtaining unfair advantages, even if such conduct is not explicitly covered by the stated rules.

##### 10. Leaderboard and Final Evaluation
Scores and rankings displayed on the leaderboard are provisional. Final evaluation results will be confirmed only after verifying the reproducibility of submissions and their compliance with competition rules.

##### 11. Changes to Competition Rules and Announcements
The organizer reserves the right to modify competition rules and evaluation procedures when necessary to ensure fair competition. Any changes and their effective dates will be announced on the official competition website.
