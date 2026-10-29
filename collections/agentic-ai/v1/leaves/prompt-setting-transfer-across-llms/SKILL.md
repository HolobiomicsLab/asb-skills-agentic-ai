---
name: prompt-setting-transfer-across-llms
description: Use when benchmarking multiple API-accessible or locally deployed LLMs
  on the same MCQ dataset and you need a single, standardized interaction configuration
  (prompt, temperature, max-token) applied uniformly to every model before scoring
  accuracy.
when_to_use_negative:
- When per-model prompt engineering is feasible and desired — the article's discussion
  notes per-model configuration may be more effective than reusing GPT-3.5-derived
  settings
- When the task requires chat-style dynamic interaction or free-text clinical output
  rather than single-shot MCQ option selection
- When the dataset is not MCQ-based with a verifiable answer key (e.g. open-ended
  clinical documentation tasks)
tools:
- name: GPT-3.5
  role: Reference model used to optimize prompt, temperature, and max-token settings
    before transfer to other LLMs; also used for the option-extraction step
- name: LangChain
  role: Standardizes API interactions so the transferred settings are applied uniformly
    across models
- name: OpenAI Python package
  role: Communication channel with OpenAI models under the transferred configuration
- name: Python 3.11
  role: Runtime for the query, extraction, and scoring pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:62ed15051f3727dad309fbe1a6ea13fb85c8c6ac9217dbab997eb1da6d6bd9b3
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
  - LangChain
  - Llama 3.3-70b
  - OpenAI Python package
  - Python
  - Python 3.11
  - Qwen 2.5-72b
  - o1-preview
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Prompt-setting transfer across LLMs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill in which a single set of model-interaction settings (prompt format, temperature, max-token) is optimized on one LLM (GPT-3.5 over a subset of the ACG-SA question set) and then reused unchanged when querying other LLMs, enabling comparable accuracy scoring across proprietary and open-source models.

## When to use

Use when benchmarking multiple API-accessible or locally deployed LLMs on the same MCQ dataset and you need a single, standardized interaction configuration (prompt, temperature, max-token) applied uniformly to every model before scoring accuracy.

## When NOT to use

- When per-model prompt engineering is feasible and desired — the article's discussion notes per-model configuration may be more effective than reusing GPT-3.5-derived settings
- When the task requires chat-style dynamic interaction or free-text clinical output rather than single-shot MCQ option selection
- When the dataset is not MCQ-based with a verifiable answer key (e.g. open-ended clinical documentation tasks)

## Inputs

- MCQ question set with answer key (e.g. 2022 ACG-SA: 300 questions, 162 text-only and 138 image-inclusive)
- Candidate prompt variants (Just QA; QA followed by 'are you sure?'; open-ended, MCQ without justification, MCQ with justification)
- API-accessible model list (o1-preview, Claude3.5-Sonnet, GPT-4o, Llama3.3-70b, Qwen-2.5-72b) and locally deployed quantized models
- Held-out subset of questions reserved for the GPT-3.5 parameter-optimization run

## Outputs

- Optimized settings tuple (prompt/function, temperature, max-token) fixed for all models
- Per-question model responses with extracted selected option (semi-automated GPT-3.5 extraction plus human review)
- Per-model accuracy table, e.g. llm_text_only_accuracy.csv

## How to apply

First run Experiment 0: use a subset of the dataset (here ACG-SA questions) and GPT-3.5 to determine the best performing settings — function, prompt variant (e.g. Just QA, QA followed by 'are you sure?', open-ended vs MCQ with/without justification), temperature, and max-token. Then reuse those optimized settings verbatim for all other models (o1-preview, Claude3.5-Sonnet, GPT-4o, Llama3.3-70b, Qwen-2.5-72b) via Python 3.11 and LangChain (0.1.17/0.2.4), communicating with OpenAI models through the OpenAI Python package. Extract the selected option from each textual response using the GPT-3.5 API extraction step with human review of ambiguous cases, exclude models giving incoherent responses on >50% of questions, and score against the ACG answer key as proportion correct (e.g. o1-preview 82.0%, Claude3.5-Sonnet 74.0%, Llama3.3-70b 65.7%, Qwen-2.5-72b 61.0%). The rationale is comparability across models, but the authors explicitly note the transfer assumption is unvalidated: the configuration was tuned only on OpenAI models.

## Related tools

- **GPT-3.5** (Reference model used to optimize prompt, temperature, and max-token settings before transfer to other LLMs; also used for the option-extraction step)
- **LangChain** (Standardizes API interactions so the transferred settings are applied uniformly across models)
- **OpenAI Python package** (Communication channel with OpenAI models under the transferred configuration)
- **Python 3.11** (Runtime for the query, extraction, and scoring pipeline)

## Evaluation signals

- Reproduced per-model accuracies match reported values within the paper's granularity (o1-preview 82.0%, Claude3.5-Sonnet 74.0%, GPT-4o 74%, Llama3.3-70b 65.7%, Qwen-2.5-72b 61.0%)
- Every response yields exactly one extracted option (A–E); ambiguous extractions are routed to human review rather than auto-scored
- Models with incoherent responses on >50% of questions are excluded from analysis (as done for Meditron-7b-Q16 and HippoMistral-7b-Q16)
- The same prompt string, temperature, and max-token values appear in the configuration for every model — no per-model divergence
- Accuracy per model is computed only over the retained question subset (e.g. 162 text-only questions) with the denominator documented in the output CSV

## Limitations

- Settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs; per-model prompt configuration may be more effective
- Generalizability of a configuration developed for one LLM to another is uncertain; only limited validation was performed
- Limited number of experimental runs — no average and SD reported, so consistency of results under the transferred settings is unquantified
- Model knowledge cutoffs changed during experimentation without documented notification, which can confound cross-model comparisons
- Quantized/local models run in beta-stage libraries, adding deployment variance when the transferred settings are applied off-API

## Evidence
