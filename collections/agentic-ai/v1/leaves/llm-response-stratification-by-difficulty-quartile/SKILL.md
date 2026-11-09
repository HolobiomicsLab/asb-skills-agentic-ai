---
name: llm-response-stratification-by-difficulty-quartile
description: Use when scoring LLM/VLM responses against the ACG-SA 2022 answer key
  and the goal is to test whether model accuracy declines with question difficulty
  as defined by human test-taker accuracy quartiles.
when_to_use_negative:
- The analysis goal is overall model ranking or text-only vs image-inclusive splits
  (162 text-only / 138 image-inclusive questions) rather than difficulty-stratified
  performance
- The dataset is not the ACG-SA 2022 edition or lacks human test-taker quartile labels
  — quartile stratification cannot be computed
- Responses have not yet been scored correct/incorrect against the answer key (scoring
  must precede stratification)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Python 3.11
  role: Loading stored response records with pandas, scoring against the answer key,
    and computing per-model/per-quartile accuracy
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: GPT-3.5
  role: Semi-automated extraction of the selected option from LLM textual responses
    prior to correct/incorrect scoring
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:51fae19a73efb8af93c4909bacbae1d9ea45bf1b4f7cbce454e4e493f1923ae4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  tools:
  - Claude 3.5 Sonnet
  - GPT-3.5
  - Llama 3.3-70b
  - Python
  - Python 3.11
  - Qwen 2.5-72b
  - o1-preview
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# llm-response-stratification-by-difficulty-quartile

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies LLM accuracy on the 2022 ACG-SA multiple-choice questions by human test-taker difficulty quartile (Q1 hardest to Q4 easiest), reproducing the paper's easy/medium/challenging accuracy breakdown (e.g., GPT-4o: 88% Q4, 74% Q2–Q3, 58.7% Q1).

## When to use

Use when scoring LLM/VLM responses against the ACG-SA 2022 answer key and the goal is to test whether model accuracy declines with question difficulty as defined by human test-taker accuracy quartiles.

## When NOT to use

- The analysis goal is overall model ranking or text-only vs image-inclusive splits (162 text-only / 138 image-inclusive questions) rather than difficulty-stratified performance
- The dataset is not the ACG-SA 2022 edition or lacks human test-taker quartile labels — quartile stratification cannot be computed
- Responses have not yet been scored correct/incorrect against the answer key (scoring must precede stratification)

## Inputs

- Stored ACG-SA 2022 response records per model (GPT-4o, o1-preview, Claude 3.5 Sonnet, Llama 3.3-70b, Qwen 2.5-72b) from the Sdamirsa/LLM-VLM-in-Gastroenterology repository
- ACG-SA 2022 answer key for binary correct/incorrect scoring
- Per-question human test-taker difficulty quartile labels (Q1 hardest to Q4 easiest) for the 300-question ACG-SA 2022 edition

## Outputs

- Per-model, per-difficulty-stratum accuracy table (one accuracy value per model per stratum: easy/Q4, medium/Q2–Q3, challenging/Q1)
- Grouped stratified bar chart of accuracy by difficulty quartile across models

## How to apply

Load the stored ACG-SA 2022 response records for each model (e.g., GPT-4o, o1-preview, Claude 3.5 Sonnet, Llama 3.3-70b, Qwen 2.5-72b) from the project repository using Python 3.11 with pandas. Score each extracted selected option as correct or incorrect against the ACG-SA 2022 answer key, replicating the paper's binary correct/incorrect evaluation. Merge each question with its human test-taker difficulty quartile label (Q1–Q4), group responses by model and quartile, and aggregate Q2 and Q3 into a single 'medium' stratum to reproduce the easy/medium/challenging grouping used in the paper. Verify recovered quartile accuracies against the reported values — GPT-4o at 88% on Q4 (easy), 74% on Q2–Q3 (medium), and 58.7% on Q1 (hardest) — within rounding tolerance before rendering stratified bar charts for all models.

## Related tools

- **Python 3.11** (Loading stored response records with pandas, scoring against the answer key, and computing per-model/per-quartile accuracy) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **GPT-3.5** (Semi-automated extraction of the selected option from LLM textual responses prior to correct/incorrect scoring)

## Examples

```
acc = df.groupby(['model','difficulty_stratum'])['correct'].mean().unstack()
```

## Evaluation signals

- GPT-4o's stratified accuracies equal the reported 88% (Q4), 74% (Q2–Q3), and 58.7% (Q1) within rounding tolerance
- The output table contains exactly one accuracy value per model per difficulty stratum (easy/medium/challenging)
- Every response record maps to a valid difficulty quartile label (Q1–Q4) after the merge; no unmatched questions
- Per-stratum denominators sum to 300 questions per model (the full ACG-SA 2022 edition)

## Limitations

- Binary correct/incorrect evaluation discards information; the paper notes 'potential information loss when evaluating LLM responses as simply correct or incorrect'
- MCQ accuracy may not reflect real-world usage; the paper cites 'the lack of access to a more suitable validation dataset'
- Even when a model selects the correct option, it 'may exhibit hallucinations in its response', so stratified accuracy overstates reasoning quality

## Evidence

- [results] GPT-4o difficulty-stratified accuracies: [evidence span withheld — anchor status: not_found]
