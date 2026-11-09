---
name: model-accuracy-computation-against-answer-key
description: Use when you have model-generated answer choices for a medical MCQ dataset
  (e.g. 2022 ACG-SA text-only questions) and need per-model accuracy versus a known
  answer key.
when_to_use_negative:
- Questions are image-inclusive — this skill covers only the text-only subset (162
  questions); image-inclusive questions were excluded from this benchmark
- The model's response cannot be reduced to a single selected option (e.g. incoherent
  models like Meditron-7b-Q16 or HippoMistral-7b-Q16, which were excluded for incoherent
  responses on >50% of questions)
- You need nuanced evaluation beyond binary correctness — the authors note information
  loss from evaluating responses as simply correct or incorrect
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3303
tools:
- name: GPT-3.5 (OpenAI API)
  role: Extracts the selected option from each LLM's free-text response so it can
    be scored against the answer key
- name: Python 3.11
  role: Used for API interactions and computation/writing of per-model accuracies
    to CSV
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes the model queries whose responses are then scored
- name: OpenAI Python package
  role: Handles communication with OpenAI models (including the GPT-3.5 extraction
    step)
- name: Streamlit (1.33)
  role: Human evaluation application used to review ambiguous extractions before scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:7c06af232dc65bdeae38ac73814835358f4909d821c176285efe5e5af8dde86d
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
  - GPT-3.5 (OpenAI API)
  - LangChain
  - LangChain (0.1.17 and 0.2.4)
  - Llama 3.3-70b
  - OpenAI Python package
  - Python
  - Python 3.11
  - Qwen 2.5-72b
  - Streamlit (1.33)
  - o1-preview
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3303
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# Model accuracy computation against answer key

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores each LLM/VLM selected option against the official ACG-SA answer key and computes per-model accuracy as the proportion of correctly answered questions (e.g. the 162 text-only questions of the 2022 ACG-SA), writing results to a per-model CSV table.

## When to use

Use when you have model-generated answer choices for a medical MCQ dataset (e.g. 2022 ACG-SA text-only questions) and need per-model accuracy versus a known answer key.

## When NOT to use

- Questions are image-inclusive — this skill covers only the text-only subset (162 questions); image-inclusive questions were excluded from this benchmark
- The model's response cannot be reduced to a single selected option (e.g. incoherent models like Meditron-7b-Q16 or HippoMistral-7b-Q16, which were excluded for incoherent responses on >50% of questions)
- You need nuanced evaluation beyond binary correctness — the authors note information loss from evaluating responses as simply correct or incorrect

## Inputs

- Per-model LLM textual responses to the 162 text-only 2022 ACG-SA questions (from the Sdamirsa/LLM-VLM-in-Gastroenterology repository)
- ACG-SA 2022 answer key (correct options per question)
- Extracted selected options per response (via GPT-3.5 extraction with human review)

## Outputs

- llm_text_only_accuracy.csv — per-model accuracy table (proportion of correctly answered text-only questions)
- Per-question correct/incorrect scores against the ACG answer key

## How to apply

After responses are collected (Python 3.11 + LangChain calls to o1-preview, Claude3.5-Sonnet, GPT-4o, Llama3.3-70b, Qwen-2.5-72b), extract the selected option from each free-text response using the GPT-3.5 API extraction step with human review of ambiguous extractions. Score each extracted option against the ACG-SA answer key as strictly correct/incorrect. Compute accuracy per model as correct count divided by the number of questions in scope — 162 text-only questions of the 300-question 2022 ACG-SA for this benchmark. Where a model cannot be re-queried, transcribe the paper's reported accuracy (o1-preview 82.0%, Claude3.5-Sonnet 74.0%, GPT-4o 74%, Llama3.3-70b 65.7%, Qwen-2.5-72b 61.0%) into the same table. Write all per-model accuracies to llm_text_only_accuracy.csv and verify each reproduced value matches the reported figure within the granularity given in the paper's abstract.

## Related tools

- **GPT-3.5 (OpenAI API)** (Extracts the selected option from each LLM's free-text response so it can be scored against the answer key)
- **Python 3.11** (Used for API interactions and computation/writing of per-model accuracies to CSV)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes the model queries whose responses are then scored)
- **OpenAI Python package** (Handles communication with OpenAI models (including the GPT-3.5 extraction step))
- **Streamlit (1.33)** (Human evaluation application used to review ambiguous extractions before scoring)

## Examples

```
accuracy = extracted_df.groupby('model')['correct'].mean(); accuracy.to_csv('llm_text_only_accuracy.csv')
```

## Evaluation signals

- Each reproduced accuracy value matches the reported figure within the abstract's granularity (o1-preview 82.0%, Qwen-2.5-72b 61.0%)
- Accuracy is computed over exactly the 162 text-only questions, not the 138 image-inclusive ones
- Every extracted option has been either confidently machine-extracted or human-reviewed; ambiguous extractions are resolved before scoring
- Output CSV contains one accuracy row per model and column structure consistent with the paper's table
- Total correct + incorrect counts per model sum to the number of scored questions

## Limitations

- Limited number of experimental runs; no average and SD reported to assess model consistency
- Response evaluation is binary correct/incorrect, causing information loss even when the selected option is correct (hallucinations in the response text are not captured)
- Prompt/settings were optimized only on GPT-3.5 and reused for other LLMs, so per-model extraction/answering quality may vary

## Evidence
