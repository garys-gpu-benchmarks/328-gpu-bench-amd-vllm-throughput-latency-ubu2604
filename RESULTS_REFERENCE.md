# Results: 328 vLLM Inference Throughput & Latency

**Main goal:** Measure vLLM serving throughput and latency  
**Tools:** Start python -m vllm.entrypoints.openai.api_server on port 8000, then run scripts/benchmark_serving.py

Reference results measured by the maintainer on the system shown in each section (generator TEMPLATE_00_139 · recorded 2026-11-13). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× AMD Instinct MI300X VF (196288 MB) · INTEL(R) XEON(R) PLATINUM 8568Y+ · Ubuntu 26.04.1 LTS · kernel 7.0.0-38-generic · driver 7.1.9.31600000
- **Run:** 2026-11-13T14:39:02Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 02:49 · benchmark 01:00 · whole run 04:59
- **Source:** run 328_20261113_143902
- **Command:** `bash run_benchmark.sh --smoke --model-name mistralai/Mistral-7B-v0.3 --dtype bfloat16 --tensor-parallel-size 1 --gpu-memory-utilization 0.9 --input-len 64 --output-len 16 --max-num-seqs 2 --max-concurrency 2`

| # | Metric | Result |
|---|---|---|
| 1 | Output token throughput (output_token_throughput_tokens_sec) | 373.5920360372786 |
| 2 | Time To First Token, ms (ttft_p50_msec) | 13.204709499859746 |
| 3 | Time Per Output Token, ms (time_per_output_token_tpot_p50_msec) | 4.488315466672551 |
| 4 | Inter-token latency, ms (inter_token_latency_itl_p50_msec) | 4.696945000205233 |
| 5 | End-to-end request latency p50, ms (end_to_end_request_latency_msec) | 80.529441499948 |
<!-- /profile:smoke -->
