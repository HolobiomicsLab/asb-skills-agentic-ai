---
name: crag-benchmark-evaluation-protocol
description: Use when reproducing or comparing RAG systems on CRAG question-answer
  pairs and you need correctness verdicts and per-domain/overall accuracy that match
  the paper's Table 1/Table 3 evaluation setup.
when_to_use_negative:
- The task requires CRAG's official graded scoring (perfect/acceptable/missing/incorrect
  with correct=1, missing=0, incorrect=-1) rather than the paper's binary semantic-equivalence
  accuracy
- The benchmark has short, deterministic gold answers where exact string or rule-based
  matching suffices, so the cost of a 72B LLM judge is unnecessary
- Predictions are not aligned to the CRAG question set (e.g. PubMedQA or BioASQ outputs),
  since the judge prompts and per-domain aggregation assume the five CRAG domains
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge that determines semantic equivalence between model predictions
    and CRAG reference answers
  repo: https://github.com/sunxin000/TTARAG
- name: TTARAG
  role: The evaluated system whose generated answers are scored by the judge pipeline
    (parallel_evaluation.py)
  repo: https://github.com/sunxin000/TTARAG
- name: NVIDIA A100 GPU
  role: Hardware used to run prediction generation over 2,706 queries and the judge
    model
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:c46d4e65fb6d183da172eaf15614294643397985c0030994ab9e04d9c4446292
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# crag-benchmark-evaluation-protocol

## Summary

Protocol for scoring RAG system answers on the CRAG benchmark (2,706 QA pairs across Finance, Sports, Music, Movie, Open domains) using an LLM-as-a-judge (Qwen2.5-72B-Instruct) that decides semantic equivalence between each model prediction and the reference answer instead of exact string matching.

## When to use

Use when reproducing or comparing RAG systems on CRAG question-answer pairs and you need correctness verdicts and per-domain/overall accuracy that match the paper's Table 1/Table 3 evaluation setup.

## When NOT to use

- The task requires CRAG's official graded scoring (perfect/acceptable/missing/incorrect with correct=1, missing=0, incorrect=-1) rather than the paper's binary semantic-equivalence accuracy
- The benchmark has short, deterministic gold answers where exact string or rule-based matching suffices, so the cost of a 72B LLM judge is unnecessary
- Predictions are not aligned to the CRAG question set (e.g. PubMedQA or BioASQ outputs), since the judge prompts and per-domain aggregation assume the five CRAG domains

## Inputs

- Generated model predictions for the 2,706 CRAG QA pairs
- CRAG benchmark question-answer pairs with gold reference answers (from facebookresearch/CRAG)
- Qwen2.5-72B-Instruct judge model weights/inference endpoint

## Outputs

- judge_verdicts.csv with one record per CRAG QA pair containing question id, prediction, reference, and verdict (equivalent / not equivalent)
- Aggregated accuracy table per domain (Finance, Sports, Music, Movie, Open) and overall, matching the Table 1 / Table 3 layout (e.g. crag_accuracy_table.csv)

## How to apply

Load the 2,706 CRAG question-answer pairs with their gold reference answers together with the generated model predictions. For each question, construct a judge prompt presenting the model prediction and the reference answer and ask Qwen2.5-72B-Instruct to decide semantic equivalence rather than exact string match; parse the verdict (equivalent / not equivalent) into a per-question record with question id, prediction, reference, and verdict. Aggregate per-domain and overall accuracy from the verdict records across the five CRAG domains (Finance, Sports, Music, Movie, Open) and save the structured records as judge_verdicts.csv alongside the aggregated scores (e.g. crag_accuracy_table.csv). The rationale for LLM judging is that CRAG answers are free-form; the paper validates the judge by measuring 98% agreement with human annotators on a random sample of 100 examples. Note the official CRAG auto-evaluation combines rule-based matching with LLM assessment, assigning correct (1 point), missing (0 points), and incorrect (-1 point) scores.

## Related tools

- **Qwen2.5-72B-Instruct** (LLM-as-a-judge that determines semantic equivalence between model predictions and CRAG reference answers) — https://github.com/sunxin000/TTARAG
- **TTARAG** (The evaluated system whose generated answers are scored by the judge pipeline (parallel_evaluation.py)) — https://github.com/sunxin000/TTARAG
- **NVIDIA A100 GPU** (Hardware used to run prediction generation over 2,706 queries and the judge model)

## Examples

```
python parallel_evaluation.py --do_generate --dataset crag --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- judge_verdicts.csv exists and contains a verdict record for each of the 2,706 CRAG QA pairs with required fields: question id, prediction, reference, verdict
- LLM-judge verdicts show ~98% agreement with human annotators on a random sample of 100 examples
- Aggregated scores cover all five CRAG domains (Finance, Sports, Music, Movie, Open) plus an overall accuracy column
- Per-model/per-condition accuracies reproduce the Table 1 / Table 3 layout of the paper

## Limitations

- Judge validity rests on the 100-example human-agreement sample (98%), so verdicts on unusual or borderline answers may diverge from human judgment
- The paper's protocol uses binary semantic-equivalence accuracy, which differs from CRAG's official four-way grading (perfect/acceptable/missing/incorrect) and 1/0/-1 scoring
- Six specialized domains are claimed in the discussion section, but CRAG itself defines five domains, and the discussion does not name the domains or report quantitative metrics there

## Evidence

- [methods] LLM-as-a-judge for semantic equivalence: [evidence span withheld — anchor status: not_found]
- [methods] Judge human agreement: [evidence span withheld — anchor status: not_found]
- [methods] CRAG scale and domains: "CRAG is a comprehensive RAG benchmark containing 2,706 question-answer pairs"
- [readme] Official CRAG auto-evaluation scoring: [evidence span withheld — anchor status: not_found]
- [readme] CRAG evaluation categories: [evidence span withheld — anchor status: not_found]
