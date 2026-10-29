---
name: grouped-bar-chart-visualization-of-model-accuracy
description: Use when multiple LLMs have been scored correct/incorrect on a stratified
  MCQ benchmark (e.g., human difficulty quartiles Q1–Q4) and you need a per-model,
  per-stratum accuracy comparison plot.
when_to_use_negative:
- You need stratification by sub-category, taxonomy, or question year rather than
  human difficulty quartiles (the article used quartile-based difficulty for this
  Fig. 4 analysis)
- Only a single model or a single difficulty stratum is present — a grouped comparison
  adds no information
- Responses are graded on a continuous scale rather than the paper's binary correct/incorrect
  evaluation of extracted selected options
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3077
- http://edamontology.org/topic_3399
tools:
- name: Python 3.11
  role: Loading stored ACG-SA 2022 response records and computing per-model, per-quartile
    accuracy with pandas before plotting
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: GPT-4o
  role: Validation model whose reported stratified accuracies (88% / 74% / 58.7%)
    anchor the correctness check of the rendered chart
- name: Llama 3.3-70b
  role: Open-source model included as a comparison bar in the grouped stratified visualization
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:488e7a9d748f70f14b3d26fc7cb88b859139161fda061ad5d43edf1fa0b8f4b4
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
  - GPT-4o
  - Llama 3.3-70b
  - Python
  - Python 3.11
  - Qwen 2.5-72b
  - o1-preview
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3077
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Grouped bar chart visualization of model accuracy

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Renders per-model, per-stratum accuracy as grouped bars so that difficulty-stratified LLM performance on the 2022 ACG-SA (e.g., GPT-4o's 88% easy vs. 58.7% hardest quartile) can be compared across models at a glance.

## When to use

Use when multiple LLMs have been scored correct/incorrect on a stratified MCQ benchmark (e.g., human difficulty quartiles Q1–Q4) and you need a per-model, per-stratum accuracy comparison plot.

## When NOT to use

- You need stratification by sub-category, taxonomy, or question year rather than human difficulty quartiles (the article used quartile-based difficulty for this Fig. 4 analysis)
- Only a single model or a single difficulty stratum is present — a grouped comparison adds no information
- Responses are graded on a continuous scale rather than the paper's binary correct/incorrect evaluation of extracted selected options

## Inputs

- Stored LLM response records per question for each model (from the project repository)
- ACG-SA 2022 answer key for binary correct/incorrect scoring
- Per-question human test-taker difficulty quartile labels (Q1–Q4)

## Outputs

- Grouped bar chart with one accuracy bar per model per difficulty stratum (easy/Q4, medium/Q2–Q3, challenging/Q1)
- Per-model, per-quartile accuracy table (one accuracy value per model per stratum)

## How to apply

Load the stored response records (e.g., GPT-4o, o1-preview, Claude 3.5 Sonnet, Llama 3.3-70b, Qwen 2.5-72b on the 2022 ACG-SA) from the project repository with Python 3.11 and pandas, then score each response binary correct/incorrect against the ACG-SA 2022 answer key. Merge each question with its human test-taker difficulty quartile label (Q1–Q4, Q1 hardest) and group responses by model and quartile. Aggregate quartiles 2 and 3 into a 'medium' stratum to reproduce the paper's easy/medium/challenging grouping, then compute per-model, per-stratum accuracy. Render grouped bars (one bar per model per stratum) and validate by checking that the recovered GPT-4o values match the reported 88% (Q4 easy), 74% (Q2–Q3 medium), and 58.7% (Q1 hardest) within rounding tolerance.

## Related tools

- **Python 3.11** (Loading stored ACG-SA 2022 response records and computing per-model, per-quartile accuracy with pandas before plotting) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **GPT-4o** (Validation model whose reported stratified accuracies (88% / 74% / 58.7%) anchor the correctness check of the rendered chart)
- **Llama 3.3-70b** (Open-source model included as a comparison bar in the grouped stratified visualization) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology

## Examples

```
import pandas as pd; acc = df.groupby(['model','quartile']).correct.mean().unstack(); acc[['Q4','Q2','Q3','Q1']].plot.bar()
```

## Evaluation signals

- GPT-4o's stratified accuracies equal the reported values (88% Q4, 74% Q2–Q3, 58.7% Q1) within rounding tolerance
- Output table contains exactly one accuracy value per model per difficulty stratum (easy, medium, challenging)
- Each question maps to exactly one quartile label (Q1–Q4) and Q2+Q3 are pooled into the medium stratum with no double-counting
- Rendered grouped bars reproduce the monotonic difficulty trend (accuracy decreasing from Q4 to Q1) for GPT-4o
- All models' overall accuracies reconcile with reported figures (e.g., o1-preview 82.0%, Claude 3.5 Sonnet 74.0%, Llama3.3-70b 65.7%, Qwen-2.5-72b 61.0%) when strata are recombined

## Limitations

- Difficulty strata are defined by human test-taker accuracy quartiles on the ACG-SA, not by an intrinsic question-property metric
- Evaluation is binary correct/incorrect on the extracted selected option, discarding information about reasoning quality
- Only the 2022 ACG-SA edition (162 text-only and 138 image-inclusive questions) anchors this stratification; other editions or benchmarks may need different strata

## Evidence

- [results] GPT-4o's performance declined with increasing question difficulty, scoring 88% on easy (quartile 4) questions, 74% on medium difficulty (quartiles 2 and 3), and 58.7% on the hardest (quartile 1) questions.: [evidence span withheld — anchor status: not_found]
