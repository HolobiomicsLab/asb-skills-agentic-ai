---
name: benchmark-stratified-performance-analysis
description: Use when per-model MCQ benchmark accuracies exist alongside human difficulty
  metadata per question and you need to test whether model performance declines with
  question difficulty.
when_to_use_negative:
- The goal is overall benchmark accuracy comparison across model families or quantization
  levels (e.g. 8-bit Llama 3.2-11b vs 6-bit Phi3-14b) rather than difficulty-stratified
  breakdown.
- Questions lack human test-taker difficulty metadata, so quartile stratification
  cannot be assigned (e.g. non-ACG-SA sets like StatPearls or USMLE SA listed in the
  README with 'None' stratified performance).
- Evaluation requires granular response analysis beyond binary correct/incorrect,
  where the paper notes 'information loss when evaluating LLM responses as simply
  correct or incorrect'.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_3315
tools:
- name: Python 3.11
  role: Pandas-based loading, answer-key scoring, quartile merging, and per-model/per-quartile
    accuracy computation
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: GPT-4o
  role: Primary model whose stratified accuracies (88%/74%/58.7%) serve as the validation
    target for the stratification pipeline
- name: o1-preview
  role: Benchmarked model included in the per-model, per-quartile accuracy table
- name: Claude 3.5 Sonnet
  role: Benchmarked model included in the stratified accuracy comparison
- name: Llama 3.3-70b
  role: Top open-source model included in the stratified accuracy comparison
  repo: Meta-llama/Llama-3.3-70B-Instruct
- name: Qwen 2.5-72b
  role: Benchmarked open-source model included in the stratified accuracy comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:c15b93121e3beead7e26bf268ae32a7d9a9af9b949068ea3f150eb329c743e9a
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# benchmark-stratified-performance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies LLM benchmark accuracy on the 2022 ACG-SA MCQ dataset by human test-taker difficulty quartile (Q1 hardest to Q4 easiest), grouping quartiles 2 and 3 into a 'medium' stratum to reproduce an easy/medium/challenging accuracy breakdown per model (e.g. GPT-4o: 88% easy / 74% medium / 58.7% challenging).

## When to use

Use when per-model MCQ benchmark accuracies exist alongside human difficulty metadata per question and you need to test whether model performance declines with question difficulty.

## When NOT to use

- The goal is overall benchmark accuracy comparison across model families or quantization levels (e.g. 8-bit Llama 3.2-11b vs 6-bit Phi3-14b) rather than difficulty-stratified breakdown.
- Questions lack human test-taker difficulty metadata, so quartile stratification cannot be assigned (e.g. non-ACG-SA sets like StatPearls or USMLE SA listed in the README with 'None' stratified performance).
- Evaluation requires granular response analysis beyond binary correct/incorrect, where the paper notes 'information loss when evaluating LLM responses as simply correct or incorrect'.

## Inputs

- Stored 2022 ACG-SA response records per model (GPT-4o, o1-preview, Claude 3.5 Sonnet, Llama 3.3-70b, Qwen 2.5-72b) from github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- ACG-SA 2022 answer key (300 questions; 162 text-only, 138 image-inclusive)
- Per-question human test-taker difficulty quartile labels (Q1–Q4)

## Outputs

- Per-model, per-difficulty-stratum accuracy table (easy/Q4, medium/Q2–Q3, challenging/Q1)
- Grouped stratified bar chart of accuracy across difficulty strata for all models

## How to apply

Load stored model response records (GPT-4o, o1-preview, Claude 3.5 Sonnet, Llama 3.3-70b, Qwen 2.5-72b) from the project repository using Python 3.11 with pandas. Score each response as correct or incorrect against the ACG-SA 2022 answer key, replicating the paper's binary evaluation of extracted selected options. Merge each question with its human test-taker difficulty quartile label (Q1–Q4), group responses by model and quartile, and aggregate Q2 and Q3 into a single 'medium' stratum to form easy/medium/challenging tiers. Compute per-model, per-quartile accuracy; the decision point is that Q2–Q3 pooling is a reproduction choice, not a re-derivation of quartiles. Validate by checking that GPT-4o's stratified accuracies equal the reported 88% (Q4) / 74% (Q2–Q3) / 58.7% (Q1) within rounding tolerance, then render grouped stratified bars for all models.

## Related tools

- **Python 3.11** (Pandas-based loading, answer-key scoring, quartile merging, and per-model/per-quartile accuracy computation) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **GPT-4o** (Primary model whose stratified accuracies (88%/74%/58.7%) serve as the validation target for the stratification pipeline)
- **o1-preview** (Benchmarked model included in the per-model, per-quartile accuracy table)
- **Claude 3.5 Sonnet** (Benchmarked model included in the stratified accuracy comparison)
- **Llama 3.3-70b** (Top open-source model included in the stratified accuracy comparison) — Meta-llama/Llama-3.3-70B-Instruct
- **Qwen 2.5-72b** (Benchmarked open-source model included in the stratified accuracy comparison)

## Examples

```
import pandas as pd; df = pd.read_parquet('acg_sa_2022_responses.parquet'); acc = df.groupby(['model', 'difficulty_quartile'])['correct'].mean(); print(acc.loc['GPT-4o'])
```

## Evaluation signals

- GPT-4o quartile accuracies equal the reported 88% (Q4), 74% (Q2–Q3), and 58.7% (Q1) within rounding tolerance
- Output table contains exactly one accuracy value per model per difficulty stratum (easy, medium, challenging)
- Monotonic accuracy ordering holds per model: Q4 ≥ Q2–Q3 ≥ Q1, with GPT-4o declining from 88% to 58.7%
- Every scored response traces to a valid ACG-SA 2022 answer-key option; no unscored or ambiguous records remain after the binary correct/incorrect merge
- Quartile labels cover the full 300-question 2022 ACG-SA edition with no unmatched questions after merging

## Limitations

- Difficulty quartiles are derived from human test-taker accuracy on a paywalled exam, so the difficulty construct may not transfer to other MCQ datasets or model populations.
- Binary correct/incorrect scoring loses information; the paper notes 'potential information loss when evaluating LLM responses as simply correct or incorrect'.
- Stratification shown for GPT-4o is validated only against that model's reported values; other models' quartile results are reproduced without independent ground truth.
- The ACG-SA questions are gastroenterology-specific; stratified difficulty effects may not generalize to all of medicine ('our benchmark is limited to questions pertaining to gastroenterology').

## Evidence

- [results] task_008 finding: [evidence span withheld — anchor status: not_found]
- [discussion] evaluation method: [evidence span withheld — anchor status: not_found]
- [readme] difficulty stratification scope: "| Gastroenterology; Suchman et al. | 455 board SA (ACG )                       | Excluded                             | • Accuaracy                                                             | •"
- [discussion] dataset composition: "we evaluated the performance of all LLMs on both 162 text-only and 138 image-inclusive questions in the 2022 ACG-SA"
- [discussion] evaluation information loss caveat: "We have already noted the potential information loss when evaluating LLM responses as simply correct or incorrect"
