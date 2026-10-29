---
name: constrained-decoding-latency-measurement
description: Use when you need to quantify the inference-latency cost of token-level
  constrained generation (e.g. a prefix-tree index restricting the vocabulary) relative
  to an unconstrained decoding baseline, on a fixed token budget.
when_to_use_negative:
- You are measuring QA accuracy/precision (use eval.py on Mintaka, WebQSP, 2WikiMH
  or Bank datasets instead) rather than decoding latency
- You need the scale/latency properties of the index itself over 800M facts rather
  than end-to-end generation time for a fixed token count
- Hardware, KV-cache setting, or token budget differs from the benchmark conditions
  and you want to compare against the reported 300.14 s / 303.89 s numbers without
  re-baselining
tools:
- name: Qwen2.5-3B
  role: Decoder LLM timed under both constrained and unconstrained conditions
  repo: https://github.com/rpo19/ReFactX
- name: PostgreSQL
  role: Disk-backed store for the prefix-tree index that constrains generation in
    the timed condition
  repo: https://github.com/rpo19/ReFactX
- name: Nvidia Tesla T4
  role: Single-GPU execution environment for both timed runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:2023bf3251c2b0e11916b5e167bf91d19a0700b367b638da7facce75a0864443
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - Nvidia Tesla T4
  - PostgreSQL
  - Qwen2.5-3B
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# Constrained-decoding latency measurement

## Summary

Benchmarking skill that quantifies the generation-time overhead a KB-guided constrained-decoding layer (e.g. ReFactX's PostgreSQL-backed prefix tree) adds over normal unconstrained LLM decoding, by timing both conditions on the same hardware and model.

## When to use

Use when you need to quantify the inference-latency cost of token-level constrained generation (e.g. a prefix-tree index restricting the vocabulary) relative to an unconstrained decoding baseline, on a fixed token budget.

## When NOT to use

- You are measuring QA accuracy/precision (use eval.py on Mintaka, WebQSP, 2WikiMH or Bank datasets instead) rather than decoding latency
- You need the scale/latency properties of the index itself over 800M facts rather than end-to-end generation time for a fixed token count
- Hardware, KV-cache setting, or token budget differs from the benchmark conditions and you want to compare against the reported 300.14 s / 303.89 s numbers without re-baselining

## Inputs

- Language model to benchmark (e.g. Qwen2.5-3B) with key-value caching enabled
- PostgreSQL-backed prefix-tree index over tokenized verbalized KB facts (configured via .env from env-sample.txt)
- Token budget for generation (4000 tokens, passed as --max-tokens 4001)
- Single GPU hardware environment (one Nvidia Tesla T4)

## Outputs

- Wall-clock time in seconds for unconstrained decoding of the token budget (baseline)
- Wall-clock time in seconds for prefix-tree-constrained decoding of the same token budget
- Computed relative overhead (constrained/unconstrained - 1, expected ~1.3%)
- JSON output file (out.json) recording both timings and the overhead

## How to apply

Install the ReFactX environment (virtualenv/conda, `pip install refactx[torch]`) and prepare the `.env` file from env-sample.txt pointing to the PostgreSQL-backed prefix-tree index. Run throughput.py twice with Qwen2.5-3B, key-value caching enabled, on one Nvidia Tesla T4, and a fixed budget of 4000 tokens (the script uses `--max-tokens 4001`): first with `--unconstrained-generation` to record the baseline wall-clock time (reported 300.14 s), then without the flag so generation is constrained by the PostgreSQL-backed prefix tree to token sequences forming existing facts (reported 303.89 s). Compute the relative overhead as constrained_time / unconstrained_time - 1, i.e. (303.89 - 300.14) / 300.14 = 1.3%. Write both timings plus the computed overhead to the output JSON; accept the result when the measured overhead is close to the reported ~1.3%, confirming the disk-backed prefix tree adds only ~1% latency.

## Related tools

- **Qwen2.5-3B** (Decoder LLM timed under both constrained and unconstrained conditions) — https://github.com/rpo19/ReFactX
- **PostgreSQL** (Disk-backed store for the prefix-tree index that constrains generation in the timed condition) — https://github.com/rpo19/ReFactX
- **Nvidia Tesla T4** (Single-GPU execution environment for both timed runs)

## Examples

```
python throughput.py --model MODEL --index INDEX --max-tokens 4001 --output out.json --unconstrained-generation
```

## Evaluation signals

- Measured overhead (constrained minus unconstrained, relative to unconstrained) falls near the reported 1.3% (e.g. 303.89 s vs 300.14 s on 4000 tokens)
- Both runs use identical settings: same model (Qwen2.5-3B), key-value caching enabled, same GPU (one Nvidia Tesla T4), same --max-tokens 4001 budget
- Output JSON contains both total wall-clock timings plus the derived overhead ratio
- Unconstrained baseline and constrained run each complete the full 4000-token generation without truncation
- The observed overhead is consistent with the paper's claim that the disk-backed prefix tree adds only ~1% latency

## Limitations

- Reported numbers are specific to Qwen2.5-3B with PostgreSQL on the same machine, key-value caching enabled, and one Nvidia Tesla T4; other models/GPUs may shift the overhead
- The measurement covers a fixed 4000-token budget only; overhead may vary with shorter or longer generations
- Validation tolerance is qualitative ('close to ~1.3%'), with no published confidence interval or variance across repeated runs

## Evidence

- [results] Constrained generation adds a 1.3% time overhead: generating 4000 tokens takes 303.89s vs 300.14s unconstrained: "the total time for generating 4000 tokens increases by only 1.3%, from 300.14 seconds to 303.89."
- [readme] Run the throughput experiment with throughput.py using --unconstrained-generation for the baseline: "python throughput.py --model MODEL --index INDEX --max-tokens 4001 --output out.json [--unconstrained-generation]"
- [intro] Disk-backed prefix tree scale and latency claim: "a disk-backed prefix tree holds up to 800M facts and adds only ∼1 % latency."
- [results] Benchmark hardware and caching conditions: "Results on 4000 tokens, using Qwen2.5-3B with PostgreSQL running on the same machine, with key-value caching enabled, using one Nvidia Tesla T4."
