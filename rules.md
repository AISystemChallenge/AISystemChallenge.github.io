---
title:
feature_text: |
  <!--# Samsung AI System Challenge 2026-->
feature_image: "/assets/logos/banner_AISC2026.png"
excerpt:
---

### 평가 기준
사무국 평가 클라이언트가 참가팀 서버에 정해진 시나리오의 요청을 보내 처리량·지연시간·정확도를 측정하며, 최대 부하 구간(saturation)의 처리량으로 순위를 정합니다.
  * &lt;상세&gt;
    * 평가 산식 : **Score = saturation 스테이지 req/s**<br>
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;**= 성공 요청 수 / (스테이지 시작 ~ 마지막 응답 완료 시간)**
    * 순위 인정 조건 (모두 충족)<br>
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
    * GPU 메모리 사용 상한: XX GiB (모델 weight 포함, TBD)
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
평가 클라이언트(bench.py)와 예제 시나리오는 GitHub repository로 제공됩니다. 문제는 평가 시점마다 seed로 새로 생성하므로 정답을 외우거나 응답을 캐싱할 수 없습니다.
※ 본 평가는 비공개 seed·시나리오를 사용하므로, 공개 시나리오로 측정한 점수는 리더보드 점수와 다를 수 있습니다.<br>
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
아래의 제한 사항을 위반하면 실격처리 됩니다.
  * &lt;허용&gt;
    * 서빙 프레임워크의 선택·수정 (스케줄러, 메모리 관리자, attention 백엔드), 커스텀 CUDA/Triton 커널
    * KV cache 관리 기법 일체: paging, prefix caching, CPU offloading, KV 양자화, eviction, chunked prefill, admission control 등 (단, 정확도 조건 충족 필요)
    * 오픈소스 라이브러리 사용 (LMCache, FlashInfer 등). 출처와 라이선스 명기 필요
  * &lt;금지&gt;
    * 지정 모델 교체, 추가 학습·증류, 별도 draft/proxy 모델 도입
    * <b>모델 weight 정밀도 변경 (BF16 고정)</b>
    * 평가 문제에 의존한 사전 계산: 응답 캐싱, 사전 계산한 KV를 이미지에 포함 (요청 처리 중의 prefix caching은 허용)
    * 요청 파라미터(prompt, max_tokens, temperature) 무시·변경, 스트리밍 타이밍 위조 (더미 첫 토큰 등)
    * GPU 메모리 상한 우회: UVM/managed memory, 별도 프로세스에 메모리 은닉 등
    * 다중 GPU·외부 노드·네트워크 사용, 평가 클라이언트·하네스 조작
<br>

### 대회 규칙
<strong>이번 대회에서는 공정한 평가 및 재현 가능한 결과를 보장하는 것을 목표로 합니다. 이를 위해 다음 사항을 준수해야 합니다.</strong>
1. 제출물은 테스트 환경에서 재현할 수 있어야 하며, 소스 코드는 부정 행위 및 결과 재현을 위해 사용됩니다.
2. 참가팀은 평가를 위해 요청된 자료를 제출해야 합니다.
3. 대회 기간 동안 참가팀들이 작성하는 소스 코드는 비공개한 상태로 운영합니다.
4. 제출물은 저작권이 있거나 독점적인 데이터 또는 특허 또는 상표, 코드 또는 비공개 소스 콘텐츠를 사용해서는 안 됩니다. 서비스 계약이나 다른 기업의 영업 비밀을 위반하는 데이터나 콘텐츠의 사용은 허용되지 않습니다.
5. 제출물에 제3자의 지식재산권을 침해하는 행위 등이 포함될 경우, 당사는 이에 대한 책임을 지지 않습니다.
6. 당사는 참가팀들 간 공유나 카피 방지 검증용으로만 제출물을 사용할 예정이며 대회 종료 후에는 어떠한 경우에도 상업적 목적으로 해당 결과물을 사용하거나 보관하지 않으며, 만약 제출물로 인해 제3자 지식재산권 침해가 발생하는 경우 해당 침해로 인한 손해는 모두 참가팀에게 책임이 있습니다.
7. 출처 없이 외부 구현을 사용하거나, 다른 참가팀의 구현을 복사하여 사용하는 경우에는 실격 처리됩니다.
8. 한 참가팀은 3명 이하로 구성할 수 있으며, 참가팀에서는 팀원이 이 대회의 다른 팀에 중복 참가하지 않음을 스스로 입증해야 합니다.
9. 대회와 관련된 주최자 및 관련자는 참가할 수 없습니다.
10. 상기 규칙을 위배하지 않더라도 신뢰에 반하거나 부당 행위를 하는 경우 실격 처리될 수 있습니다. 
<br>
<hr />

### Evaluation Criteria
The organizing committee's evaluation client sends requests to participating teams' servers according to predefined scenarios to measure throughput, latency, and accuracy. Rankings are determined by throughput during the maximum-load (saturation) stage.
  * &lt;Details&gt;
    * Formula : **Score = saturation-stage req/s**<br>
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;**= Number of successful requests / (Stage start time ~ Completion time of the last response)**
    * Ranking eligibility requirements (all must be satisfied)<br>
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
    * GPU memory usage limit: XX GiB (including model weights, TBD)
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
※ The official evaluation uses private seeds and scenarios; scores measured using public scenarios may differ from leaderboard scores.
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
Violations of the following restrictions will result in disqualification.
  * &lt;Permitted&gt;
    * Selection and modification of serving frameworks (schedulers, memory managers, attention backends), custom CUDA/Triton kernels
    * All KV cache management techniques: paging, prefix caching, CPU offloading, KV quantization, eviction, chunked prefill, admission control, etc. (accuracy requirements must be satisfied)
    * Use of open-source libraries (LMCache, FlashInfer, etc.). Sources and licenses must be acknowledged
  * &lt;Prohibited&gt;
    * Replacing the designated model, additional training or distillation, introducing separate draft/proxy models
    * <b>Changing model weight precision (BF16 must be maintained)</b>
    * Precomputation based on evaluation problems: response caching or including precomputed KV caches in the image (prefix caching during request processing is permitted)
    * Ignoring or modifying request parameters (prompt, max_tokens, temperature), falsifying streaming timing (dummy first tokens, etc.)
    * Circumventing GPU memory limits: UVM/managed memory, concealing memory usage in separate processes, etc.
    * Using multiple GPUs, external nodes, or networks; manipulating the evaluation client or harness
<br>

### Competition Rules
<strong>This competition aims to ensure fair evaluation and reproducible results. All participants must comply with the following rules.</strong>
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
