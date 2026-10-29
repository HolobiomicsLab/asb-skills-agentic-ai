---
name: mcq-option-extraction-from-llm-responses
description: Use when raw LLM/VLM outputs from MCQ benchmarks (e.g., ACG-SA 2021-2023)
  are free-text and must be converted into a discrete selected option before scoring
  correctness against an answer key.
when_to_use_negative:
- Responses already contain a cleanly structured option letter, so no extraction step
  is needed
- Model responses are incoherent on >50% of questions (as with Meditron-7b-Q16 and
  HippoMistral-7b-Q16) — exclude the model rather than force extraction
- The evaluation goal is reasoning quality or hallucination analysis, which binary
  correct/incorrect extraction discards
tools:
- name: GPT-3.5 API
  role: Semi-automated extraction of the selected MCQ option letter from each free-text
    LLM response
- name: Streamlit (1.33)
  role: Human-evaluation application for reviewing ambiguous option extractions
- name: Python 3.11
  role: Scripting language for API interactions and pandas-based scoring of extracted
    options against the answer key
- name: OpenAI Python package
  role: Client used to communicate with the OpenAI models performing the option extraction
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:8c947e5c8fb3d55e6ce967353b8d26387fc1758418cfeaecd772177b9c80e1f0
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
  - GPT-3.5 API
  - LangChain
  - Llama 3.3-70b
  - OpenAI Python package
  - Python
  - Python 3.11
  - Qwen 2.5-72b
  - Streamlit (1.33)
  - o1-preview
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# MCQ option extraction from LLM responses

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A semi-automated pipeline that extracts the selected multiple-choice option letter from free-text LLM responses using a GPT-3.5 API call, with human review of ambiguous extractions via a Streamlit app, enabling accuracy scoring against the ACG-SA answer key.

## When to use

Use when raw LLM/VLM outputs from MCQ benchmarks (e.g., ACG-SA 2021-2023) are free-text and must be converted into a discrete selected option before scoring correctness against an answer key.

## When NOT to use

- Responses already contain a cleanly structured option letter, so no extraction step is needed
- Model responses are incoherent on >50% of questions (as with Meditron-7b-Q16 and HippoMistral-7b-Q16) — exclude the model rather than force extraction
- The evaluation goal is reasoning quality or hallucination analysis, which binary correct/incorrect extraction discards

## Inputs

- Raw free-text LLM responses to ACG-SA MCQs (stored model outputs from LangChain/OpenAI API runs, loaded with Python 3.11 + pandas)
- Official ACG-SA answer key mapping question IDs to correct option letters
- Model/edition metadata (e.g., GPT-4-0613 vs GPT-3.5-0125 on 2021/2022/2023 editions)

## Outputs

- Per-question extracted option letter with correctness flag against the answer key
- Aggregated per-model, per-edition accuracy percentages (e.g., fig7a_gpt_accuracy.csv with 6 rows: model, edition, accuracy)

## How to apply

For each free-text LLM response, call the GPT-3.5 API (via the OpenAI Python package with Python 3.11) with a prompt asking which option letter was selected, mirroring the paper's semi-automated evaluation step. Route ambiguous extractions to a human-review application built with Streamlit (1.33) rather than auto-scoring them. Match each extracted letter against the official ACG-SA answer key to mark per-question correctness, then aggregate into per-model accuracy percentages. This extraction step is applied identically across models (e.g., GPT-4-0613, GPT-3.5-0125, o1-preview, Llama3.3-70b) so accuracies are comparable; the paper reused GPT-3.5-derived settings from the parameter-optimization experiment across all LLMs.

## Related tools

- **GPT-3.5 API** (Semi-automated extraction of the selected MCQ option letter from each free-text LLM response)
- **Streamlit (1.33)** (Human-evaluation application for reviewing ambiguous option extractions)
- **Python 3.11** (Scripting language for API interactions and pandas-based scoring of extracted options against the answer key)
- **OpenAI Python package** (Client used to communicate with the OpenAI models performing the option extraction)

## Examples

```
from openai import OpenAI; client = OpenAI(); r = client.chat.completions.create(model='gpt-3.5-turbo', messages=[{'role':'user','content': llm_response + '\nWhich option letter was selected? Reply with the letter only.'}])
```

## Evaluation signals

- Each ambiguous extraction is routed to human review (Streamlit app) rather than silently auto-scored
- Aggregated accuracies reproduce reported values within 0.1 percentage points (e.g., GPT-4-0613: 71.59%, 65.38%, 70.35% and GPT-3.5-0125: 44.57%, 46.30%, 53.49% for 2021/2022/2023)
- Output CSV contains the expected number of rows with model and edition fields present (e.g., 6 rows for fig7a_gpt_accuracy.csv)
- The same extraction procedure is applied uniformly across all models so per-model accuracies are directly comparable

## Limitations

- The GPT-3.5-derived extraction configuration was optimized only for OpenAI models and reused for other LLMs; its generalizability is uncertain with only limited validation
- Evaluating responses as simply correct or incorrect loses information about reasoning quality and possible hallucinations even when the correct option is selected
- Ambiguous extractions require human review, so the pipeline is semi-automated rather than fully reproducible end-to-end

## Evidence
