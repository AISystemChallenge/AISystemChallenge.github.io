---
title:
feature_text: |
  # Samsung AI System Challenge 2026
feature_image: "/assets/logos/banner_last.png"
excerpt:
---

### 평가 기준
사무국 평가 클라이언트가 참가팀 서버에 정해진 시나리오의 요청을 보내 처리량·지연시간·정확도를 측정하며, 최대 부하 구간(saturation)의 처리량으로 순위를 정합니다.<br>
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
참가팀은 서빙 서버를 Docker 이미지로 제출하고, 사무국 평가 서버가 아래 격리 조건에서 실행합니다. 서빙 프레임워크는 자유입니다 (vLLM, SGLang, llama.cpp, 직접 구현 등).<br>
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
평가 클라이언트(bench.py)와 예제 시나리오는 GitHub repository로 제공됩니다. 문제는 평가 시점마다 seed로 새로 생성하므로 정답을 외우거나 응답을 캐싱할 수 없습니다.<br>
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
  * &lt;부하 스케줄&gt; open-loop: 응답과 무관하게 미리 정한 시각에 전송
    * low / mid / high: Poisson 2 / 8 / 24 req/s, 각 60 / 240 / 720건
    * saturation: 앞 요청이 모두 끝난 뒤 600건 동시 투입
<br>

### 제한 사항
아래의 제한 사항을 위반하면 실격처리 됩니다.<br>
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

#### Evaluation Criteria
The goal of this competition is to minimize the execution time of a specified benchmark on the given Gem5 simulator, and the score is evaluated by averaging the performance improvement (speedup) compared to the baseline execution time using the geometric mean.
* Evaluation formula: <br>
&emsp;&emsp;**GAP_GEOMEAN(ref_time/candidate_time)  + TSVC_GEOMEAN(ref_time/candidate_time)**
  * Baseline Branch Predictor: TournamentBP
  * Baseline Compiler: Compilation options in the Makefile of the provided Docker image
    * GAP Compilation Optimization Baseline: clang -O3 -fno-vectorize  -fno-slp-vectorize
    * TSVC Compilation Optimization Baseline: clang -O3
<br>


#### Evaluation Environment
The same environment is provided through Docker images, and the submitted source code must run and terminate normally in that environment. <br>
**※ Docker images : [ghcr.io/cechallenge/ce_challenge_2025](https://ghcr.io/cechallenge/ce_challenge_2025)** <br>
* Gem5 version 25.0.0 (The latest stable version)
  * 1GHz single core
  * fetch_width=8
  * width=4
  * rob_size=128
  * int_regs=128, fp_regs=128
  * lq_entries=32, sq_entries=32
  * L1 Cache=8-way 32KB
  * L2 Cache=8-way 512KB
  * RVV enabled, VLEN=256 bits, ELEN=64 bits
* LLVM version 19.1.7 (stable version)
<br>

   
#### Benchmark
The benchmark is provided through a Docker image, and the benchmark source code cannot be modified arbitrarily. <br>
**※ Benchmarks are included in the CE challenge [Github repository](https://github.com/cechallenge/2025).** <br>
**※ Provided benchmarks and those actually used during the evaluation are not identical.** <br>
  * GAP (\<repo directory\>/gapbs)
    * GAP consists of 8 benchmarks, and the type and size of the graph data performed for each benchmark, as well as the number of iterations, are selected differently to reflect the characteristics and execution time of each benchmark.
    * bfs, cc, cc_sv, sssp, tc, pr, pr_spmv, bc
  * TSVC (\<repo directory\>/tsvc)
    * A total of 20 benchmarks representing each category have been selected.
    * s000, s121, s131, s171, s1213, s2244, s251, s1281, s311, s351, s491, s442, s212, s222, s231, s281, s341, s352, s4114, s321
<br>


#### Restrictions
###### Gem5
  * You can modify or add branch predictors under the \<repo directory\>/gem5/cpu/pred directory.
    * The branch predictor to be submitted to the MyCeChallengeBP class in \<repo directory\>/ce_challenge_2025/components/processors_ce_challenge.py must be connected and function normally in the evaluation environment.
    * The total size of the branch predictor is limited to 128KB.
    * The branchPred parameter of the existing RiscvO3CPU model will be changed to the submitted branch predictor for evaluation, and only the interface provided in the BranchPredictor class will be used.   
    * You can modify the BranchPredictor provided in gem5 or create a branch predictor in C++.
      * If you add a new branch predictor in C++, you must add the path to the file to \<repo directory\>/gem5/cpu/pred/SConscript and then rebuild gem5.
    * You must submit an explanation of the modifications or additions, and describe the hardware size consumed by the branch predictor to remember the branch history in bits.
  * All other modules cannot be modified.
<br>

###### LLVM
  * The LLVM build script is provided as llvm_build.sh in the \<repo directory\>/llvm directory. This build script cannot be modified.
  * You can implement the pass by modifying and adding files in the following subdirectories.
    * \<repo directory\>/llvm/llvm/lib
      * Directory: Analysis, Transforms, Target/RISCV, Passes
    * \<repo directory\>/llvm/llvm/include/llvm
      * Directory: Analysis, Transforms, Passes
  * You must submit a description of any modifications or additions you make.
  * All other modules cannot be modified.
<br>


###### Benchmarks
  * You can compile using the make command in each benchmark directory and run ./RUN_SELF_TEST in \<repo directory\>/ce_challenge_2025 to test it.
    * <ins>The provided SELF_TEST script runs each benchmark once for evaluation purposes. In actual evaluations, the number of repetitions and parameters may vary, and evaluation settings are not disclosed. Therefore, the scores generated by SELF_TEST may differ from those on the Leaderboard.</ins>
    * <ins>If an error occurs in the evaluation environment, we will notify you via email of the benchmark where the error occurred and the error message.</ins>
  * Except for the following restrictions, you can add compilation options for the implemented path to the “OPT_FLAGS” in each benchmark's Makefile.
    * GAP benchmark: Do not insert vectorization-related options (e.g., --vectorize-slp).
    * Do not insert hints related to RISC-V vector length (e.g., -force-vector-width, -mrvv-vector-bits).
  * No other parts of the Makefile except for “OPT_FLAGS” can be modified.
<br>You cannot arbitrarily modify the environment and scripts for other tests.<br>
<br>
**Violation of the above restrictions will result in disqualification.**
<br>


#### Competition Rules
* The goal of this competition is to ensure fair evaluation and reproducible results. To this end, the following rules must be observed.
1. Submissions must be reproducible in the test environment, and source code will be used to verify the absence of cheating and to reproduce results.
2. Participating teams must submit the requested materials for evaluation.
3. Source code written by participating teams during the competition period will be kept confidential.
4. Submissions must not use copyrighted or proprietary data, patents, trademarks, code, or confidential source content. The use of data or content that violates service agreements or other companies' trade secrets is not permitted.
5. We are not responsible for any infringement of third-party intellectual property rights contained in submissions.
6. We will use submissions solely for the purpose of verifying sharing and copying prevention among participating teams. After the competition ends, we will not use or store the results for commercial purposes under any circumstances. If a third party's intellectual property rights are infringed upon due to a submission, the participating team will be held responsible for all damages resulting from such infringement.
7. Using external implementations without attribution or copying another participating team's implementation will result in disqualification.
8. Each participating team may consist of up to three members, and participating teams must verify that their members are not participating in other teams in this competition.
9. Organizers and related parties involved in the competition are not eligible to participate.
10. Even if the above rules are not violated, disqualification may occur if there is a breach of trust or unfair conduct. 
