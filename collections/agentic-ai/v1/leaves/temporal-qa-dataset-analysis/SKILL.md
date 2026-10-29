---
name: temporal-qa-dataset-analysis
description: Use when auditing or reproducing the composition of a streaming VideoQA
  benchmark's question taxonomy — specifically when the Dynamic QA Construction schema
  defines six specialized question types and per-type counts must be verified.
when_to_use_negative:
- Input QA pairs come from a different VideoQA dataset without the six-type Dynamic
  QA Construction taxonomy — type mapping would be undefined
- The goal is to analyze reasoning chains (final_cot/ JSONL files) or bounding-box
  annotations rather than the question-type composition of the QA pairs
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: StreamingCoT dataset repository
  role: Source of the dynamic QA-pair annotation files to be parsed and tabulated
  repo: https://github.com/Fleeting-hyh/StreamingCoT
- name: pandas
  role: value_counts tabulation and normalization of question-type frequencies to
    percentages
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:49cde2790d0eeeccd8ec693e84afcb02ca07e6933e12b3c06726498b6f4b023b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - StreamingCoT dataset repository
  - pandas
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# temporal-qa-dataset-analysis

## Summary

Tabulates the distribution of StreamingCoT's 34,470 dynamic QA pairs across its six temporally evolving question types (cumulative counting, periodic pattern recognition, sequential step recognition, state duration, object state recognition, clue-revealing responses), validating against the paper's Section 3.6 reported percentages.

## When to use

Use when auditing or reproducing the composition of a streaming VideoQA benchmark's question taxonomy — specifically when the Dynamic QA Construction schema defines six specialized question types and per-type counts must be verified.

## When NOT to use

- Input QA pairs come from a different VideoQA dataset without the six-type Dynamic QA Construction taxonomy — type mapping would be undefined
- The goal is to analyze reasoning chains (final_cot/ JSONL files) or bounding-box annotations rather than the question-type composition of the QA pairs

## Inputs

- StreamingCoT dynamic QA-pair annotation files (deposited JSON/JSONL records from anonymous.4open.science or github.com/Fleeting-hyh/StreamingCoT)
- Dynamic QA Construction schema defining the six specialized question types

## Outputs

- qa_type_distribution.csv with columns: question type, count, percentage
- Validation report confirming percentages sum to ~100% and match the Section 3.6 reference distribution

## How to apply

Load the deposited StreamingCoT dynamic QA-pair annotation files from the dataset repository (anonymous.4open.science or github.com/Fleeting-hyh/StreamingCoT) using Python. Parse each QA record and map it to one of the six question types defined in the dataset's Dynamic QA Construction schema (cumulative counting, periodic pattern recognition, sequential step recognition, state duration, object state recognition, clue-revealing responses). Tabulate frequencies with pandas value_counts and normalize to percentages of the total QA-pair count, then write the (question type, count, percentage) table to qa_type_distribution.csv. Validate that percentages sum to 100% within rounding and match the reported distribution: cumulative counting 18.2%, periodic pattern recognition 15.7%, sequential step recognition 22.4%, state duration 16.1%, object state recognition 19.3%, clue-revealing responses 8.3%.

## Related tools

- **StreamingCoT dataset repository** (Source of the dynamic QA-pair annotation files to be parsed and tabulated) — https://github.com/Fleeting-hyh/StreamingCoT
- **pandas** (value_counts tabulation and normalization of question-type frequencies to percentages)

## Examples

```
python -c "import pandas as pd; df = pd.read_json('StreamingCoT/final_qa.jsonl', lines=True); counts = df['question_type'].value_counts(); (counts / counts.sum() * 100).round(1).to_csv('qa_type_distribution.csv')"
```

## Evaluation signals

- Total record count equals 34,470 dynamic QA pairs
- Six type percentages sum to 100% (within rounding tolerance)
- Per-type percentages reproduce: cumulative counting 18.2%, periodic pattern recognition 15.7%, sequential step recognition 22.4%, state duration 16.1%, object state recognition 19.3%, clue-revealing responses 8.3%
- Every QA record maps to exactly one of the six schema-defined question types (no unmapped/unknown category in value_counts output)
- qa_type_distribution.csv contains the expected (question type, count, percentage) schema

## Limitations

- The six-type taxonomy is defined by this dataset's Dynamic QA Construction schema; other streaming VideoQA datasets use different or static annotation mechanisms
- Repository contains no changelog, and no license/availability statement was found in the article's discussion section, so annotation-file versions may vary
- Dataset access depends on the anonymous.4open.science portal and YouTube-hosted videos subject to original content creators' rights

## Evidence
