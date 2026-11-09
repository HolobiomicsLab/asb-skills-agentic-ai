---
name: qa-benchmark-experiment-reproduction
description: Use when the goal is to reproduce or extend a published LLM+knowledge-base
  QA benchmark comparison (e.g., ReFactX vs LLM-only on 2WikiMH, Mintaka, WebQSP,
  Bank) with EM and LLM-as-a-Judge scoring.
when_to_use_negative:
- The task is a quick trial rather than benchmark reproduction — use the try_refactx.ipynb
  notebook with the in-memory 31k-facts prefix tree instead of the full Wikidata index
- Only throughput/latency measurement is needed (use throughput.py alone rather than
  the full eval pipeline)
- Questions target knowledge already covered by model parametric knowledge where ReFactX
  is known to underperform, e.g., Mintaka or WebQSP where LLM-only reaches 82.0% and
  80.5% accuracy
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: PostgreSQL
  role: Serves the disk-backed Wikidata prefix-tree index with key-value caching enabled
    on the same machine
  repo: https://github.com/rpo19/ReFactX
- name: Nvidia Tesla T4
  role: Single GPU used for generation and key-value caching during evaluation
- name: Qwen2.5-72B
  role: 'Best reported model: judge precision 85.2% and accuracy 78.9% with ReFactX'
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-7B
  role: Smaller model variant achieving 73.0% accuracy on 2WikiMH
  repo: https://github.com/rpo19/ReFactX
- name: Llama3.3-70B
  role: Model variant with reported P=77.8% and C=70.6%
  repo: https://github.com/rpo19/ReFactX
- name: Phi-4
  role: Model variant that maintains consistent precision across all answer types
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:3589307847f0f5352bf4ddc21e05449d183be37cc38b23472bc2c00690a0c59e
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# qa-benchmark-experiment-reproduction

## Summary

Reproduces the ReFactX (ISWC 2025) QA benchmark table by running constrained-generation evaluation over a Wikidata prefix-tree index of ~800M verbalized facts against LLM-only baselines, scoring answers with Exact Match and LLM-as-a-Judge accuracy and precision across 2WikiMH, Mintaka, WebQSP, and Bank.

## When to use

Use when the goal is to reproduce or extend a published LLM+knowledge-base QA benchmark comparison (e.g., ReFactX vs LLM-only on 2WikiMH, Mintaka, WebQSP, Bank) with EM and LLM-as-a-Judge scoring.

## When NOT to use

- The task is a quick trial rather than benchmark reproduction — use the try_refactx.ipynb notebook with the in-memory 31k-facts prefix tree instead of the full Wikidata index
- Only throughput/latency measurement is needed (use throughput.py alone rather than the full eval pipeline)
- Questions target knowledge already covered by model parametric knowledge where ReFactX is known to underperform, e.g., Mintaka or WebQSP where LLM-only reaches 82.0% and 80.5% accuracy

## Inputs

- Disk-backed Wikidata prefix-tree index of ~800M verbalized facts (via PostgreSQL)
- QA benchmark datasets: 2WikiMH, Mintaka, WebQSP, Bank
- LLM checkpoints: Qwen2.5-3B, Qwen2.5-7B, Qwen2.5-72B, Llama3.3-70B, Phi-4
- Index/dataset/model python import files for INDEX, MODEL, DATASET arguments

## Outputs

- Results table of accuracy and precision (Exact Match and LLM-as-a-Judge) per model/dataset pair
- Constrained-generation (ReFactX) vs unconstrained (LLM-only) baseline comparison
- Throughput output JSON (e.g., out.) from throughput.py runs

## How to apply

Create a virtualenv or conda environment and pip install refactx[torch]; prepare the .env from env-sample.txt and build the disk-backed Wikidata prefix tree following PrefixTree.md, served via PostgreSQL with key-value caching enabled on a single Nvidia Tesla T4. For each model (Llama3.3-70B, Phi-4, Qwen2.5-7B, Qwen2.5-72B) and dataset (2WikiMH, Mintaka, WebQSP, Bank), run eval.py with constrained generation: the LLM is instructed via In-Context Learning to invoke the Fact command, after which generation follows only valid prefix-tree paths, then decoding reverts to normal. Repeat each run with the throughput.py --unconstrained-generation flag (or equivalent) to obtain the parametric-knowledge LLM-only baselines. Score answers with Exact Match and LLM-as-a-Judge, computing accuracy and precision per model/dataset pair, and assemble the results table for comparison against the paper's reported figures.

## Related tools

- **PostgreSQL** (Serves the disk-backed Wikidata prefix-tree index with key-value caching enabled on the same machine) — https://github.com/rpo19/ReFactX
- **Nvidia Tesla T4** (Single GPU used for generation and key-value caching during evaluation)
- **Qwen2.5-72B** (Best reported model: judge precision 85.2% and accuracy 78.9% with ReFactX) — https://github.com/rpo19/ReFactX
- **Qwen2.5-7B** (Smaller model variant achieving 73.0% accuracy on 2WikiMH) — https://github.com/rpo19/ReFactX
- **Llama3.3-70B** (Model variant with reported P=77.8% and C=70.6%) — https://github.com/rpo19/ReFactX
- **Phi-4** (Model variant that maintains consistent precision across all answer types) — https://github.com/rpo19/ReFactX

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Qwen2.5-72B judge precision ≈ 85.2% and accuracy ≈ 78.9%; Llama3.3-70B P=77.8% and C=70.6%
- 2WikiMH: ReFactX improves both accuracy and precision by more than 20% over LLM-only; Qwen2.5-72B reaches 92.5% judge accuracy and 96.4% precision vs 45.0% and 45.2% for LLM-only
- Qwen2.5-7B achieves 73.0% accuracy and Qwen2.5-72B-class model 92.5% on 2WikiMH
- Bank dataset: ReFactX precision up to 51.3% and accuracy up to 42.8%
- Constrained generation adds only ~1.3% time overhead (4000 tokens: 303.89s vs 300.14s unconstrained)

## Limitations

- ReFactX performs worse than LLM-only on Mintaka and WebQSP, where LLM parametric knowledge already covers the datasets (82.0% and 80.5% accuracy respectively)
- Bank dataset is especially challenging, capping at 51.3% precision and 42.8% accuracy
- Constrained generation only narrows the vocabulary to tokens forming existing KB facts; the final token choice always remains with the LLM's probability estimates
- Reproducing the full-scale Wikidata index requires a substantial PostgreSQL-backed disk setup (800M facts) and GPU resources

## Evidence
