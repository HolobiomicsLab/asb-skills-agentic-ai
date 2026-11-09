---
name: exact-match-qa-scoring
description: Use when comparing constrained-generation or LLM-only QA answers to gold
  answers on benchmarks like 2WikiMH, Mintaka, WebQSP, or Bank and you need per-model
  accuracy and precision to fill a results table.
when_to_use_negative:
- The task is open-ended generation with no gold answer available — Exact Match and
  precision cannot be computed
- You only need answer speed/throughput, which throughput.py measures separately,
  not answer quality
- The dataset answers require reasoning the string metric cannot capture and no judge
  model is available — scores will be systematically deflated
tools:
- name: Qwen2.5-72B
  role: Candidate answering model whose outputs are scored; reported with judge P=85.2%
    and A=78.9
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-7B
  role: Smaller model scored in the same protocol, achieving 73.0% accuracy
  repo: https://github.com/rpo19/ReFactX
- name: Llama3.3-70B
  role: Model scored in the same protocol, achieving P=77.8% and C=70.6%
  repo: https://github.com/rpo19/ReFactX
- name: PostgreSQL
  role: Serves the disk-backed Wikidata prefix-tree index whose answers feed the scoring
    runs
  repo: https://github.com/rpo19/ReFactX
- name: Nvidia Tesla T4
  role: GPU used for the scoring runs with key-value caching enabled
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:7f6be722d6a833b323ba65abbedef553a5531440281f742d91527d5105cc64f0
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Exact Match QA Scoring

## Summary

Scores generated QA answers against gold answers using string-based Exact Match alongside LLM-as-a-Judge accuracy/precision, as done in the ReFactX evaluation across 2WikiMH, Mintaka, WebQSP, and Bank.

## When to use

Use when comparing constrained-generation or LLM-only QA answers to gold answers on benchmarks like 2WikiMH, Mintaka, WebQSP, or Bank and you need per-model accuracy and precision to fill a results table.

## When NOT to use

- The task is open-ended generation with no gold answer available — Exact Match and precision cannot be computed
- You only need answer speed/throughput, which throughput.py measures separately, not answer quality
- The dataset answers require reasoning the string metric cannot capture and no judge model is available — scores will be systematically deflated

## Inputs

- QA benchmark datasets with gold answers: 2WikiMH, Mintaka, WebQSP, Bank
- Generated answer strings from constrained-generation (ReFactX) and unconstrained (LLM-only) runs
- Model/dataset configuration (e.g., Qwen2.5-72B on 2WikiMH)

## Outputs

- Per model/dataset Exact Match, accuracy (A), and precision (P) scores
- Results table reproducing the paper's reported figures (e.g., judge P=85.2%/A=78.9 for Qwen2.5-72B; Bank precision up to 51.3% and accuracy up to 42.8%)

## How to apply

Run the QA model (e.g., Qwen2.5-7B/72B, Llama3.3-70B, Phi-4) with and without constrained generation over the Wikidata prefix-tree index, collecting answers per model/dataset pair via eval.py. Score each answer with Exact Match against the gold answer, and additionally with an LLM-as-a-Judge to compute accuracy (A) and precision (P). Compute both metrics per model/dataset pair, e.g., Qwen2.5-72B judge P=85.2% and A=78.9, and ReFactX accuracy of 73.0% with Qwen2.5-7B and 92.5% with the larger Qwen2.5 model. Use precision (share of correct answers among all given answers) as the key differentiator, since ReFactX always outperforms LLM-only models in precision under LLM-as-a-Judge. Treat Exact Match as the strict baseline and the judge scores as the validation target: accept the assembled results table only if entries match the paper's reported figures.

## Related tools

- **Qwen2.5-72B** (Candidate answering model whose outputs are scored; reported with judge P=85.2% and A=78.9) — https://github.com/rpo19/ReFactX
- **Qwen2.5-7B** (Smaller model scored in the same protocol, achieving 73.0% accuracy) — https://github.com/rpo19/ReFactX
- **Llama3.3-70B** (Model scored in the same protocol, achieving P=77.8% and C=70.6%) — https://github.com/rpo19/ReFactX
- **PostgreSQL** (Serves the disk-backed Wikidata prefix-tree index whose answers feed the scoring runs) — https://github.com/rpo19/ReFactX
- **Nvidia Tesla T4** (GPU used for the scoring runs with key-value caching enabled) — https://github.com/rpo19/ReFactX

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Judge scores for Qwen2.5-72B match P=85.2% and A=78.9
- Llama3.3-70B scores match P=77.8% and C=70.6
- ReFactX accuracy is 73.0% with Qwen2.5-7B and 92.5% with the larger Qwen2.5 model
- ReFactX precision is higher than LLM-only precision for every model under LLM-as-a-Judge
- On 2WikiMH, ReFactX accuracy and precision exceed LLM-only by more than 20 percentage points (e.g., 92.5%/96.4% vs 45.0%/45.2% for Qwen2.5-72B)

## Limitations

- ReFactX performs worse than LLM-only in accuracy on Mintaka and WebQSP, where LLM-only parametric knowledge reaches 82.0% and 80.5% accuracy
- The Bank dataset is especially challenging: precision tops out at 51.3% and accuracy at 42.8%
- Exact Match is a strict string metric; the paper complements it with LLM-as-a-Judge to capture paraphrased correct answers

## Evidence
