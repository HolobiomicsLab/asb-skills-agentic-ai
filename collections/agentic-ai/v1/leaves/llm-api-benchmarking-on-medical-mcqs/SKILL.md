---
name: llm-api-benchmarking-on-medical-mcqs
description: Use when the goal is to reproduce or extend a reported MCQ accuracy benchmark
  for specific LLMs (e.g., o1-preview 82.0%, Claude3.5-Sonnet 74.0%) on a medical
  question set with API-accessible models.
when_to_use_negative:
- The benchmark targets image-inclusive questions requiring vision-language models
  — exclude image-containing questions when applying this text-only LLM skill
- Models that cannot provide coherent responses on >50% of questions (as with Meditron-7b-Q16
  and HippoMistral-7b-Q16 in the paper) should be excluded rather than forced through
  the pipeline
- You need real-world dynamic interaction, documentation/extraction, or hallucination
  assessment — MCQ accuracy is only a proxy and loses information when responses are
  judged simply correct or incorrect
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3376
tools:
- name: Python 3.11
  role: Runtime for API interactions and benchmark scripts
- name: LangChain
  role: Standardizes interactions (versions 0.1.17 and 0.2.4) with each LLM endpoint
- name: OpenAI Python package
  role: Communication with the OpenAI models (o1-preview, GPT-4o) and the GPT-3.5
    answer-extraction step
- name: Streamlit (1.33)
  role: Human evaluation application for reviewing ambiguous option extractions
- name: o1-preview
  role: Proprietary LLM under benchmark; top accuracy (82.0%) on text-only ACG-SA
    questions
- name: Llama 3.3-70b
  role: Open-source LLM under benchmark (65.7% text-only accuracy)
  repo: https://huggingface.co/Meta-llama/Llama-3.3-70B-Instruct
- name: Qwen 2.5-72b
  role: Open-source LLM under benchmark (61.0% text-only accuracy)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:9eca16503d0a73fe62a9471e09df04da4bad2ec7940caaa165027c34bd09fe96
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
  - LangChain
  - Llama 3.3-70b
  - OpenAI Python package
  - Python
  - Python 3.11
  - Qwen 2.5-72b
  - Streamlit (1.33)
  - o1-preview
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# llm-api-benchmarking-on-medical-mcqs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A reproducible protocol for benchmarking proprietary and open-source LLMs on medical multiple-choice questions (e.g., 162 text-only questions from the 2022 ACG-SA gastroenterology exam), scoring accuracy against an answer key via a semi-automated GPT-3.5 extraction pipeline.

## When to use

Use when the goal is to reproduce or extend a reported MCQ accuracy benchmark for specific LLMs (e.g., o1-preview 82.0%, Claude3.5-Sonnet 74.0%) on a medical question set with API-accessible models.

## When NOT to use

- The benchmark targets image-inclusive questions requiring vision-language models — exclude image-containing questions when applying this text-only LLM skill
- Models that cannot provide coherent responses on >50% of questions (as with Meditron-7b-Q16 and HippoMistral-7b-Q16 in the paper) should be excluded rather than forced through the pipeline
- You need real-world dynamic interaction, documentation/extraction, or hallucination assessment — MCQ accuracy is only a proxy and loses information when responses are judged simply correct or incorrect

## Inputs

- Medical MCQ question set with answer key (e.g., 2022 ACG-SA: 300 questions, 162 text-only and 138 image-inclusive) from the Sdamirsa/LLM-VLM-in-Gastroenterology repository
- API-accessible model endpoints (o1-preview, Claude 3.5 Sonnet, GPT-4o, Llama 3.3-70b, Qwen 2.5-72b) or transcribed reported accuracies
- GPT-3.5-optimized settings: prompt, temperature, max-token

## Outputs

- Per-model accuracy table written to llm_text_only_accuracy.csv (e.g., o1-preview 82.0%, Claude3.5-Sonnet 74.0%, Llama3.3-70b 65.7%, Qwen-2.5-72b 61.0%)
- Per-question extracted selected-option labels and correct/incorrect scores

## How to apply

Load the question set (e.g., the 2022 ACG-SA with 300 questions; retain only the 162 text-only questions for a text benchmark, excluding image-inclusive ones) from the Sdamirsa/LLM-VLM-in-Gastroenterology repository. First run a parameter-optimization experiment on a subset with GPT-3.5 to select the best-performing prompt, temperature, and max-token settings, then reuse those settings for other LLMs via Python 3.11 and LangChain (0.1.17/0.2.4), communicating with OpenAI models through the OpenAI Python package. Extract the selected option from each free-text LLM response using the GPT-3.5 API, with human review of ambiguous extractions (e.g., via a Streamlit 1.33 evaluation app). Score each extracted answer against the ACG answer key and compute accuracy as the proportion of correctly answered questions per model; where a model cannot be queried, transcribe reported accuracies (o1-preview 82.0%, Claude3.5-Sonnet 74.0%, GPT-4o 74%, Llama3.3-70b 65.7%, Qwen-2.5-72b 61.0%) into the same table. Write per-model accuracies to a CSV (e.g., llm_text_only_accuracy.csv).

## Related tools

- **Python 3.11** (Runtime for API interactions and benchmark scripts)
- **LangChain** (Standardizes interactions (versions 0.1.17 and 0.2.4) with each LLM endpoint)
- **OpenAI Python package** (Communication with the OpenAI models (o1-preview, GPT-4o) and the GPT-3.5 answer-extraction step)
- **Streamlit (1.33)** (Human evaluation application for reviewing ambiguous option extractions)
- **o1-preview** (Proprietary LLM under benchmark; top accuracy (82.0%) on text-only ACG-SA questions)
- **Llama 3.3-70b** (Open-source LLM under benchmark (65.7% text-only accuracy)) — https://huggingface.co/Meta-llama/Llama-3.3-70B-Instruct
- **Qwen 2.5-72b** (Open-source LLM under benchmark (61.0% text-only accuracy))

## Examples

```
python run_benchmark.py --dataset 2022_ACG-SA --text-only --models o1-preview,Claude3.5-Sonnet,Llama3.3-70b,Qwen-2.5-72b --out llm_text_only_accuracy.csv
```

## Evaluation signals

- Reproduced accuracy for each model matches the reported figure within the granularity of the paper's abstract (o1-preview 82.0%, Claude3.5-Sonnet 74.0%, GPT-4o 74%, Llama3.3-70b 65.7%, Qwen-2.5-72b 61.0%)
- llm_text_only_accuracy.csv contains exactly one accuracy row per model, all computed over the same 162 text-only questions
- Every scored response has an extracted selected option traceable to the GPT-3.5 extraction step, with ambiguous extractions resolved by documented human review
- Excluded image-inclusive questions (138 of 300) are not scored; the denominator equals the text-only question count

## Limitations

- Prompt/settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs; per-model configuration may be more effective and generalizability is uncertain
- Limited number of experimental runs; no average and SD reported to assess model consistency
- Benchmark limited to gastroenterology questions and may not generalize to all of medicine; MCQ accuracy does not capture hallucinations or real-world dynamic interactions
- Model knowledge cutoffs changed during experimentation without documented notification
- ACG-SA access is restricted by a paywall, which is presumed to exclude the questions from LLM training data

## Evidence
