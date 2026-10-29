---
name: question-answering-accuracy-aggregation
description: Use when you have model predictions plus gold reference answers for a
  QA benchmark (e.g., 2,706 CRAG question-answer pairs) and need correctness scores
  that tolerate paraphrase instead of exact string match.
when_to_use_negative:
- Exact string match or rule-based scoring alone is sufficient and an LLM judge would
  add unnecessary cost
- The task is not reference-based question answering (e.g., you need standard classification
  metrics on labeled feature tables)
- Gold reference answers are unavailable, so semantic equivalence against a reference
  cannot be judged
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge that determines semantic equivalence between each model prediction
    and the reference answer to score correctness
  repo: https://github.com/sunxin000/TTARAG
- name: facebookresearch/CRAG
  role: Source of the evaluation benchmark, its evaluation protocol, and local_evaluation.py
    reference implementation
  repo: https://github.com/facebookresearch/CRAG
- name: NVIDIA A100 GPU
  role: Hardware used to run evaluation over the 2,706 queries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:42f3d6e69667301256fdfe10264e90ace5f18bbf511f70ceb0283f09a467d33e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - NVIDIA A100 GPU
  - Qwen2.5-72B-Instruct
  - facebookresearch/CRAG
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# question-answering-accuracy-aggregation

## Summary

Scores model-generated QA answers against gold references using an LLM-as-a-judge (Qwen2.5-72B-Instruct) that judges semantic equivalence, then aggregates per-question verdicts into per-domain and overall accuracy across the five CRAG domains (Finance, Sports, Music, Movie, Open).

## When to use

Use when you have model predictions plus gold reference answers for a QA benchmark (e.g., 2,706 CRAG question-answer pairs) and need correctness scores that tolerate paraphrase instead of exact string match.

## When NOT to use

- Exact string match or rule-based scoring alone is sufficient and an LLM judge would add unnecessary cost
- The task is not reference-based question answering (e.g., you need standard classification metrics on labeled feature tables)
- Gold reference answers are unavailable, so semantic equivalence against a reference cannot be judged

## Inputs

- Model-generated predictions for the CRAG benchmark (per-question answers)
- 2,706 CRAG question-answer pairs with gold reference answers
- Judge prompt template following the CRAG evaluation protocol
- Qwen2.5-72B-Instruct judge model (e.g., served on one NVIDIA A100 GPU)

## Outputs

- judge_verdicts.csv with one record per CRAG QA pair containing question id, prediction, reference, and verdict
- Per-domain accuracy scores across the five CRAG domains (Finance, Sports, Music, Movie, Open)
- Overall aggregated accuracy score

## How to apply

Load the generated predictions alongside the 2,706 CRAG question-answer pairs with their gold references. For each question, build a judge prompt following the CRAG evaluation protocol that presents the prediction and the reference and asks Qwen2.5-72B-Instruct to decide semantic equivalence rather than exact string match; parse the verdict (equivalent / not equivalent) into a per-question record with question id, prediction, reference, and verdict. Aggregate the verdicts into per-domain and overall accuracy scores across the five CRAG domains and save the records to judge_verdicts.csv with the aggregated scores. Note the CRAG protocol also allows rule-based matching alongside LLM assessment, assigning three scores: correct (1 point), missing (0 points), and incorrect (-1 point). Validate that judge_verdicts.csv has one record per QA pair and that LLM-judge verdicts agree with human annotators at roughly the reported 98% on a random 100-example sample.

## Related tools

- **Qwen2.5-72B-Instruct** (LLM-as-a-judge that determines semantic equivalence between each model prediction and the reference answer to score correctness) — https://github.com/sunxin000/TTARAG
- **facebookresearch/CRAG** (Source of the evaluation benchmark, its evaluation protocol, and local_evaluation.py reference implementation) — https://github.com/facebookresearch/CRAG
- **NVIDIA A100 GPU** (Hardware used to run evaluation over the 2,706 queries)

## Examples

```
python local_evaluation.py
```

## Evaluation signals

- judge_verdicts.csv exists and contains one verdict record for each of the 2,706 CRAG QA pairs with required fields (question id, prediction, reference, verdict)
- LLM-judge verdicts show ~98% agreement with human annotators on a random sample of 100 examples
- Aggregated scores cover all five CRAG domains (Finance, Sports, Music, Movie, Open) plus an overall accuracy
- Verdicts parse cleanly into the equivalent / not equivalent categories with no unparsed judge outputs

## Limitations

- The LLM judge is not perfect: reported agreement with human annotators is 98% on a 100-example sample, so a small fraction of verdicts may disagree with human judgment
- The official CRAG scoring is richer than binary correctness, assigning correct (1 point), missing (0 points), and incorrect (-1 point) via rule-based matching plus LLM assessment; a pure equivalence judge collapses missing vs incorrect
- Running a 72B judge over the full 2,706-query set requires substantial GPU resources (evaluated on one NVIDIA A100 GPU)

## Evidence

- [methods] LLM-as-a-judge verdicts determine correctness: [evidence span withheld — anchor status: not_found]
- [methods] Judge-human agreement: [evidence span withheld — anchor status: not_found]
- [readme] CRAG auto-evaluation scoring scheme: [evidence span withheld — anchor status: not_found]
