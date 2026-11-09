---
name: videoqa-annotation-parsing
description: Use when analyzing StreamingCoT dynamic QA annotations to tabulate or
  verify question-type frequencies, or when auditing a VideoQA dataset's temporal
  question taxonomy.
when_to_use_negative:
- The goal is reasoning-chain (CoT) analysis rather than dynamic QA-pair question
  typing — use the final_cot/ JSONL chains instead
- Input is static VideoQA annotations, not the temporally evolving six-type StreamingCoT
  QA taxonomy
edam_operation: http://edamontology.org/operation_2940
tools:
- name: pandas
  role: value_counts tabulation and normalization of parsed QA records into a percentage
    distribution
- name: StreamingCoT dataset
  role: source of the 34,470 dynamic QA-pair annotation files and the six-type question
    schema
  repo: https://github.com/Fleeting-hyh/StreamingCoT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1145/3746027.3758311
    title: 'StreamingCoT: A Dataset for Temporal Dynamics and Multimodal Chain-of-Thought
      Reasoning in Streaming VideoQA'
  source_content_hash: sha256:a72c376ebb357b6a379b110f0598c3a0081fbd205f8861d42f3526e1cd9c964d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Fleeting-hyh/StreamingCoT
  tools:
  - StreamingCoT dataset
  - pandas
  edam_operation: http://edamontology.org/operation_2940
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# videoqa-annotation-parsing

## Summary

Parse StreamingCoT dynamic QA-pair annotation files and reproduce the distribution of the 34,470 QA pairs across the six temporally evolving question types, validating against the Section 3.6 reported percentages.

## When to use

Use when analyzing StreamingCoT dynamic QA annotations to tabulate or verify question-type frequencies, or when auditing a VideoQA dataset's temporal question taxonomy.

## When NOT to use

- The goal is reasoning-chain (CoT) analysis rather than dynamic QA-pair question typing — use the final_cot/ JSONL chains instead
- Input is static VideoQA annotations, not the temporally evolving six-type StreamingCoT QA taxonomy

## Inputs

- StreamingCoT dynamic QA-pair annotation files (JSONL records) from the dataset repository
- Dynamic QA Construction schema defining the six question types

## Outputs

- qa_type_distribution.csv with columns (question type, count, percentage)
- Sanity-check report confirming percentages sum to 100% and match the reported six-type distribution

## How to apply

Load the deposited StreamingCoT dynamic QA-pair annotation files from the dataset repository (anonymous.4open.science or github.com/Fleeting-hyh/StreamingCoT) and parse each JSONL record. Map every QA pair to one of the six specialized question types defined in the dataset's Dynamic QA Construction schema: cumulative counting, periodic pattern recognition, sequential step recognition, state duration measurement, object state recognition, and clue-revealing responses. Tabulate frequencies with pandas value_counts and normalize to percentages of the total QA-pair count (expected total: 34,470). Write the resulting (question type, count, percentage) table to qa_type_distribution.csv and validate that percentages sum to 100% within rounding and reproduce the Section 3.6 distribution: cumulative counting 18.2%, periodic pattern recognition 15.7%, sequential step recognition 22.4%, state duration 16.1%, object state recognition 19.3%, clue-revealing responses 8.3%.

## Related tools

- **pandas** (value_counts tabulation and normalization of parsed QA records into a percentage distribution)
- **StreamingCoT dataset** (source of the 34,470 dynamic QA-pair annotation files and the six-type question schema) — https://github.com/Fleeting-hyh/StreamingCoT

## Examples

```
df = pd.read_json('final_qa_pairs.jsonl', lines=True); dist = df['question_type'].value_counts(normalize=True).mul(100).round(1); dist.to_csv('qa_type_distribution.csv')
```

## Evaluation signals

- Total parsed QA pairs equals 34,470
- Percentages sum to 100% within rounding tolerance
- Reproduces the reported distribution exactly: cumulative counting 18.2%, periodic pattern recognition 15.7%, sequential step recognition 22.4%, state duration 16.1%, object state recognition 19.3%, clue-revealing responses 8.3%
- Every QA record maps to exactly one of the six question types (no unmapped or ambiguous labels)

## Limitations

- The anonymous.4open.science mirror may change or expire; the README notes no changelog is available for the repository
- QA annotations are human-verified but mapping to the six types may be ambiguous for mixed-type questions, requiring manual expert review

## Evidence
