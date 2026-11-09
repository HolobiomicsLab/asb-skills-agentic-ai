---
name: question-type-taxonomy-mapping
description: Use when you need to classify StreamingCoT dynamic QA-pair annotation
  files by question type and reproduce or extend the paper's reported distribution
  over the six temporally evolving question types.
when_to_use_negative:
- The QA corpus is not from StreamingCoT or does not conform to its six-type Dynamic
  QA Construction schema — other taxonomies will not align with the reported distribution
- The analysis target is the multimodal reasoning-chain (CoT) annotations rather than
  the dynamic question-answer pairs
- Records cannot be unambiguously assigned to one of the six question types (e.g.
  truncated or corrupted annotation files) — mapping without a resolution rule would
  corrupt the distribution
tools:
- name: pandas
  role: Parse QA-pair records and tabulate/normalize question-type frequencies via
    value_counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:40e8d26e7af3fe15be8b5d1734b4a44753e62480ab6bd89f84eb445137311370
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# question-type-taxonomy-mapping

## Summary

Maps StreamingCoT's 34,470 dynamic question-answer pairs onto the dataset's six-type question taxonomy (cumulative counting, periodic pattern recognition, sequential step recognition, state duration, object state recognition, clue-revealing responses) and tabulates their frequency distribution, as reported in Section 3.6 of the paper.

## When to use

Use when you need to classify StreamingCoT dynamic QA-pair annotation files by question type and reproduce or extend the paper's reported distribution over the six temporally evolving question types.

## When NOT to use

- The QA corpus is not from StreamingCoT or does not conform to its six-type Dynamic QA Construction schema — other taxonomies will not align with the reported distribution
- The analysis target is the multimodal reasoning-chain (CoT) annotations rather than the dynamic question-answer pairs
- Records cannot be unambiguously assigned to one of the six question types (e.g. truncated or corrupted annotation files) — mapping without a resolution rule would corrupt the distribution

## Inputs

- StreamingCoT dynamic QA-pair annotation files (per-video JSON/JSONL records, e.g. final_cot/VIDEO_ID.jsonl) downloaded from github.com/Fleeting-hyh/StreamingCoT or the anonymous.4open.science mirror

## Outputs

- qa_type_distribution.csv with columns (question type, count, percentage) over the 34,470 dynamic QA pairs
- Validated percentage breakdown matching Section 3.6: 18.2% cumulative counting, 15.7% periodic pattern recognition, 22.4% sequential step recognition, 16.1% state duration, 19.3% object state recognition, 8.3% clue-revealing responses

## How to apply

Load the deposited StreamingCoT dynamic QA-pair annotation files from the repository (anonymous.4open.science / github.com/Fleeting-hyh/StreamingCoT), which are stored as per-video JSON/JSONL records (e.g. final_cot/VIDEO_ID.jsonl). Parse each QA record and map it to one of the six specialized question types defined in the dataset's Dynamic QA Construction schema: cumulative counting, periodic pattern recognition, sequential step recognition, state duration measurement, object state recognition, and clue-revealing responses. Tabulate frequencies with pandas value_counts and normalize to percentages of the 34,470 total pairs, writing a (question type, count, percentage) table. Validate by checking the percentages sum to ~100% and match the Section 3.6 distribution: cumulative counting 18.2%, periodic pattern recognition 15.7%, sequential step recognition 22.4%, state duration 16.1%, object state recognition 19.3%, clue-revealing responses 8.3%.

## Related tools

- **pandas** (Parse QA-pair records and tabulate/normalize question-type frequencies via value_counts)

## Examples

```
python -c "import pandas as pd; df=pd.read_json('final_cot/VIDEO_ID.jsonl',lines=True); print(df['question_type'].value_counts(normalize=True)*100)"
```

## Evaluation signals

- Sum of the six question-type percentages equals 100% (within rounding)
- Reproduced distribution matches the Section 3.6 values: 18.2%, 15.7%, 22.4%, 16.1%, 19.3%, 8.3% for cumulative counting, periodic pattern recognition, sequential step recognition, state duration, object state recognition, and clue-revealing responses respectively
- Total tabulated QA-pair count equals 34,470
- Every parsed QA record maps to exactly one of the six taxonomy types with no unclassified remainder
- Output CSV has the schema (question type, count, percentage) with counts and percentages mutually consistent

## Limitations

- Only the StreamingCoT corpus (5,745 videos, 34,470 dynamic QA pairs) defines this six-type taxonomy; percentages are corpus-specific and not generalizable
- A public changelog for the StreamingCoT repository is not available, so schema evolution between paper versions and repo snapshots cannot be tracked
- No explicit license statement was found in the paper discussion section; the README states the dataset is released for non-commercial research and videos remain subject to YouTube's Terms of Service

## Evidence
