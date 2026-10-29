---
name: llm-judge-semantic-equivalence-scoring
description: Use when evaluating free-form open-domain or biomedical QA predictions
  (e.g., CRAG, BioASQ, PubMedQA) where exact-match scoring would undercount semantically
  correct answers.
when_to_use_negative:
- The task has a single canonical short answer where rule-based exact/normalized string
  matching is sufficient (e.g., factoid lookup already scored by CRAG's local_evaluation.py
  rule-based matching)
- No reference answers are available for the questions being scored
- The judge model must run locally and a 72B-parameter judge (Qwen2.5-72B-Instruct)
  is infeasible on available hardware
tools:
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge that determines semantic equivalence between model predictions
    and reference answers
- name: TTARAG
  role: Source of the generated answers being judged; its parallel_evaluation.py pipeline
    saves predictions and summaries for scoring
  repo: https://github.com/sunxin000/TTARAG
- name: CRAG
  role: Benchmark providing the QA pairs and reference answers; its local_evaluation.py
    implements rule-based matching plus LLM assessment with correct/missing/incorrect
    scoring
  repo: https://github.com/facebookresearch/CRAG
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:535120148545371857356a031e06d3c57635b861915ac89898dcfaf5d7c6fb42
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Brave Search API
  - CRAG
  - ChatGLM-3-6b
  - Llama-2-7b-chat
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - NVIDIA A100 GPU
  - Pyserini
  - Qwen2.5-72B-Instruct
  - TTARAG
  - ms-marco-MiniLM-L-6-v2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# llm-judge-semantic-equivalence-scoring

## Summary

Scores RAG model predictions against reference answers using Qwen2.5-72B-Instruct as an LLM-as-a-judge that judges semantic equivalence rather than exact string match, enabling evaluation of free-form QA outputs from CRAG, BioASQ, and PubMedQA experiments.

## When to use

Use when evaluating free-form open-domain or biomedical QA predictions (e.g., CRAG, BioASQ, PubMedQA) where exact-match scoring would undercount semantically correct answers.

## When NOT to use

- The task has a single canonical short answer where rule-based exact/normalized string matching is sufficient (e.g., factoid lookup already scored by CRAG's local_evaluation.py rule-based matching)
- No reference answers are available for the questions being scored
- The judge model must run locally and a 72B-parameter judge (Qwen2.5-72B-Instruct) is infeasible on available hardware

## Inputs

- Model prediction strings (answers from naive-RAG, CoT, ICL, TTARAG runs)
- Reference/gold answer strings from CRAG (2,706 QA pairs), BioASQ (500 QA pairs), or PubMedQA (1,000 QA pairs)
- Qwen2.5-72B-Instruct weights or serving endpoint for the judge model

## Outputs

- Per-question judge verdicts (semantically equivalent vs not)
- Aggregated accuracy (%) per model-condition and per domain (e.g., crag_accuracy_table.csv, medical_domain_accuracy.csv)

## How to apply

Collect the generated answers (e.g., from naive-RAG, CoT, ICL, TTARAG runs via parallel_evaluation.py) alongside the gold reference answers for each benchmark. Present each prediction-reference pair to Qwen2.5-72B-Instruct, which decides whether the two are semantically equivalent, and record the binary judge verdict. Aggregate verdicts into accuracy (%) per domain and per model-condition (e.g., the CRAG Table 1 layout or per-dataset CSVs like crag_accuracy_table.csv). Validate the judge itself by comparing its labels to human annotations on a random sample of 100 examples — the paper reports 98% agreement, so a fresh judge run should reproduce agreement in that range before trusting large-scale scores.

## Related tools

- **Qwen2.5-72B-Instruct** (LLM-as-a-judge that determines semantic equivalence between model predictions and reference answers)
- **TTARAG** (Source of the generated answers being judged; its parallel_evaluation.py pipeline saves predictions and summaries for scoring) — https://github.com/sunxin000/TTARAG
- **CRAG** (Benchmark providing the QA pairs and reference answers; its local_evaluation.py implements rule-based matching plus LLM assessment with correct/missing/incorrect scoring) — https://github.com/facebookresearch/CRAG

## Evaluation signals

- Judge labels agree with human annotators on a random sample of 100 examples at roughly the reported 98% rate before large-scale scoring is trusted
- Verdicts are binary semantic-equivalence decisions, not exact-match string comparisons — paraphrased correct answers score as correct
- Aggregated accuracy (%) values per model-condition are reproducible and match the paper's Table 1/Table 3 layout (e.g., Llama-3.1-8b-it TTARAG 75.0 vs naive-RAG 55.6 on BioASQ)
- Deltas computed vs naive-RAG (e.g., +19.4% BioASQ, +10.8% PubMedQA for Llama-3.1-8b-it) fall in the reported positive range for all three backbones

## Limitations

- Judge quality is anchored on a single 100-example human-agreement sample (98%); residual disagreement up to ~2% may bias absolute accuracy values
- Evaluation differs from CRAG's official scoring scheme (correct=1 / missing=0 / incorrect=-1 with rule-based matching), so results are not directly comparable to the original CRAG leaderboard numbers

## Evidence

- [methods] LLM-as-a-judge protocol: [evidence span withheld — anchor status: not_found]
- [methods] Judge validation: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark scoring scheme: [evidence span withheld — anchor status: not_found]
