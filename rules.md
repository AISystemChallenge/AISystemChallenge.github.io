---
title:
feature_text: |
  <!--# Samsung AI System Challenge 2026-->
feature_image: "/assets/logos/banner_AISC2026_13b.png"
excerpt:
---

### 평가 기준
사무국 평가 클라이언트가 참가팀 서버에 정해진 시나리오의 요청을 보내 처리량·지연시간·정확도를 측정하며, 최대 부하 구간(saturation)의 처리량으로 순위를 정합니다.
  * &lt;상세&gt;
    * **평가 산식 : Score = saturation 스테이지 req/s** <br>
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;= 성공 요청 수 / (스테이지 시작 ~ 마지막 응답 완료 시간)
    * **순위 인정 조건 (모두 충족)** <br>
      ① 오류 0건 (HTTP 오류, 연결 끊김, 600초 타임아웃 모두 오류로 집계)<br>
      ② 전체 정확도 ≥ Baseline 정확도 − 0.02
    * Baseline: vLLM v0.32.0 기본 설정 (제공 예제 team_baseline)
    * 참고 지표 (순위 미반영): TTFT·TPOT p50/p99, 출력 tok/s, 태스크별 정확도
    * 모든 시간은 클라이언트 측 wall-clock으로 측정하며, 출력 토큰 수는 클라이언트 토크나이저로 계산합니다 (서버 usage 값 미사용).
<br>

### 평가 환경
참가팀은 서빙 서버를 Docker 이미지로 제출하고, 사무국 평가 서버가 아래 격리 조건에서 실행합니다. 서빙 프레임워크는 자유입니다 (vLLM, SGLang, llama.cpp, 직접 구현 등).
  * &lt;하드웨어 / 격리&gt;
    * GPU 1장: NVIDIA H100 80GB (TBD)
    * **GPU 메모리 사용 상한: XX GiB (모델 weight 포함, TBD)**
    * CPU 8코어 고정 (cpuset), Host 메모리 128 GiB, shm 16 GiB
    * 인터넷 차단, 평가 클라이언트 접속만 허용, 컨테이너 권한 제한 (--cap-drop ALL)
    * 모델: Qwen3-8B (BF16). /model 에 읽기 전용으로 마운트
    * 기동 제한 15분: /v1/models 응답과 1토큰 생성이 모두 성공해야 준비 완료로 판정

  * &lt;인터페이스 규약&gt;
    * 이미지 ENTRYPOINT로 서버 기동 (추가 인자 없음), 0.0.0.0:8000
    * POST /v1/completions : stream=true (SSE) 필수
    * GET /v1/models, 모델 이름 submission
    * 환경변수 MODEL_PATH, SERVED_MODEL_NAME, PORT 제공
    * 요청은 채팅 템플릿이 이미 적용된 raw prompt이며, temperature=0, max_tokens가 지정되어 들어옵니다.
<br>
   
### 벤치마크
평가 클라이언트(bench.py)와 예제 시나리오는 GitHub repository로 제공됩니다. 문제는 평가 시점마다 seed로 새로 생성하므로 정답을 외우거나 응답을 캐싱할 수 없습니다.<br>
※ **본 평가는 비공개 seed·시나리오를 사용하므로, 공개 시나리오로 측정한 점수는 리더보드 점수와 다를 수 있습니다.**
  * &lt;태스크&gt;
    * kv_retrieval
        * 약 3.1k 토큰의 key-value 목록에서 값 찾기, 출력 ≤ 24 토큰
        * 문서 하나를 질문 4개가 공유해 prefix가 재사용됨
    * arith_chain
      * 약 110 토큰의 8단계 산술 풀이, 출력 ≤ 768 토큰
      * 마지막 ANSWER: <n> 으로 채점
    * longbench_v2
      * LongBench-v2 장문 4지선다, 컨텍스트를 자르지 않고 전송 (수k ~ 1M 토큰), 출력 ≤ 128 토큰
      * The correct answer is (X) 로 채점
  * &lt;부하 스케줄&gt;
    * open-loop: 응답과 무관하게 미리 정한 시각에 전송
    * low / mid / high: Poisson 2 / 8 / 24 req/s, 각 60 / 240 / 720건
    * saturation: 앞 요청이 모두 끝난 뒤 600건 동시 투입
<br>

### 제한 사항
**아래의 제한 사항을 위반하면 실격처리 됩니다.**
  * &lt;허용&gt;
    * 서빙 프레임워크의 선택·수정 (스케줄러, 메모리 관리자, attention 백엔드), 커스텀 CUDA/Triton 커널
    * KV cache 관리 기법 일체: paging, prefix caching, CPU offloading, KV 양자화, eviction, chunked prefill, admission control 등 (단, 정확도 조건 충족 필요)
    * 오픈소스 라이브러리 사용 (LMCache, FlashInfer 등). 출처와 라이선스 명기 필요
  * &lt;금지&gt;
    * 지정 모델 교체, 추가 학습·증류, 별도 draft/proxy 모델 도입
    * **모델 weight 정밀도 변경 (BF16 고정)**
    * 평가 문제에 의존한 사전 계산: 응답 캐싱, 사전 계산한 KV를 이미지에 포함 (요청 처리 중의 prefix caching은 허용)
    * 요청 파라미터(prompt, max_tokens, temperature) 무시·변경, 스트리밍 타이밍 위조 (더미 첫 토큰 등)
    * GPU 메모리 상한 우회: UVM/managed memory, 별도 프로세스에 메모리 은닉 등
    * 다중 GPU·외부 노드·네트워크 사용, 평가 클라이언트·하네스 조작
<br>

### 대회 규칙
**이번 대회에서는 공정한 평가 및 재현 가능한 결과를 보장하는 것을 목표로 합니다. 이를 위해 다음 사항을 준수해야 합니다.**

#### 1. 평가 환경 및 재현성
제출물은 주최 측이 지정한 하드웨어 및 소프트웨어 환경에서 정상적으로 실행되고 평가 결과가 재현 가능해야 합니다. 재현에 실패한 제출물의 평가 결과는 무효 처리됩니다.

#### 2. 평가 자료 제출
참가팀은 평가 및 검증을 위해 주최 측이 요청하는 소스 코드, 실행 방법, 환경 설정 등 필요한 자료를 제출해야 합니다. 요청된 자료를 제출하지 않을 경우 해당 제출물의 평가 결과는 무효 처리됩니다.

#### 3. 제출물 비공개 및 활용
주최 측은 참가팀의 제출물 및 소스 코드를 외부에 공개하지 않으며, 대회 평가, 결과 재현 및 부정행위 검증 목적으로만 활용합니다. 참가팀의 별도 동의 없이 상업적 목적으로 사용하지 않으며, 평가 및 검증이 완료된 후 삭제합니다.

#### 4. 외부 오픈소스 및 지식재산권
공개된 오픈소스, 라이브러리 및 외부 구현은 관련 라이선스와 이용 조건을 준수하는 범위에서 사용할 수 있으며, 사용한 외부 구현의 출처를 명시해야 합니다. 참가팀은 제출물에 사용된 자료 및 기술에 대해 적법한 사용 권한을 확보해야 하며, 제출물로 인해 제3자의 지식재산권 침해가 발생하는 경우 이에 따른 모든 법적 책임은 해당 참가팀에 있으며, 주최 측은 이에 대한 책임을 지지 않습니다.

#### 5. 참가팀 간 코드 및 구현 공유 금지
참가팀은 대회 기간 중 자신의 소스 코드, 구현 내용 및 제출물을 다른 참가팀에 공개하거나 공유할 수 없으며, 다른 참가팀의 구현을 무단 복제하는 행위도 금지됩니다. 단, 공개된 논문, 오픈소스 및 기술 자료의 활용은 허용됩니다.

#### 6. 참가 자격 및 팀 구성
참가팀은 최대 3명으로 구성할 수 있으며, 참가자는 하나의 팀에만 소속될 수 있습니다. 동일인이 복수의 팀에 중복 참가하는 것은 허용되지 않습니다.

#### 7. 대회 관계자 참가 제한
대회 운영 및 평가에 직접 관여하는 주최 측 관계자는 참가할 수 없습니다.

#### 8. 평가 시스템 우회 및 부정행위 금지
평가 프로그램 또는 성능 측정 결과를 무단으로 변경하거나, 실제 추론을 대체하는 등 평가 결과를 부당하게 왜곡하는 행위를 금지합니다. <br>
특히 다음 행위는 허용되지 않습니다.<br>
- 평가 입력에 대한 출력값을 사전에 준비하거나 하드코딩하여 실제 추론을 대체하는 행위
- 평가 시간, GPU 메모리 사용량 등 성능 측정값을 조작하는 행위
- 평가 프로그램 또는 측정 로직을 무단으로 변경하거나 우회하는 행위
- 허용되지 않은 외부 GPU, API 등 연산 자원을 사용하는 행위

#### 9. 부정행위 및 실격
상기 규칙을 위반하거나, 명시된 규칙 외에도 평가 결과 조작, 부정한 이익 취득 등 대회의 공정성과 신뢰성을 훼손하는 행위가 확인된 경우 실격 처리될 수 있습니다.

#### 10. 리더보드 및 최종 평가
리더보드에 표시된 점수와 순위는 잠정 결과이며, 최종 평가 결과는 제출물의 재현성 및 규정 준수 여부를 검증한 후 확정됩니다.

#### 11. 대회 규칙 변경 및 공지
주최 측은 공정한 대회 운영을 위해 필요한 경우 규칙 및 평가 절차를 변경할 수 있으며, 변경 사항과 적용 시점은 대회 공식 홈페이지를 통해 공지합니다.
<br>
<hr />

### Evaluation Criteria
The organizing committee's evaluation client sends requests to participating teams' servers according to predefined scenarios to measure throughput, latency, and accuracy. Rankings are determined by throughput during the maximum-load (saturation) stage.
  * &lt;Details&gt;
    * **Formula : Score = saturation-stage req/s** <br>
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;= Number of successful requests / (Stage start time ~ Completion time of the last response)
    * **Ranking eligibility requirements (all must be satisfied)** <br>
      ① Zero errors (HTTP errors, disconnections, and 600-second timeouts are all counted as errors)<br>
      ② Overall accuracy ≥ Baseline accuracy − 0.02
    * Baseline: vLLM v0.32.0 default settings (provided example: team_baseline)
    * Reference metrics (not included in rankings): TTFT·TPOT p50/p99, output tok/s, per-task accuracy
    * All times are measured using client-side wall-clock time, and output token counts are calculated using the client-side tokenizer (server usage values are not used).
<br>

### Evaluation Environment
Participating teams submit their serving servers as Docker-images, which are executed by the organizing committee's evaluation server under the following isolated conditions. Teams may freely choose their serving framework (vLLM, SGLang, llama.cpp, custom implementations, etc.).
  * &lt;Hardware / Isolation&gt;
    * One GPU: NVIDIA H100 80GB (TBD)
    * **GPU memory usage limit: XX GiB (including model weights, TBD)**
    * 8 dedicated CPU cores (cpuset), 128 GiB host memory, 16 GiB shm
    * Internet access disabled, only evaluation-client connections permitted, restricted container privileges (--cap-drop ALL)
    * Model: Qwen3-8B (BF16), mounted read-only at /model
    * 15-minute startup limit: both a successful /v1/models response and successful one-token generation are required for readiness

  * &lt;Interface Requirements&gt;
    * Start the server using the image ENTRYPOINT (no additional arguments), listening on 0.0.0.0:8000
    * POST /v1/completions : stream=true (SSE) required
    * GET /v1/models, model name submission
    * Environment variables MODEL_PATH, SERVED_MODEL_NAME, PORT are provided
    * Requests contain raw prompts with the chat template already applied, with temperature=0 and max_tokens specified.
<br>
   
### Benchmark
The evaluation client (bench.py) and example scenarios are provided through a GitHub repository. Problems are regenerated using a new seed for each evaluation, so answers cannot be memorized or responses cached.<br>
※ **The official evaluation uses private seeds and scenarios; scores measured using public scenarios may differ from leaderboard scores.**
  * &lt;Tasks&gt;
    * kv_retrieval
        * Retrieve values from a key-value list of approximately 3.1k tokens, output ≤ 24 tokens
        * Four questions share one document, allowing prefix reuse
    * arith_chain
      * Approximately 110-token, eight-step arithmetic problem, output ≤ 768 tokens
      * Evaluated using the final ANSWER: &lt;n&gt; format
    * longbench_v2
      * LongBench-v2 long-context four-choice questions, context transmitted without truncation (several thousand to 1M tokens), output ≤ 128 tokens
      * Evaluated using The correct answer is (X)
  * &lt;Load Schedule&gt;
    * open-loop: requests are sent at predetermined times regardless of responses
    * low / mid / high: Poisson 2 / 8 / 24 req/s, with 60 / 240 / 720 requests, respectively
    * saturation: 600 requests submitted concurrently after all preceding requests have completed
<br>

### Restrictions
**Violations of the following restrictions will result in disqualification.**
  * &lt;Permitted&gt;
    * Selection and modification of serving frameworks (schedulers, memory managers, attention backends), custom CUDA/Triton kernels
    * All KV cache management techniques: paging, prefix caching, CPU offloading, KV quantization, eviction, chunked prefill, admission control, etc. (accuracy requirements must be satisfied)
    * Use of open-source libraries (LMCache, FlashInfer, etc.). Sources and licenses must be acknowledged
  * &lt;Prohibited&gt;
    * Replacing the designated model, additional training or distillation, introducing separate draft/proxy models
    * **Changing model weight precision (BF16 must be maintained)**
    * Precomputation based on evaluation problems: response caching or including precomputed KV caches in the image (prefix caching during request processing is permitted)
    * Ignoring or modifying request parameters (prompt, max_tokens, temperature), falsifying streaming timing (dummy first tokens, etc.)
    * Circumventing GPU memory limits: UVM/managed memory, concealing memory usage in separate processes, etc.
    * Using multiple GPUs, external nodes, or networks; manipulating the evaluation client or harness
<br>

### Competition Rules
**This competition aims to ensure fair evaluation and reproducible results. All participants must comply with the following rules.**
1. Submissions must be reproducible in the evaluation environment, and source code will be used to investigate misconduct and reproduce results.
2. Participating teams must submit any materials requested for evaluation.
3. Source code developed by participating teams must remain confidential throughout the competition.
4. Submissions must not use copyrighted or proprietary data, patents, trademarks, code, or confidential source content without authorization. Data or content that violates service agreements or another company's trade secrets is not permitted.
5. The organizer assumes no responsibility for submissions that infringe third-party intellectual property rights.
6. The organizer will use submissions solely to verify sharing or copying between participating teams and will not use them commercially or retain them after the competition. Participating teams are solely responsible for damages arising from third-party intellectual property infringement caused by their submissions.
7. Using external implementations without attribution or copying another team's implementation will result in disqualification.
8. Each team may have no more than three members and must demonstrate that no member participates in another team.
9. Competition organizers and related personnel are not eligible to participate.
10. Any conduct that undermines trust or constitutes unfair behavior may result in disqualification, even if it does not explicitly violate the rules above.
<br>
