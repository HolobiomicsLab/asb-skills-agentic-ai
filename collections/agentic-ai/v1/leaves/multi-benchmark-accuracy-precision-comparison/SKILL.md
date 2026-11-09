---
name: multi-benchmark-accuracy-precision-comparison
description: Use when comparing an LLM with externally-grounded generation (e.g.,
  constrained generation over a KB prefix tree) against LLM-only baselines across
  multiple QA benchmarks (Bank, Mintaka, 2WikiMH, WebQSP), reporting accuracy and
  precision.
when_to_use_negative:
- The goal is only to reproduce the single-model throughput/latency experiment — use
  throughput.py rather than the multi-benchmark eval table
- The question concerns a small KB that fits an in-memory prefix tree (e.g., the 31k-facts
  try_refactx notebook); the full benchmark comparison targets the large disk-backed
  Wikidata index
- Only one benchmark is of interest and no LLM-only baseline comparison is needed
  — the skill's value is the constrained-vs-parametric cross-benchmark comparison
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3345
- http://edamontology.org/topic_0219
tools:
- name: eval.py
  role: ReFactX reproduction script; run with --index, --model, --dataset to produce
    constrained-generation answers per benchmark
  repo: https://github.com/rpo19/ReFactX
- name: PostgreSQL
  role: Serves the disk-backed Wikidata prefix-tree index with key-value caching during
    constrained generation
- name: Nvidia Tesla T4
  role: GPU on which the indexed-fact generation runs in the reported results
- name: Qwen2.5-72B
  role: Best-performing model in the comparison table (judge P=85.2%, A=78.9)
- name: Llama3.3-70B
  role: Comparison model (judge P=77.8%, C=70.6%)
- name: Phi-4
  role: Comparison model noted for consistent precision across all answer types
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:b114b11ad313acf14bb5356d9ab58e74c27b231bea2582cf146e3f986903bbea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - Llama3.3-70B
  - Nvidia Tesla T4
  - Phi-4
  - PostgreSQL
  - Qwen2.5-3B
  - Qwen2.5-72B
  - Qwen2.5-7B
  - eval.py
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3345
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# multi-benchmark-accuracy-precision-comparison

## Summary

Reproduce ReFactX's four-benchmark QA evaluation by running constrained-generation (Wikidata prefix-tree index) and LLM-only (parametric) variants of each model, then scoring answers with Exact Match and LLM-as-a-Judge to compare accuracy and precision per model/dataset pair.

## When to use

Use when comparing an LLM with externally-grounded generation (e.g., constrained generation over a KB prefix tree) against LLM-only baselines across multiple QA benchmarks (Bank, Mintaka, 2WikiMH, WebQSP), reporting accuracy and precision.

## When NOT to use

- The goal is only to reproduce the single-model throughput/latency experiment — use throughput.py rather than the multi-benchmark eval table
- The question concerns a small KB that fits an in-memory prefix tree (e.g., the 31k-facts try_refactx notebook); the full benchmark comparison targets the large disk-backed Wikidata index
- Only one benchmark is of interest and no LLM-only baseline comparison is needed — the skill's value is the constrained-vs-parametric cross-benchmark comparison

## Inputs

- Wikidata prefix-tree index of verbalized facts (built via PrefixTree.md instructions)
- QA benchmark datasets: 2WikiMH, Mintaka, WebQSP, Bank
- LLM checkpoints: Llama3.3-70B, Phi-4, Qwen2.5-72B, Qwen2.5-7B
- PostgreSQL-backed disk index configuration (.env prepared from env-sample.txt)

## Outputs

- Accuracy and precision table per model/dataset pair (constrained ReFactX vs LLM-only), under Exact Match and LLM-as-a-Judge scoring
- Per-dataset comparison findings (e.g., >20% gains on 2WikiMH; degraded accuracy on Mintaka and WebQSP)

## How to apply

Install ReFactX in a virtualenv or conda environment via `pip install refactx[torch]`, then load the disk-backed Wikidata prefix-tree index of verbalized facts (up to 800M facts) served through PostgreSQL with key-value caching on an Nvidia Tesla T4. For each model (Llama3.3-70B, Phi-4, Qwen2.5-72B, Qwen2.5-7B) and each dataset (2WikiMH, Mintaka, WebQSP, Bank), run `eval.py` with constrained generation: the LLM is instructed via In-Context Learning to invoke the Fact command, after which decoding follows only valid prefix-tree paths. Repeat the same runs with `--unconstrained-generation` to obtain the LLM-only parametric-knowledge baselines. Score all answers with both Exact Match and LLM-as-a-Judge, computing accuracy and precision per model/dataset pair, then assemble the results table and check it against the paper's reference values (e.g., Qwen2.5-72B judge P=85.2% / A=78.9; accuracy 73.0% with Qwen2.5-7B and 92.5% with the larger Qwen2.5 model; Bank precision up to 51.3% and accuracy up to 42.8%).

## Related tools

- **eval.py** (ReFactX reproduction script; run with --index, --model, --dataset to produce constrained-generation answers per benchmark) — https://github.com/rpo19/ReFactX
- **PostgreSQL** (Serves the disk-backed Wikidata prefix-tree index with key-value caching during constrained generation)
- **Nvidia Tesla T4** (GPU on which the indexed-fact generation runs in the reported results)
- **Qwen2.5-72B** (Best-performing model in the comparison table (judge P=85.2%, A=78.9))
- **Llama3.3-70B** (Comparison model (judge P=77.8%, C=70.6%))
- **Phi-4** (Comparison model noted for consistent precision across all answer types)

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Qwen2.5-72B LLM-as-a-Judge precision ≈ 85.2% and accuracy ≈ 78.9% on the reported benchmark table
- ReFactX accuracy ≈ 73.0% with Qwen2.5-7B and ≈ 92.5% with the larger Qwen2.5 model
- On 2WikiMH, ReFactX improves both accuracy and precision by more than 20% relative to LLM-only models (e.g., 92.5% vs 45.0% accuracy for Qwen2.5-72B)
- Bank dataset precision up to 51.3% and accuracy up to 42.8%
- On Mintaka and WebQSP, ReFactX accuracy should be lower than LLM-only baselines, which reach 82.0% and 80.5% respectively

## Limitations

- ReFactX performs worse than LLM-only on Mintaka and WebQSP, where the models' parametric knowledge already covers the datasets largely
- The Bank dataset proved especially challenging, with accuracy up to only 42.8%
- Throughput results are reported for a specific setup (4000 tokens, Qwen2.5-3B with PostgreSQL on the same machine, key-value caching enabled, one Nvidia Tesla T4); other setups may not match
- Constrained generation adds a ~1.3% time overhead (303.89s vs 300.14s for 4000 tokens)

## Evidence

- [readme] Unconstrained baseline flag: [evidence span withheld — anchor status: not_found]
