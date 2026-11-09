---
name: prompt-variant-benchmarking
description: Use when you must choose a prompt/configuration for an LLM MCQ benchmark,
  or need to quantify how seed, temperature, and prompt complexity affect run-to-run
  consistency of GPT-3.5/GPT-4 outputs.
when_to_use_negative:
- The GPT-3.5-optimized configuration is being applied to a different LLM or VLM family
  — the article notes prompts/settings were optimized only for OpenAI models and reused
  for others, and generalizability remains uncertain; per-model prompt configuration
  may be more effective
- The goal is to improve absolute accuracy rather than measure consistency — the paper
  found that the most consistent configuration did not necessarily improve accuracy
- The evaluation target is real-world dynamic interaction, documentation/extraction
  behavior, or hallucination detection — MCQ accuracy and consistency do not capture
  these
tools:
- name: GPT-3.5
  role: Model under test and the semi-automated option-extraction pipeline applied
    to its own responses
- name: Python 3.11
  role: Driver language for loading run outputs, scoring, and computing accuracy spreads
    with pandas
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes API interactions across the multi-run, multi-prompt-variant experiments
- name: OpenAI Python package
  role: Performs communication with the OpenAI models for each prompt-variant run
- name: pandas
  role: Loads the three run outputs and answer key and writes per-run accuracies and
    variation to consistency_summary.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:948baba8c3d4c1523916f6424bebe04477784516c508bbd498dcef83cbd22bd1
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
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# prompt-variant-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for benchmarking prompt variants (e.g., 'Just QA', QA followed by 'are you sure?', open-ended vs. MCQ with/without justification) and model settings (random seed, temperature, max-token) for reproducible LLM evaluation on gastroenterology MCQs, quantifying run-to-run output consistency as well as accuracy.

## When to use

Use when you must choose a prompt/configuration for an LLM MCQ benchmark, or need to quantify how seed, temperature, and prompt complexity affect run-to-run consistency of GPT-3.5/GPT-4 outputs.

## When NOT to use

- The GPT-3.5-optimized configuration is being applied to a different LLM or VLM family — the article notes prompts/settings were optimized only for OpenAI models and reused for others, and generalizability remains uncertain; per-model prompt configuration may be more effective
- The goal is to improve absolute accuracy rather than measure consistency — the paper found that the most consistent configuration did not necessarily improve accuracy
- The evaluation target is real-world dynamic interaction, documentation/extraction behavior, or hallucination detection — MCQ accuracy and consistency do not capture these

## Inputs

- Three separate GPT-3.5 API run outputs (per-question textual responses) from the Sdamirsa/LLM-VLM-in-Gastroenterology repository
- 2022 ACG-SA answer key (300-question ACG Self-Assessment edition)
- Prompt variant definitions: 'Just QA', QA followed by 'are you sure?', QA + 'answer following question', and open-ended vs. MCQ without/with justification
- Model settings per run: random seed, temperature, max-token, output format

## Outputs

- consistency_summary.csv with per-run accuracies and run-to-run max-minus-min accuracy variation per configuration
- Per-question correct/incorrect labels for each run scored against the 2022 ACG-SA answer key
- A selected prompt/configuration (output format, prompt, temperature, max-token) reused for downstream model benchmarking

## How to apply

First run a parameter-optimization pass on a dataset subset with GPT-3.5 to select the best output format, prompt, temperature, and max-token settings. Then run at least three separate API runs per configuration — simple 'Just QA' versus multi-turn variants such as QA followed by 'are you sure?', with and without a fixed random seed and temperature 0 — using Python 3.11, LangChain (0.1.17/0.2.4), and the OpenAI Python package for standardized API interactions. Extract the selected option from each textual response via the semi-automated GPT-3.5 extraction pipeline with human review of disagreements, score each 2022 ACG-SA question as correct/incorrect against the answer key, and compute overall accuracy per run. Quantify consistency as the max-minus-min accuracy spread across the three runs (the paper reports ~±5% GPT-3.5 variation), then compare spreads across configurations: the fixed-seed, temperature-zero, simple-prompt configuration should yield the smallest spread. Note the paper's caveat that increased consistency did not necessarily improve accuracy, so report both metrics.

## Related tools

- **GPT-3.5** (Model under test and the semi-automated option-extraction pipeline applied to its own responses)
- **Python 3.11** (Driver language for loading run outputs, scoring, and computing accuracy spreads with pandas)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes API interactions across the multi-run, multi-prompt-variant experiments)
- **OpenAI Python package** (Performs communication with the OpenAI models for each prompt-variant run)
- **pandas** (Loads the three run outputs and answer key and writes per-run accuracies and variation to consistency_summary.csv)

## Examples

```
import pandas as pd; df = pd.read_csv('runs.csv'); df.groupby('config')['accuracy'].agg(lambda s: s.max()-s.min()).to_csv('consistency_summary.csv')
```

## Evaluation signals

- Reproduced GPT-3.5 run-to-run accuracy variation falls within approximately ±5 percentage points under the fixed-seed, temperature-zero, simple-prompt configuration
- The fixed-seed, temperature-0, simple-prompt configuration shows the smallest max-minus-min accuracy spread across three runs compared with all other configurations
- consistency_summary.csv contains one row per configuration with per-run accuracies and the run-to-run variation column
- Every question response has a successfully extracted selected option (via GPT-3.5 extraction with human review of disagreements) before scoring
- Each configuration comprises exactly three independent API runs scored against the same 2022 ACG-SA answer key

## Limitations

- The limited number of experimental runs means no average and SD are reported to fully assess model consistency
- The GPT-3.5-derived configuration was reused for other LLMs without thorough validation; per-model prompt configuration may be more effective
- Increased consistency did not necessarily improve accuracy; evaluating responses as simply correct/incorrect loses information (e.g., hallucinations despite a correct option)
- Benchmark outputs are limited to gastroenterology MCQs and may not generalize to all of medicine

## Evidence
