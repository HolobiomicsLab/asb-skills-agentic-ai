---
name: prefix-tree-index-latency-overhead-analysis
description: Use when you need to quantify the runtime cost added by constrained decoding
  (or another index-backed inference constraint) relative to a normal unconstrained
  LLM decoding baseline, over a fixed token budget.
when_to_use_negative:
- You want to evaluate answer quality (precision/accuracy on QA benchmarks) rather
  than inference throughput — use the eval.py experiment workflow instead
- Your generation workload has a different token budget or hardware setup, since the
  ~1.3% overhead is calibrated for 4000 tokens on one Nvidia Tesla T4 with key-value
  caching enabled
- The prefix tree is the small in-memory 31k-facts index from try_refactx.ipynb, which
  does not exercise the disk-backed PostgreSQL path measured by this experiment
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: Qwen2.5-3B
  role: LLM whose decoding throughput (constrained vs unconstrained) is benchmarked
    over 4000 tokens
  repo: https://github.com/rpo19/ReFactX
- name: PostgreSQL
  role: Disk-backed storage for the prefix-tree index consulted during constrained
    generation; its lookup latency is what the experiment quantifies
  repo: https://github.com/rpo19/ReFactX
- name: Nvidia Tesla T4
  role: Single-GPU hardware on which both timing runs are executed with key-value
    caching enabled
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:9f810a01d808180f88bbd062649087fa9c2960a5603624aeef93b90df838ab8c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - Nvidia Tesla T4
  - PostgreSQL
  - Qwen2.5-3B
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# prefix-tree-index-latency-overhead-analysis

## Summary

Measures the generation-time latency overhead introduced by KB-guided constrained decoding over a disk-backed prefix tree, by comparing token throughput of constrained vs unconstrained LLM generation (ReFactX reports ~1.3% overhead on 4000 tokens with a PostgreSQL-backed index).

## When to use

Use when you need to quantify the runtime cost added by constrained decoding (or another index-backed inference constraint) relative to a normal unconstrained LLM decoding baseline, over a fixed token budget.

## When NOT to use

- You want to evaluate answer quality (precision/accuracy on QA benchmarks) rather than inference throughput — use the eval.py experiment workflow instead
- Your generation workload has a different token budget or hardware setup, since the ~1.3% overhead is calibrated for 4000 tokens on one Nvidia Tesla T4 with key-value caching enabled
- The prefix tree is the small in-memory 31k-facts index from try_refactx.ipynb, which does not exercise the disk-backed PostgreSQL path measured by this experiment

## Inputs

- Model to benchmark (e.g., Qwen2.5-3B) accessible from the runtime machine
- PostgreSQL-backed prefix-tree index over verbalized KB facts (e.g., Wikidata-derived 800M facts)
- throughput.py script from the ReFactX repository with --model, --index, --max-tokens, --output, and optional --unconstrained-generation flags

## Outputs

- Unconstrained-decoding total wall-clock time for the fixed token budget (e.g., 300.14 s for 4000 tokens)
- Constrained-decoding total wall-clock time for the same budget (e.g., 303.89 s)
- Computed relative overhead percentage ((303.89-300.14)/300.14 ≈ 1.3%) written to the output JSON file (out.)

## How to apply

Set up the environment (virtualenv/conda, pip install refactx[torch]) and configure the .env file from env-sample.txt to point to the PostgreSQL-backed prefix-tree index. Run the throughput.py baseline with --unconstrained-generation using Qwen2.5-3B with key-value caching enabled on one Nvidia Tesla T4, requesting --max-tokens 4001 so that 4000 tokens are generated, and record the total wall-clock time (reported: 300.14 s). Re-run throughput.py without the flag so decoding is constrained to token sequences forming existing facts in the prefix tree, recording the total time (reported: 303.89 s). Compute the relative overhead as (constrained_time - unconstrained_time) / unconstrained_time, which yields (303.89-300.14)/300.14 ≈ 1.3%, and write both timings plus the computed overhead to the output JSON. The rationale is that a disk-backed prefix tree should hold up to 800M facts while adding only ~1% latency, so the paired same-hardware, same-token-budget comparison isolates the constraint-checking cost.

## Related tools

- **Qwen2.5-3B** (LLM whose decoding throughput (constrained vs unconstrained) is benchmarked over 4000 tokens) — https://github.com/rpo19/ReFactX
- **PostgreSQL** (Disk-backed storage for the prefix-tree index consulted during constrained generation; its lookup latency is what the experiment quantifies) — https://github.com/rpo19/ReFactX
- **Nvidia Tesla T4** (Single-GPU hardware on which both timing runs are executed with key-value caching enabled)

## Examples

```
python throughput.py --model MODEL --index INDEX --max-tokens 4001 --output out. --unconstrained-generation
```

## Evaluation signals

- Unconstrained baseline time for 4000 tokens is close to the reported 300.14 s on the same hardware (Qwen2.5-3B, one Nvidia Tesla T4, key-value caching enabled)
- Constrained time is close to the reported 303.89 s, i.e., only slightly above the baseline
- Computed relative overhead (constrained/unconstrained - 1) is approximately 1.3%, consistent with the claim that the disk-backed prefix tree adds only ~1% latency
- Both timing runs generate the same number of tokens (--max-tokens 4001), so the comparison is token-budget-matched
- Output file (out.) contains both raw timings and the derived overhead figure

## Limitations

- The ~1.3% overhead figure is specific to Qwen2.5-3B, PostgreSQL on the same machine, key-value caching enabled, and a single Nvidia Tesla T4; other models or hardware may differ
- The overhead was measured over a 4000-token generation; different token budgets or fact-access patterns may change the relative cost
- Constrained generation only narrows the vocabulary to tokens forming an existing fact; the benchmark measures latency, not whether constrained outputs are more accurate

## Evidence

- [readme] throughput.py CLI for the throughput experiment: [evidence span withheld — anchor status: not_found]
