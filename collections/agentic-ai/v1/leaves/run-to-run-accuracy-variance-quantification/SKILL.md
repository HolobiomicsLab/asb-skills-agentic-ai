---
name: run-to-run-accuracy-variance-quantification
description: Use when you have ≥2 independent runs of the same model on an MCQ benchmark
  and need to measure run-to-run accuracy consistency, or when selecting model settings
  (seed, temperature, prompt) that minimize output variability.
when_to_use_negative:
- You are evaluating a single model run — there is no variance to quantify; use plain
  accuracy scoring instead
- You need statistical dispersion measures (mean and SD across many runs) — the article
  explicitly notes this was a limitation with only a limited number of experimental
  runs
- The runs differ in question set, model version, or answer key — spreads would confound
  setting effects with input differences
edam_operation: http://edamontology.org/operation_3799
tools:
- name: GPT-3.5
  role: Model queried in three separate API runs whose run-to-run consistency is quantified;
    also used for semi-automated option extraction from responses
- name: Python 3.11
  role: Implementation language for loading run outputs, scoring against the ACG-SA
    key, and computing accuracy spreads with pandas
- name: OpenAI Python package
  role: Communication channel with the OpenAI models used to generate the repeated
    runs and extraction calls
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes the prompt/API interactions (e.g., 'Just QA' vs. QA followed
    by 'are you sure?') across the repeated runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:35e104ee47300848f884de125af5b0887fa5ef7c65254bcdd68893878c2a3b6e
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
  - LangChain (0.1.17 and 0.2.4)
  - OpenAI Python package
  - Python
  - Python 3.11
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Run-to-run accuracy variance quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies LLM output consistency by re-scoring multiple independent API runs of the same MCQ dataset (e.g., three GPT-3.5 runs on the 2022 ACG-SA) and computing the max-minus-min accuracy spread per configuration, as used to tune seed, temperature, and prompt settings in the gastroenterology LLM benchmark.

## When to use

Use when you have ≥2 independent runs of the same model on an MCQ benchmark and need to measure run-to-run accuracy consistency, or when selecting model settings (seed, temperature, prompt) that minimize output variability.

## When NOT to use

- You are evaluating a single model run — there is no variance to quantify; use plain accuracy scoring instead
- You need statistical dispersion measures (mean and SD across many runs) — the article explicitly notes this was a limitation with only a limited number of experimental runs
- The runs differ in question set, model version, or answer key — spreads would confound setting effects with input differences

## Inputs

- Raw LLM textual responses from ≥2 (typically 3) independent API runs over the same question set
- Answer key for the MCQ dataset (e.g., 2022 ACG-SA answer key)
- Configuration metadata per run (prompt variant, temperature, random seed)

## Outputs

- Per-run accuracy values per configuration
- Run-to-run accuracy spread (max-minus-min, in percentage points) per configuration
- consistency_summary.csv containing per-run accuracies and run-to-run variation per configuration

## How to apply

Load each API run's raw textual responses plus the answer key (e.g., the 2022 ACG-SA key with 300 questions) with Python 3.11 and pandas. Extract the selected option from each response using the paper's semi-automated pipeline — GPT-3.5 extraction followed by human review of disagreements — then score each question correct/incorrect and compute overall accuracy per run under each configuration (e.g., simple 'Just QA' prompt vs. QA followed by 'are you sure?'). Quantify consistency as the max-minus-min accuracy spread across the three runs, targeting the reported ~±5% GPT-3.5 variation. Compare spreads across configurations (temperature 0, fixed random seed, prompt complexity) and write per-run accuracies and variation per configuration to consistency_summary.csv; the most consistent configuration (fixed seed, temperature 0, simpler prompt) is preferred, but note that higher consistency did not necessarily improve accuracy.

## Related tools

- **GPT-3.5** (Model queried in three separate API runs whose run-to-run consistency is quantified; also used for semi-automated option extraction from responses)
- **Python 3.11** (Implementation language for loading run outputs, scoring against the ACG-SA key, and computing accuracy spreads with pandas)
- **OpenAI Python package** (Communication channel with the OpenAI models used to generate the repeated runs and extraction calls)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes the prompt/API interactions (e.g., 'Just QA' vs. QA followed by 'are you sure?') across the repeated runs)

## Examples

```
df = pd.read_csv('consistency_summary.csv'); spread = df.groupby('config')['accuracy'].agg(lambda s: s.max()-s.min())
```

## Evaluation signals

- Under the fixed-seed, temperature-0, simple-prompt configuration, the reproduced GPT-3.5 run-to-run accuracy variation should fall within approximately ±5 percentage points, matching the reported finding
- consistency_summary.csv contains one row per configuration with per-run accuracies (3 runs) and the max-minus-min spread
- Selected-option extraction from raw responses is validated by human review of GPT-3.5 extraction disagreements before scoring
- Spread ordering across configurations should confirm that fixed seed + temperature 0 + simpler prompt yields the smallest spread
- Per-run accuracies should be comparable in magnitude to published GPT-3.5 gastroenterology MCQ scores (e.g., ~55–65% range reported across similar benchmarks)

## Limitations

- Only a limited number of experimental runs (three) were performed; no average and standard deviation were reported to fully assess model consistency
- Increased consistency did not necessarily improve accuracy, so a low spread should not be read as better performance
- The prompt/settings configuration was optimized only for OpenAI (GPT-3.5) and reused for other LLMs; per-model prompt configuration may be more effective and generalizability is uncertain
- Knowledge cutoffs for commercial models changed during experimentation without documented notification, which can affect run comparability

## Evidence
