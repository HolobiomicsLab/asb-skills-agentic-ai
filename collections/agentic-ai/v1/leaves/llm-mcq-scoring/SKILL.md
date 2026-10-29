---
name: llm-mcq-scoring
description: Use when raw free-text LLM responses to an MCQ exam (e.g., ACG-SA with
  a known answer key) must be converted into per-question correct/incorrect labels
  and per-run accuracies, and consistency across repeated runs must be quantified.
when_to_use_negative:
- The benchmark questions are not MCQs with a verifiable answer key (e.g., open-ended
  clinical reasoning or documentation tasks) — the paper notes MCQ accuracy is a proxy
  that misses hallucinations and dynamic interactions
- You need average and SD of consistency across many runs — the article flags the
  limited number of experimental runs as a limitation, so this skill supports only
  the three-run max-minus-min spread design
- The goal is to evaluate the option-extraction step itself rather than score answers
  — extraction is semi-automated and requires human review of disagreements
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3379
- http://edamontology.org/topic_3433
tools:
- name: Python 3.11
  role: Used for API interactions and pandas-based scoring of run outputs against
    the ACG-SA answer key
  repo: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: GPT-3.5
  role: Model whose repeated runs are scored; also used to extract the selected option
    from each textual response in the semi-automated pipeline
- name: OpenAI Python package
  role: Communication with the OpenAI (GPT-3.5) models when re-running the API runs
    to be scored
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes the GPT-3.5 API interactions so prompt variants (e.g., 'Just
    QA', QA followed by 'are you sure?') are applied consistently across runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:87bf9a73a633d81bf7d6451709cbaee260a6dc2e51043cb65a6ecbbecb79a888
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  repo_url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  tools:
  - GPT-3.5
  - LangChain (0.1.17 and 0.2.4)
  - OpenAI Python package
  - Python
  - Python 3.11
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3379
  - http://edamontology.org/topic_3433
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# llm-mcq-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores LLM answers to multiple-choice medical questions (correct/incorrect) against an official answer key and quantifies run-to-run consistency, as done for GPT-3.5 on the 2022 ACG Self-Assessment. It separates accuracy measurement from consistency measurement (max-minus-min spread across repeated API runs).

## When to use

Use when raw free-text LLM responses to an MCQ exam (e.g., ACG-SA with a known answer key) must be converted into per-question correct/incorrect labels and per-run accuracies, and consistency across repeated runs must be quantified.

## When NOT to use

- The benchmark questions are not MCQs with a verifiable answer key (e.g., open-ended clinical reasoning or documentation tasks) — the paper notes MCQ accuracy is a proxy that misses hallucinations and dynamic interactions
- You need average and SD of consistency across many runs — the article flags the limited number of experimental runs as a limitation, so this skill supports only the three-run max-minus-min spread design
- The goal is to evaluate the option-extraction step itself rather than score answers — extraction is semi-automated and requires human review of disagreements

## Inputs

- Three (or more) separate LLM API run outputs (free-text responses to MCQs) from the LLM-VLM-in-Gastroenterology repository
- Official answer key (2022 ACG Self-Assessment)
- Prompt/configuration metadata per run (e.g., 'Just QA' vs. QA + 'are you sure?', temperature 0, fixed random seed)

## Outputs

- Per-question correct/incorrect labels per run
- Per-run overall accuracy per configuration
- Run-to-run consistency metric (max-minus-min accuracy spread across runs, e.g., ~±5 percentage points for GPT-3.5)
- consistency_summary.csv written with pandas

## How to apply

Load each separate API run's output (e.g., the three GPT-3.5 run outputs from the Sdamirsa/LLM-VLM-in-Gastroenterology repository) plus the official answer key (2022 ACG-SA) using Python 3.11 and pandas. Extract the selected option from each free-text response with the semi-automated pipeline — GPT-3.5-based option extraction with human review of disagreements. Score each question against the key and compute overall accuracy per run under each configuration (e.g., 'Just QA' vs. QA followed by 'are you sure?', temperature 0, fixed random seed). Quantify consistency as the max-minus-min accuracy spread across the three runs, targeting the reported ~±5% GPT-3.5 variation, and compare spreads across configurations to confirm that the fixed seed, temperature 0, and simpler prompt yield the most consistent outputs. Write per-run accuracies and spread per configuration to a summary CSV. Note that higher consistency does not necessarily imply higher accuracy — report both.

## Related tools

- **Python 3.11** (Used for API interactions and pandas-based scoring of run outputs against the ACG-SA answer key) — https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **GPT-3.5** (Model whose repeated runs are scored; also used to extract the selected option from each textual response in the semi-automated pipeline)
- **OpenAI Python package** (Communication with the OpenAI (GPT-3.5) models when re-running the API runs to be scored)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes the GPT-3.5 API interactions so prompt variants (e.g., 'Just QA', QA followed by 'are you sure?') are applied consistently across runs)

## Examples

```
import pandas as pd; per_run = pd.read_csv('run_accuracies.csv'); summary = per_run.groupby('config')['accuracy'].agg(['max','min']); summary['spread'] = summary['max']-summary['min']; summary.to_csv('consistency_summary.csv')
```

## Evaluation signals

- Reproduced GPT-3.5 run-to-run accuracy variation under the fixed-seed, temperature-zero, simple-prompt ('Just QA') configuration falls within approximately ±5 percentage points, matching the reported finding
- Per-question labels are binary correct/incorrect and every question in the 2022 ACG-SA set is scored exactly once per run (no missing or double-counted answers)
- Extracted selected options are validated by human review wherever the GPT-3.5 extraction pipeline's outputs disagree
- consistency_summary.csv contains one row per configuration with per-run accuracies and the max-minus-min spread, and configurations with fixed seed / temperature 0 / simpler prompts show smaller spreads than multi-turn variants
- Higher consistency does not automatically imply higher accuracy — accuracy and spread are reported as separate columns and checked for decoupling as in the paper

## Limitations

- Limited number of experimental runs; no average and SD reported to assess model consistency
- Information loss from evaluating responses as simply correct or incorrect; even a correct selected option may be accompanied by hallucinations
- Prompt/settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs; generalizability of this configuration to other LLMs is uncertain

## Evidence
