---
name: confidence-interval-estimation
description: Use when comparing MCQ accuracy across LLM configurations (structured
  vs free-text output, prompt variants, temperature, max-tokens) and you must report
  accuracy deltas with 95% CIs instead of bare point estimates.
when_to_use_negative:
- When benchmarking full model families (o1-preview, Claude3.5-Sonnet, Llama3.3-70b)
  on the 162 text-only / 138 image-inclusive 2022 ACG-SA questions — that is head-to-head
  accuracy benchmarking, not parameter-optimization CI estimation
- When the goal is caption-based VLM analysis (human vs LLM-generated captions), where
  the reported effect is a ~10% accuracy change rather than a prompt-configuration
  CI
- When the dataset is not the ACG-SA MCQ set and no per-question correct/incorrect
  labels exist to define a delta
tools:
- name: Python 3.11
  role: Runs the scoring loop that computes per-configuration accuracy deltas and
    their 95% CIs against the ACG-SA answer key
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: GPT-3.5
  role: Model queried across the parameter grid (output format, prompt, temperature,
    max-token) whose accuracy effects are being CI-estimated
- name: OpenAI Python package
  role: Used to communicate with GPT-3.5 to generate the response sets that feed the
    delta calculations
- name: LangChain
  role: Standardizes model interactions so that each configuration variant is queried
    consistently before scoring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:036c27d80f645ab5686d6a020daa07c421ab9ef84777c78b8ef0d804923093f5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  tools:
  - GPT-3.5
  - LangChain
  - OpenAI Python package
  - Python
  - Python 3.11
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# confidence-interval-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute and report 95% confidence intervals on LLM accuracy deltas when benchmarking model settings (e.g., prompt techniques on the 60-question ACG-SA subset), so that observed gains like the +5.00% combined-prompt effect are reported with uncertainty bounds rather than as point estimates.

## When to use

Use when comparing MCQ accuracy across LLM configurations (structured vs free-text output, prompt variants, temperature, max-tokens) and you must report accuracy deltas with 95% CIs instead of bare point estimates.

## When NOT to use

- When benchmarking full model families (o1-preview, Claude3.5-Sonnet, Llama3.3-70b) on the 162 text-only / 138 image-inclusive 2022 ACG-SA questions — that is head-to-head accuracy benchmarking, not parameter-optimization CI estimation
- When the goal is caption-based VLM analysis (human vs LLM-generated captions), where the reported effect is a ~10% accuracy change rather than a prompt-configuration CI
- When the dataset is not the ACG-SA MCQ set and no per-question correct/incorrect labels exist to define a delta

## Inputs

- Per-configuration accuracy scores on the 60-question ACG-SA parameter-optimization subset (correct/incorrect per question vs answer key)
- Baseline configuration accuracy (e.g., free-text output, plain QA prompt)
- Counts of scored responses per configuration

## Outputs

- Accuracy deltas with 95% confidence intervals per factor (e.g., +5.00% [1.11–7.78%] for the combined prompt)
- parameter_optimization_results.csv with per-setting accuracies, deltas, and CI bounds
- Summary figure matching Fig. 1 and Supplementary Tables 3–6

## How to apply

Score each configuration's responses against the ACG-SA answer key, then compute the accuracy delta of each factor (e.g., best combined prompt with all 5 techniques, answer-and-justification prompting) versus the baseline and attach a 95% CI. In this article's Experiment 0, the combined-prompt effect was +5.00% (95% CI 1.11–7.78%) and answer-and-justify was +4.44% (95% CI 2.78–6.11%); structured output gave a 5–10% gain. The number of scored questions (60-question ACG-SA subset for parameter optimization) determines CI width, so keep the per-configuration counts and record them in parameter_optimization_results.csv alongside point estimates. Validate by checking that reproduced intervals bracket the reported values and that the point estimate lies inside its own interval.

## Related tools

- **Python 3.11** (Runs the scoring loop that computes per-configuration accuracy deltas and their 95% CIs against the ACG-SA answer key) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **GPT-3.5** (Model queried across the parameter grid (output format, prompt, temperature, max-token) whose accuracy effects are being CI-estimated)
- **OpenAI Python package** (Used to communicate with GPT-3.5 to generate the response sets that feed the delta calculations)
- **LangChain** (Standardizes model interactions so that each configuration variant is queried consistently before scoring)

## Examples

```
from statsmodels.stats.proportion import proportion_confint; lo, hi = proportion_confint(k_correct, n=60, alpha=0.05)  # 95% CI on a config's accuracy delta
```

## Evaluation signals

- Reproduced combined-prompt effect equals +5.00% with 95% CI 1.11–7.78%
- Reproduced answer-and-justify effect equals +4.44% (95% CI 2.78–6.11%)
- Structured output shows a 5–10% accuracy gain over free-text responses
- Every reported point estimate lies inside its own CI, and CI width scales sensibly with the 60-question subset size
- parameter_optimization_results.csv contains delta + CI bounds for each factor and matches Supplementary Tables 3–6

## Limitations

- The limited number of experimental runs means no average and SD were reported to assess model consistency, inflating CI uncertainty
- The GPT-3.5-derived configuration was reused for other LLMs without per-model CI validation, so generalizability of these deltas is uncertain
- Evaluation is binary correct/incorrect, which loses information and cannot capture hallucinations when the correct option is selected

## Evidence

- [other] combined prompt effect with CI: [evidence span withheld — anchor status: not_found]
- [other] single-technique CI: [evidence span withheld — anchor status: not_found]
- [other] validation target: [evidence span withheld — anchor status: not_found]
