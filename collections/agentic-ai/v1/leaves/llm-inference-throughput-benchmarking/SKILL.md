---
name: llm-inference-throughput-benchmarking
description: Use when you need to quantify the generation-time overhead introduced
  by constrained or index-guided decoding versus normal unconstrained LLM decoding
  on identical hardware and token budgets.
when_to_use_negative:
- When the goal is QA accuracy/precision evaluation rather than timing — use eval.py
  with a DATASET argument instead, since throughput.py only measures generation latency
- When the index is the small in-memory prefix tree from the try_refactx notebook
  (31k facts) — that setup skips the .env/PostgreSQL configuration needed to reproduce
  the disk-backed overhead measurement
- When you need per-question or per-dataset breakdowns; this skill yields a single
  aggregate wall-clock figure for a fixed token budget (4000 tokens), not question-level
  metrics
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0130
tools:
- name: throughput.py
  role: CLI script that runs generation for --max-tokens tokens in either unconstrained
    or prefix-tree-constrained mode and writes timings to the output JSON file
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-3B
  role: LLM whose decoding throughput is benchmarked in both constrained and unconstrained
    modes with key-value caching enabled
- name: PostgreSQL
  role: Disk-backed store for the Wikidata prefix-tree index that imposes constrained
    decoding; running on the same machine as the model
- name: Nvidia Tesla T4
  role: Single GPU used for both baseline and constrained runs so hardware is held
    constant
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:ff4f0aca3f18671fac9bccea331aac119b8a41a772d5c99a850cec57a64347ba
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - Nvidia Tesla T4
  - PostgreSQL
  - Qwen2.5-3B
  - throughput.py
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0130
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# llm-inference-throughput-benchmarking

## Summary

Measures the wall-clock latency added by constrained decoding against a disk-backed prefix-tree index compared to unconstrained LLM generation, to quantify retrieval/KB-guidance overhead. Used here to show ReFactX's PostgreSQL-backed prefix tree adds only ~1.3% latency over 4000-token generation.

## When to use

Use when you need to quantify the generation-time overhead introduced by constrained or index-guided decoding versus normal unconstrained LLM decoding on identical hardware and token budgets.

## When NOT to use

- When the goal is QA accuracy/precision evaluation rather than timing — use eval.py with a DATASET argument instead, since throughput.py only measures generation latency
- When the index is the small in-memory prefix tree from the try_refactx notebook (31k facts) — that setup skips the .env/PostgreSQL configuration needed to reproduce the disk-backed overhead measurement
- When you need per-question or per-dataset breakdowns; this skill yields a single aggregate wall-clock figure for a fixed token budget (4000 tokens), not question-level metrics

## Inputs

- ReFactX environment with pip-installed refactx[torch]
- .env configuration file (cp env-sample.txt .env) pointing to the PostgreSQL-backed prefix-tree index
- Qwen2.5-3B model checkpoint with key-value caching enabled
- throughput.py script with --model, --index, --max-tokens 4001 flags
- One Nvidia Tesla T4 GPU

## Outputs

- out.json containing total wall-clock time (seconds) for unconstrained generation of 4000 tokens (baseline ~300.14 s)
- out.json containing total wall-clock time (seconds) for constrained generation of 4000 tokens (~303.89 s)
- Computed relative overhead percentage: (constrained_time - unconstrained_time)/unconstrained_time, reported as ~1.3%

## How to apply

Set up the ReFactX environment (virtualenv/conda, pip install refactx[torch]) and prepare the .env configuration from env-sample.txt pointing at the PostgreSQL-backed prefix-tree index. First run the unconstrained-decoding baseline with throughput.py (python throughput.py --model MODEL --index INDEX --max-tokens 4001 --output out.json --unconstrained-generation) using Qwen2.5-3B with key-value caching enabled on one Nvidia Tesla T4, recording the total wall-clock time for 4000 tokens (reported baseline: 300.14 s). Then re-run throughput.py without the --unconstrained-generation flag so generation is constrained by the PostgreSQL-backed prefix tree to only token sequences forming existing facts, recording the total time (reported: 303.89 s). Compute the relative overhead as constrained_time/unconstrained_time - 1 ((303.89-300.14)/300.14 = 1.3%); write both timings plus the computed overhead to out.json. The key decision point is holding model, index, hardware (T4 GPU), and KV caching constant across both runs so the only variable is the constrained-vs-unconstrained decoding mode.

## Related tools

- **throughput.py** (CLI script that runs generation for --max-tokens tokens in either unconstrained or prefix-tree-constrained mode and writes timings to the output JSON file) — https://github.com/rpo19/ReFactX
- **Qwen2.5-3B** (LLM whose decoding throughput is benchmarked in both constrained and unconstrained modes with key-value caching enabled)
- **PostgreSQL** (Disk-backed store for the Wikidata prefix-tree index that imposes constrained decoding; running on the same machine as the model)
- **Nvidia Tesla T4** (Single GPU used for both baseline and constrained runs so hardware is held constant)

## Examples

```
python throughput.py --model Qwen2.5-3B --index wikidata_prefix_tree --max-tokens 4001 --output out.json --unconstrained-generation  # baseline run; re-run without the flag for the constrained measurement
```

## Evaluation signals

- Unconstrained baseline of 4000 tokens should take approximately 300.14 s under Qwen2.5-3B + KV caching + one Nvidia Tesla T4
- Constrained run of 4000 tokens should take approximately 303.89 s under the same configuration
- Computed relative overhead (constrained/unconstrained - 1) should be close to the reported ~1.3%
- Both runs must use the same model, index, GPU, and key-value caching settings; any change to those variables invalidates the comparison
- out.json should contain both raw timings and the derived overhead percentage, matching the throughput.py --output convention

## Limitations

- The ~1.3% overhead was measured for one model (Qwen2.5-3B), one index (PostgreSQL-backed Wikidata prefix tree), and one GPU (Tesla T4); different hardware or models may shift the number
- The prefix tree holds up to 800M facts from Wikidata; smaller or in-memory indexes (e.g., the 31k-facts notebook setup) will not reproduce the disk-backed latency profile
- The overhead is reported for generating 4000 tokens with key-value caching enabled; other token budgets or caching configurations are not characterized in the article

## Evidence
