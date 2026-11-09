---
name: per-domain-accuracy-aggregation
description: Use when you have per-question model predictions and reference answers
  for a multi-domain benchmark (e.g., CRAG's Finance, Sports, Music, Movie, Open domains)
  and need to compute and compare accuracy (%) per domain and overall across model
  conditions.
when_to_use_negative:
- You only have raw model outputs without reference answers or judge verdicts — accuracy
  cannot be computed and scoring must be run first
- Your benchmark is not partitioned into domains (e.g., PubMedQA or BioASQ single-domain
  sets), where per-domain breakdown adds no information beyond a single overall metric
- You need CRAG's official perfect/acceptable/missing/incorrect graded scoring (correct
  1 / missing 0 / incorrect -1) rather than the paper's binary semantic-equivalence
  accuracy
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge scoring semantic equivalence between model predictions and
    reference answers before aggregation
  repo: https://github.com/sunxin000/TTARAG
- name: parallel_evaluation.py
  role: Generates and saves predictions for each model-condition pair (with or without
    --do_tta) that feed the aggregation
  repo: https://github.com/sunxin000/TTARAG
- name: Brave Search API
  role: Retrieves up to 50 full HTML pages per CRAG question so all conditions share
    identical retrieval content
- name: MiniLM-v6
  role: Reranks sentences in the HTML pages and selects the top 20 sentences as the
    fixed retrieval content per question
  repo: https://github.com/sunxin000/TTARAG
- name: TTARAG
  role: Test-time adaptation condition (via parallel_evaluation.py --do_tta) whose
    accuracy is compared against baselines in the aggregated table
  repo: https://github.com/sunxin000/TTARAG
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:cc602adaea7c488a87cc6bcf3a19252d09b8161e03b043d2fefb839cec307ad1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - Brave Search API
  - Llama-3.1-8B-Instruct
  - MiniLM-v6
  - NVIDIA A100 GPU
  - Qwen2.5-72B-Instruct
  - TTARAG
  - ms-marco-MiniLM-L-6-v2
  - parallel_evaluation.py
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# per-domain-accuracy-aggregation

## Summary

Aggregate LLM-as-a-judge scored QA predictions into a per-domain and overall accuracy table for every model-condition pair (e.g., TTARAG vs naive-RAG, CoT, ICL, Self-RAG, Ret-Robust, RAAT) on the CRAG benchmark, reproducing the paper's Table 1 / Table 3 layout.

## When to use

Use when you have per-question model predictions and reference answers for a multi-domain benchmark (e.g., CRAG's Finance, Sports, Music, Movie, Open domains) and need to compute and compare accuracy (%) per domain and overall across model conditions.

## When NOT to use

- You only have raw model outputs without reference answers or judge verdicts — accuracy cannot be computed and scoring must be run first
- Your benchmark is not partitioned into domains (e.g., PubMedQA or BioASQ single-domain sets), where per-domain breakdown adds no information beyond a single overall metric
- You need CRAG's official perfect/acceptable/missing/incorrect graded scoring (correct 1 / missing 0 / incorrect -1) rather than the paper's binary semantic-equivalence accuracy

## Inputs

- CRAG benchmark predictions per model-condition pair (2,706 QA pairs across Finance, Sports, Music, Movie, Open domains)
- CRAG reference answers
- LLM-as-a-judge verdicts from Qwen2.5-72B-Instruct (semantic equivalence per prediction)

## Outputs

- crag_accuracy_table.csv with accuracy (%) per domain and overall for every model-condition pair, matching the Table 1 / Table 3 layout

## How to apply

Run all model-condition pairs (naive-RAG, CoT, ICL prompting baselines, TTARAG via --do_tta, and pretrained baselines Ret-Robust, RAAT, Self-RAG) on the same 2,706 CRAG queries, saving predictions per model. Score every prediction with the LLM-as-a-judge Qwen2.5-72B-Instruct, judging semantic equivalence against the reference answers; the judge shows 98% agreement with human annotators on a 100-example sample, so rely on it rather than exact string matching. Group scored predictions by CRAG domain and compute accuracy (%) per domain plus an overall average for each model-condition pair, then assemble the rows/columns into a table matching the paper's Table 1 / Table 3 layout and save it as crag_accuracy_table.csv. Use identical retrieval content for all conditions (up to 50 HTML pages via Brave Search API, reranked by MiniLM-v6, top 20 sentences selected) so accuracy differences are attributable to the answering strategy.

## Related tools

- **Qwen2.5-72B-Instruct** (LLM-as-a-judge scoring semantic equivalence between model predictions and reference answers before aggregation) — https://github.com/sunxin000/TTARAG
- **parallel_evaluation.py** (Generates and saves predictions for each model-condition pair (with or without --do_tta) that feed the aggregation) — https://github.com/sunxin000/TTARAG
- **Brave Search API** (Retrieves up to 50 full HTML pages per CRAG question so all conditions share identical retrieval content)
- **MiniLM-v6** (Reranks sentences in the HTML pages and selects the top 20 sentences as the fixed retrieval content per question) — https://github.com/sunxin000/TTARAG
- **TTARAG** (Test-time adaptation condition (via parallel_evaluation.py --do_tta) whose accuracy is compared against baselines in the aggregated table) — https://github.com/sunxin000/TTARAG

## Examples

```
python parallel_evaluation.py --do_generate --dataset crag --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- The aggregated table covers every model-condition pair across all CRAG domains plus an overall column, matching the Table 1 / Table 3 layout
- Judge-based scoring reproduces the reported 98% agreement with human annotators on a 100-example sample
- TTARAG rows show substantial improvements over naive-RAG and win in 19 out of 24 experimental settings versus standard RAG, CoT, and ICL baselines
- All conditions were evaluated on the same 2,706 queries with identical top-20 reranked retrieval content, so per-domain counts are consistent across rows
- Output CSV parses cleanly with domain columns and accuracy values in a plausible 0-100 (%) range

## Limitations

- TTARAG accuracy depends on adaptation hyperparameters; learning rates between 1e-6 to 1e-5 and 3-5 adaptation pairs are optimal, though performance is not highly sensitive to these choices
- Semantic-equivalence judging is automated and may disagree with human graders on edge cases despite 98% agreement
- CRAG's official metric uses graded scores (correct 1 / missing 0 / incorrect -1) with rule-based matching plus LLM assessment; the paper's binary accuracy is not identical to that scheme

## Evidence

- [methods] LLM-judge agreement: [evidence span withheld — anchor status: not_found]
- [methods] Judge role: [evidence span withheld — anchor status: not_found]
- [intro] Headline result expectation: [evidence span withheld — anchor status: not_found]
