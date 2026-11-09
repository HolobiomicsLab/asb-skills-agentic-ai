---
name: mcq-answer-consistency-analysis
description: Use when you have 2+ separate LLM API runs over the same MCQ set (e.g.,
  2022 ACG-SA) and need to quantify output stability across runs or test how seed,
  temperature, and prompt design affect it.
when_to_use_negative:
- You have only a single LLM run — consistency cannot be quantified without multiple
  runs
- You need fine-grained response quality (reasoning errors, hallucinations) rather
  than binary correct/incorrect scoring, since the article notes information loss
  from correct/incorrect-only evaluation
- Your goal is comparing different models' accuracy (benchmarking) rather than one
  model's stability across repeated runs
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3379
- http://edamontology.org/topic_3077
tools:
- name: Python 3.11
  role: Language used for API interactions, scoring, and computing run-to-run accuracy
    spreads
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes the GPT-3.5 API interactions across the three consistency runs
- name: OpenAI Python package
  role: Communication with the GPT-3.5 model for the three separate runs and for option
    extraction
- name: GPT-3.5
  role: Semi-automated extraction of the selected option from each LLM textual response,
    with human review of disagreements
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:f2199e856052d79818bae86da500fc8f7513789ab41247bc08730a254d5354b0
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3379
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mcq-answer-consistency-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies run-to-run consistency of LLM answers on multiple-choice medical questions by scoring three separate API runs under controlled settings (fixed random seed, temperature 0, simple 'just QA' prompt) and computing max-minus-min accuracy spread. Used because LLM outputs vary across runs, and consistency must be distinguished from accuracy.

## When to use

Use when you have 2+ separate LLM API runs over the same MCQ set (e.g., 2022 ACG-SA) and need to quantify output stability across runs or test how seed, temperature, and prompt design affect it.

## When NOT to use

- You have only a single LLM run — consistency cannot be quantified without multiple runs
- You need fine-grained response quality (reasoning errors, hallucinations) rather than binary correct/incorrect scoring, since the article notes information loss from correct/incorrect-only evaluation
- Your goal is comparing different models' accuracy (benchmarking) rather than one model's stability across repeated runs

## Inputs

- Three separate GPT-3.5 API run outputs (free-text responses) over the 2022 ACG-SA questions
- 2022 ACG Self-Assessment (ACG-SA) answer key (300 questions; 162 text-only and 138 image-inclusive evaluated)
- Configuration metadata per run (prompt variant, temperature, random seed)
- Python 3.11 environment with pandas, LangChain 0.1.17/0.2.4, and the OpenAI Python package

## Outputs

- Per-question correct/incorrect labels per run per configuration
- Per-run overall accuracy values per configuration
- Run-to-run consistency metric (max-minus-min accuracy spread, expected ~±5 percentage points for GPT-3.5)
- consistency_summary.csv with per-run accuracies and run-to-run variation per configuration

## How to apply

Load each of the three GPT-3.5 API run outputs and the 2022 ACG-SA answer key (from the LLM-VLM-in-Gastroenterology repository) using Python 3.11 and pandas. Extract the selected option from each free-text response with the semi-automated pipeline (GPT-3.5-based option extraction followed by human review of disagreements). Score each question correct/incorrect against the key and compute overall accuracy per run under each configuration (simple 'just QA' vs. multi-turn variants such as QA followed by 'are you sure?', temperature 0, fixed random seed). Quantify run-to-run consistency as the max-minus-min accuracy spread across the three runs; the paper reports ~±5% variation for GPT-3.5. Compare spreads across configurations to confirm that fixed seed, temperature 0, and the simpler prompt yield the most consistent outputs, then write per-run accuracies and spreads per configuration to consistency_summary.csv. Note that higher consistency did not necessarily improve accuracy, so report both metrics.

## Related tools

- **Python 3.11** (Language used for API interactions, scoring, and computing run-to-run accuracy spreads)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes the GPT-3.5 API interactions across the three consistency runs)
- **OpenAI Python package** (Communication with the GPT-3.5 model for the three separate runs and for option extraction)
- **GPT-3.5** (Semi-automated extraction of the selected option from each LLM textual response, with human review of disagreements)

## Examples

```
import pandas as pd; runs = [pd.read_csv(f'run_{i}.csv') for i in (1,2,3)]; acc = [r['correct'].mean() for r in runs]; spread = max(acc) - min(acc); pd.DataFrame({'accuracy': acc, 'spread': spread}).to_csv('consistency_summary.csv')
```

## Evaluation signals

- Reproduced GPT-3.5 run-to-run accuracy variation falls within approximately ±5 percentage points under the fixed-seed, temperature-zero, simple-prompt configuration
- consistency_summary.csv exists and contains per-run accuracies plus a max-minus-min spread per configuration
- Option-extraction disagreements between GPT-3.5 extraction and raw responses were resolved by human review before scoring
- The fixed-seed / temperature-0 / simple-prompt configuration shows the smallest spread relative to multi-turn variants such as QA followed by 'are you sure?'
- Per-run accuracy values are consistent with a denominator equal to the evaluated question count (e.g., the 2022 ACG-SA set)

## Limitations

- Only a limited number of experimental runs were performed; no average and SD were reported to fully assess model consistency
- Increased consistency did not necessarily improve accuracy, so consistency gains must not be read as accuracy gains
- The GPT-3.5-derived configuration was reused for other LLMs without per-model optimization, and its generalizability to other LLMs is uncertain

## Evidence
