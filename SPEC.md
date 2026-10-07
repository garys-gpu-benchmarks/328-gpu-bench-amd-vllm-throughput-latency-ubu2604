# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

Starts python -m vllm.entrypoints.openai.api_server with mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, waits for /v1/models, runs scripts/benchmark_serving.py once, then stops the server. input_len, output_len, and concurrency come from yaml. Measures serving throughput, TTFT, TPOT, ITL, and end-to-end latency. output_format: csv Sweep dimensions: model_name, dtype, tensor_parallel_size, gpu_memory_utilization, prompt_source, input_len, output_len, max_num_seqs.

## Parameters

| Parameter | CLI Flag | Tested Values | Default | Description |
| --- | --- | --- | --- | --- |
| model_name | `--model-name` | smoke=mistralai/Mistral-7B-v0.3, baseline=mistralai/Mistral-7B-v0.3, extended=mistralai/Mistral-7B-v0.3 | mistralai/Mistral-7B-v0.3 | From Parameter list; see Execution Description With Parameters. |
| dtype | `--dtype` | smoke=bfloat16, baseline=bfloat16, extended=bfloat16 | bfloat16 | From Parameter list; see Execution Description With Parameters. |
| tensor_parallel_size | `--tensor-parallel-size` | smoke=1, baseline=1, extended=1 | 1 | From Parameter list; see Execution Description With Parameters. |
| gpu_memory_utilization | `--gpu-memory-utilization` | smoke=0.9, baseline=0.9, extended=0.9 | 0.9 | From Parameter list; see Execution Description With Parameters. |
| prompt_source | `--prompt-source` | smoke=synthetic, baseline=synthetic, extended=synthetic | synthetic | From Parameter list; see Execution Description With Parameters. |
| input_len | `--input-len` | smoke=64, baseline=512, extended=1024 | 512 | From Parameter list; see Execution Description With Parameters. |
| output_len | `--output-len` | smoke=16, baseline=13320, extended=16500 | 13320 | From Parameter list; see Execution Description With Parameters. |
| max_num_seqs | `--max-num-seqs` | smoke=2, baseline=8, extended=16 | 8 | From Parameter list; see Execution Description With Parameters. |
| max_concurrency | `--max-concurrency` | smoke=2, baseline=8, extended=8 | 8 | From Parameter list; see Execution Description With Parameters. |

## Invocation

```bash
Start python -m vllm.entrypoints.openai.api_server on port 8000, then run scripts/benchmark_serving.py
```

## Raw Output Format

raw_results.csv with the serving summary written twice. Alias columns repeat the same p50 values under both _ms and _msec names

sample_index,status,output_token_throughput_tokens_s,tokens_per_sec,time_to_first_token_p50_ms,ttft_p50_ms,time_per_output_token_p50_ms,tpot_p50_ms,inter_token_latency_itl_p50_ms,itl_p50_ms,end_to_end_request_latency_ms,e2e_ms,output_token_throughput_tokens_sec,ttft_p50_msec,time_per_output_token_tpot_p50_msec,inter_token_latency_itl_p50_msec,end_to_end_request_latency_msec,error_message
0,ok,80,80,20,20,15,15,14,14,200,200,80,20,15,14,200,

## Metrics

- **#1: Output token throughput** — stored as `output_token_throughput_tokens_sec`.
- **#2: Time To First Token, ms** — stored as `ttft_p50_msec`.
- **#3: Time Per Output Token, ms** — stored as `time_per_output_token_tpot_p50_msec`.
- **#4: Inter-token latency, ms** — stored as `inter_token_latency_itl_p50_msec`.
- **#5: End-to-end request latency, ms** — stored as `end_to_end_request_latency_msec`.

## Framework

Starts python -m vllm.entrypoints.openai.api_server with mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, waits for /v1/models, runs scripts/benchmark_serving.py once, then stops the server. input_len, output_len, and concurrency come from yaml. Measures serving throughput, TTFT, TPOT, ITL, and end-to-end latency.

## Installation and Execution Summary

Start python -m vllm.entrypoints.openai.api_server --model mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, wait for /v1/models, run scripts/benchmark_serving.py once with yaml input_len, output_len, and concurrency, then stop the server, to measure serving throughput, TTFT, TPOT, ITL, and end-to-end latency

## Platform Portability

- **AMD (primary):** ```bash
Start python -m vllm.entrypoints.openai.api_server on port 8000, then run scripts/benchmark_serving.py
```
- **NVIDIA:** Primary target is AMD ROCm. NVIDIA notes in this section are reference only and are not the execution path.

## Model Context Protocols

- **Active:** None

## Execution-Loop Validation Contract

EXECUTION CHAIN: `run_benchmark.sh` ➔ raw output ➔ `scripts/parse_results.py` ➔ `results/benchmark.db` ➔ `scripts/validate_results.py`

This benchmark uses a lightweight, SQLite-integrated execution loop for result validation. All validation is performed by `scripts/validate_results.py`.

### Validation script usage

```bash
export BENCHMARK_PYTHON=/usr/bin/python3.13  # optional; select the installed interpreter

# After a live run:
".venv/bin/python" scripts/validate_results.py --db results/benchmark.db

# CI / no-GPU path (seeds fixture and validates it):
".venv/bin/python" scripts/validate_results.py --seed-fixture --quiet

# Override DB path via environment variable:
BENCHMARK_DB=tests/fixtures/benchmark.db \
  ".venv/bin/python" scripts/validate_results.py
```

### Run artifact contract

raw_results.csv with the serving summary written twice. Alias columns repeat the same p50 values under both _ms and _msec names

sample_index,status,output_token_throughput_tokens_s,tokens_per_sec,time_to_first_token_p50_ms,ttft_p50_ms,time_per_output_token_p50_ms,tpot_p50_ms,inter_token_latency_itl_p50_ms,itl_p50_ms,end_to_end_request_latency_ms,e2e_ms,output_token_throughput_tokens_sec,ttft_p50_msec,time_per_output_token_tpot_p50_msec,inter_token_latency_itl_p50_msec,end_to_end_request_latency_msec,error_message
0,ok,80,80,20,20,15,15,14,14,200,200,80,20,15,14,200,

```bash
bash run_benchmark.sh --help
bash run_benchmark.sh --profile smoke --validate
bash run_benchmark.sh --profile baseline --validate
bash run_benchmark.sh --profile extended --validate
```
`run_benchmark.sh --help` prints usage and exits. The harness calls `scripts/ensure_setup.sh` when `.setup_state` is absent.

### Required integrity checks (built into `validate_results.py`)

1. Latest run exists and `runs.status = 'ok'`.
2. `run.error_message` is NULL.
3. `started_at` and `finished_at` are valid ISO-8601 UTC strings.
4. All required aggregate metrics in `runs` are non-NULL and finite.
5. All required aggregate metrics are physically sensible (positive values). Starts python -m vllm.entrypoints.openai.api_server with mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, waits for /v1/models, runs scripts/benchmark_serving.py once, then stops the server. input_len, output_len, and concurrency come from yaml. Measures serving throughput, TTFT, TPOT, ITL, and end-to-end latency.
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Starts python -m vllm.entrypoints.openai.api_server with mistralai/Mistral-7B-v0.3 on 127.0.0.1:8000 for every profile, waits for /v1/models, runs scripts/benchmark_serving.py once, then stops the server. input_len, output_len, and concurrency come from yaml. Measures serving throughput, TTFT, TPOT, ITL, and end-to-end latency.

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
