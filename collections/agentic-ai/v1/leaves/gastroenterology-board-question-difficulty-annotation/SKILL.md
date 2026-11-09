---
name: gastroenterology-board-question-difficulty-annotation
description: Use when MCQ responses from models such as GPT-4o, o1-preview, or Claude
  3.5 Sonnet on the 2022 ACG-SA are already scored correct/incorrect and each question
  carries a human-difficulty quartile label (Q1–Q4) that must be joined for stratified
  accuracy analysis.
when_to_use_negative:
- Image-inclusive questions are under VLM captioning experiments rather than text-only
  difficulty stratification — this skill assumes the 162 text-only ACG-SA 2022 questions
- Questions lack human test-taker quartile annotations, so difficulty cannot be defined
  independently of model performance
- The goal is model-vs-model comparison of overall accuracy (e.g., 82.0% o1-preview
  vs 65.7% Llama3.3-70b) without difficulty stratification
edam_operation: http://edamontology.org/operation_0336
tools:
- name: Python 3.11
  role: Runtime for pandas-based merging of quartile labels with scored responses
    and computation of per-model, per-quartile accuracy
- name: GPT-4o
  role: Model whose quartile-stratified accuracies (88% Q4 / 74% Q2-Q3 / 58.7% Q1)
    serve as the validation target
- name: Llama 3.3-70b
  role: Open-source model included in the stratified accuracy comparison across difficulty
    quartiles
  repo: Meta-llama/Llama-3.3-70B-Instruct
- name: Claude 3.5 Sonnet
  role: Proprietary model included in the stratified accuracy comparison across difficulty
    quartiles
- name: Qwen 2.5-72b
  role: Open-source model included in the stratified accuracy comparison across difficulty
    quartiles
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:52adb34477b2fe4a7b6478a757eec6facc08ad5a26ad0dc5e419e6892668642b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  repo_url: https://github.com/Meta-llama/Llama-3.3-70B-Instruct
  tools:
  - Claude 3.5 Sonnet
  - GPT-4o
  - Llama 3.3-70b
  - Python
  - Python 3.11
  - Qwen 2.5-72b
  - o1-preview
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# gastroenterology-board-question-difficulty-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies LLM performance on the 2022 ACG-SA gastroenterology board questions by human test-taker difficulty quartiles (Q1 hardest to Q4 easiest), reproducing the paper's easy/medium/challenging grouping of model accuracy.

## When to use

Use when MCQ responses from models such as GPT-4o, o1-preview, or Claude 3.5 Sonnet on the 2022 ACG-SA are already scored correct/incorrect and each question carries a human-difficulty quartile label (Q1–Q4) that must be joined for stratified accuracy analysis.

## When NOT to use

- Image-inclusive questions are under VLM captioning experiments rather than text-only difficulty stratification — this skill assumes the 162 text-only ACG-SA 2022 questions
- Questions lack human test-taker quartile annotations, so difficulty cannot be defined independently of model performance
- The goal is model-vs-model comparison of overall accuracy (e.g., 82.0% o1-preview vs 65.7% Llama3.3-70b) without difficulty stratification

## Inputs

- Stored 2022 ACG-SA response records per model (pandas-readable files from github.com/Sdamirsa/LLM-VLM-in-Gastroenterology)
- ACG-SA 2022 answer key for binary correct/incorrect scoring
- Per-question human test-taker difficulty quartile labels (Q1–Q4)

## Outputs

- Per-model, per-difficulty-stratum accuracy table (one accuracy value per model per easy/medium/challenging stratum)
- Grouped stratified bar chart of accuracy across difficulty quartiles for all models

## How to apply

Load the stored ACG-SA 2022 response records for each model (e.g., GPT-4o, o1-preview, Claude 3.5 Sonnet, Llama 3.3-70b, Qwen 2.5-72b) from the project repository using Python 3.11 with pandas. Score each response as correct or incorrect against the ACG-SA 2022 answer key, replicating the paper's binary correct/incorrect evaluation of extracted selected options. Merge each question with its human test-taker difficulty quartile label (Q1–Q4) and group responses by model and quartile, aggregating Q2 and Q3 into a single 'medium' stratum to reproduce the easy/medium/challenging grouping. Compute per-model, per-quartile accuracy; the article's benchmark case is GPT-4o declining from 88% on easy (Q4) to 74% on medium (Q2–Q3) to 58.7% on the hardest (Q1).

## Related tools

- **Python 3.11** (Runtime for pandas-based merging of quartile labels with scored responses and computation of per-model, per-quartile accuracy)
- **GPT-4o** (Model whose quartile-stratified accuracies (88% Q4 / 74% Q2-Q3 / 58.7% Q1) serve as the validation target)
- **Llama 3.3-70b** (Open-source model included in the stratified accuracy comparison across difficulty quartiles) — Meta-llama/Llama-3.3-70B-Instruct
- **Claude 3.5 Sonnet** (Proprietary model included in the stratified accuracy comparison across difficulty quartiles)
- **Qwen 2.5-72b** (Open-source model included in the stratified accuracy comparison across difficulty quartiles)

## Examples

```
import pandas as pd; acc = df.groupby(['model','quartile'])['correct'].mean(); med = pd.concat([acc.xs('Q2', level=1), acc.xs('Q3', level=1)]).groupby('model').mean()
```

## Evaluation signals

- GPT-4o's stratified accuracies equal the reported 88% (Q4), 74% (Q2-Q3), and 58.7% (Q1) within rounding tolerance
- The output table contains exactly one accuracy value per model per difficulty stratum (easy/medium/challenging)
- Each 2022 ACG-SA question is joined to exactly one quartile label (Q1–Q4) before aggregation
- Binary correct/incorrect scoring against the ACG-SA 2022 answer key reproduces the paper's per-question evaluation of extracted selected options

## Limitations

- Binary correct/incorrect evaluation loses information about reasoning quality — 'We have already noted the potential information loss when evaluating LLM responses as simply correct or incorrect'
- The benchmark is limited to gastroenterology questions and 'may not generalize to all of medicine'
- Difficulty is defined only by human test-taker accuracy quartiles; no alternative difficulty taxonomy (e.g., Bloom level) is available in this workflow

## Evidence

- [results] GPT-4o's performance declined with increasing question difficulty, scoring 88% on easy (quartile 4) questions, 74% on medium difficulty (quartiles 2 and 3), and 58.7% on the hardest (quartile 1) questions.: [evidence span withheld — anchor status: not_found]
