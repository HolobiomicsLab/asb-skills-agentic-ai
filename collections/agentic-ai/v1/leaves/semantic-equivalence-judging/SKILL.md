---
name: semantic-equivalence-judging
description: Use when you must score free-form generative QA answers (e.g., CRAG 2,706
  QA pairs) for correctness against reference answers and exact string matching is
  inappropriate.
when_to_use_negative:
- Answers can be scored by rule-based string matching alone — CRAG's auto-evaluation
  also employs rule-based matching before LLM assessment, so an LLM judge is unnecessary
  for exact-match-able answers
- You need graded response quality ratings (perfect / acceptable / missing / incorrect)
  from the original CRAG KDD Cup protocol rather than the binary equivalence verdict
  used by this paper's pipeline
- No reference answer exists for the questions, so no equivalence judgment can be
  made
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: Qwen2.5-72B-Instruct
  role: LLM-as-a-judge queried with per-question prompts to determine semantic equivalence
    between model predictions and reference answers
  repo: https://github.com/sunxin000/TTARAG
- name: CRAG
  role: Evaluation benchmark supplying the 2,706 question-answer pairs, gold references,
    five domains, and scoring protocol (correct/missing/incorrect)
  repo: https://github.com/facebookresearch/CRAG
- name: parallel_evaluation.py
  role: Parallel evaluation and result saving script whose prediction outputs feed
    the judge and whose results summaries receive aggregated scores
  repo: https://github.com/sunxin000/TTARAG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/icassp55912.2026.11464478
    title: Predict the Retrieval! Test Time Adaptation for Retrieval Augmented Generation
  source_content_hash: sha256:a0b6c01777b9d67c83363093e321e9534190f1e11e22d6243fd14ddce6d34407
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sunxin000/TTARAG
  tools:
  - CRAG
  - Qwen2.5-72B-Instruct
  - parallel_evaluation.py
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# semantic-equivalence-judging

## Summary

LLM-as-a-judge evaluation skill: use Qwen2.5-72B-Instruct to decide semantic equivalence between each model prediction and the CRAG gold reference answer, replacing exact string matching for correctness scoring, with 98% agreement with human annotators on a 100-example sample.

## When to use

Use when you must score free-form generative QA answers (e.g., CRAG 2,706 QA pairs) for correctness against reference answers and exact string matching is inappropriate.

## When NOT to use

- Answers can be scored by rule-based string matching alone — CRAG's auto-evaluation also employs rule-based matching before LLM assessment, so an LLM judge is unnecessary for exact-match-able answers
- You need graded response quality ratings (perfect / acceptable / missing / incorrect) from the original CRAG KDD Cup protocol rather than the binary equivalence verdict used by this paper's pipeline
- No reference answer exists for the questions, so no equivalence judgment can be made

## Inputs

- Generated model predictions for the 2,706 CRAG question-answer pairs (predictions/ output of parallel_evaluation.py)
- CRAG QA pairs with gold reference answers (facebookresearch/CRAG dataset, schema per docs/dataset.md)
- Judge prompt template from prompts/ evaluation prompt templates

## Outputs

- judge_verdicts.csv with one record per question containing question id, prediction, reference, and verdict
- Aggregated per-domain and overall accuracy scores across the five CRAG domains (Finance, Sports, Music, Movie, Open)
- Summaries under results/<dataset>/

## How to apply

Load the generated model predictions together with the 2,706 CRAG question-answer pairs and their gold reference answers. For each question, construct a judge prompt following the CRAG evaluation protocol that presents the model prediction and the reference answer and asks the judge model to decide semantic equivalence rather than exact string match; CRAG scoring assigns correct (1 point), missing (0 points), or incorrect (-1 point). Query Qwen2.5-72B-Instruct with each prompt, parse its verdict (equivalent / not equivalent) into a per-question record containing question id, prediction, reference, and verdict, then aggregate per-domain and overall accuracy across the five CRAG domains (Finance, Sports, Music, Movie, Open). Validate by sampling 100 random examples and checking the judge's agreement against human annotators; the paper reports 98% agreement for this judge configuration. Persist the structured verdict records to judge_verdicts.csv alongside the aggregated scores.

## Related tools

- **Qwen2.5-72B-Instruct** (LLM-as-a-judge queried with per-question prompts to determine semantic equivalence between model predictions and reference answers) — https://github.com/sunxin000/TTARAG
- **CRAG** (Evaluation benchmark supplying the 2,706 question-answer pairs, gold references, five domains, and scoring protocol (correct/missing/incorrect)) — https://github.com/facebookresearch/CRAG
- **parallel_evaluation.py** (Parallel evaluation and result saving script whose prediction outputs feed the judge and whose results summaries receive aggregated scores) — https://github.com/sunxin000/TTARAG

## Examples

```
python parallel_evaluation.py --do_generate --dataset pubmedqa --model_name models/Llama-3.1-8B-Instruct --do_tta --learning_rate 1e-5 --max_adapt_pairs 3 --cross_encoder_name models/ms-marco-MiniLM-L-6-v2
```

## Evaluation signals

- judge_verdicts.csv exists and contains a verdict record for each of the 2,706 CRAG QA pairs with required fields (question id, prediction, reference, verdict)
- LLM-judge verdicts show ~98% agreement with human annotators on a random sample of 100 examples
- Aggregated scores cover all five CRAG domains (Finance, Sports, Music, Movie, Open) plus an overall score
- Verdicts parse cleanly into equivalent / not equivalent values (no unparsed judge outputs)
- Per-question verdicts map to CRAG point semantics (correct = 1 point, missing = 0, incorrect = -1) when computing accuracy

## Limitations

- Judge validity rests on a 100-example human-agreement sample (98%), not on exhaustive verification over all 2,706 pairs
- Processing 2,706 queries requires substantial compute — the paper used one NVIDIA A100 GPU
- Automatic evaluation combines rule-based matching and LLM assessment; relying on the LLM judge alone may misclassify answers the CRAG protocol would handle via rule-based matching

## Evidence

- [methods] LLM-as-a-judge semantic equivalence: [evidence span withheld — anchor status: not_found]
- [methods] Human agreement of the judge: [evidence span withheld — anchor status: not_found]
- [readme] CRAG scoring rubric: [evidence span withheld — anchor status: not_found]
- [readme] CRAG auto-evaluation mixes rules and LLM: [evidence span withheld — anchor status: not_found]
