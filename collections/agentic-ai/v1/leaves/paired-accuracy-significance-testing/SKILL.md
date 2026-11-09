---
name: paired-accuracy-significance-testing
description: Use when the same fixed question set (e.g., the 138 image-inclusive 2022
  ACG-SA MCQs) is answered by each model under two or more conditions and you must
  attach p-values to per-model accuracy gains.
when_to_use_negative:
- When conditions are answered by different question sets or different models — the
  comparisons here are paired on the same 138 image-based questions within each VLM
- When evaluating text-only performance on the 162 text-only ACG-SA questions, where
  no image-condition pairing exists
- When you need graded response quality or hallucination assessment rather than binary
  correct/incorrect scoring
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: Python 3.11
  role: Runtime for the paired statistical comparison of per-scenario correctness
    vectors
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes the API interactions that produce the per-condition responses
    being compared
- name: OpenAI Python package
  role: GPT-3.5 semi-automated extraction of the selected MCQ option from each textual
    response before scoring
- name: Streamlit (1.33)
  role: Human evaluation app for reviewing the extracted options prior to correctness
    scoring and testing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:ad80c6346e8aba864c5e8b428ccb84802730e86cf3f20e98ad732a971d3c936d
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
  - LangChain (0.1.17 and 0.2.4)
  - Llama 3.2-11b (8-bit)
  - OpenAI Python package
  - Python
  - Python 3.11
  - Streamlit (1.33)
  - o1-preview
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# paired-accuracy-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistically tests whether per-model accuracy changes across paired experimental conditions (no-image baseline vs. LLM captions, human captions, or direct image input) on the same 138 image-based 2022 ACG-SA questions, attaching p-values to reported accuracy deltas.

## When to use

Use when the same fixed question set (e.g., the 138 image-inclusive 2022 ACG-SA MCQs) is answered by each model under two or more conditions and you must attach p-values to per-model accuracy gains.

## When NOT to use

- When conditions are answered by different question sets or different models — the comparisons here are paired on the same 138 image-based questions within each VLM
- When evaluating text-only performance on the 162 text-only ACG-SA questions, where no image-condition pairing exists
- When you need graded response quality or hallucination assessment rather than binary correct/incorrect scoring

## Inputs

- Per-question binary correct/incorrect vectors for each VLM under each scenario (no-image baseline, LLM-generated captions, human-generated captions, direct image input) over the 138 image-based 2022 ACG-SA questions
- Answer key for the 138 image-inclusive questions from the 2022 ACG-SA
- Extracted multiple-choice options from each model response (GPT-3.5 extraction plus human review in the Streamlit 1.33 app)

## Outputs

- Accuracy per VLM per scenario over the 138 image-based questions
- Per-model paired test p-values comparing each image scenario to the no-image baseline
- Validation table showing the reported pattern: ~10% human-caption gain, unchanged with original images, decline with LLM captions, significant direct-image gains for Claude3-Sonnet (p=0.001) and Llama3.2-11b (p=0.047)

## How to apply

Score each VLM response as correct/incorrect against the ACG-SA answer key per condition (no-image baseline, VLM-generated captions, human-generated captions, direct image input), giving per-question binary vectors that are paired across scenarios for each model. Run paired statistical tests of each image-informed scenario against the no-image baseline per VLM — not pooled across models — because effects are model-specific. Report both the accuracy delta and its p-value together (e.g., direct-image gains of +20.2% for Claude3-Sonnet, p=0.001, and +8.7% for the 8-bit Llama 3.2-11b, p=0.047; ~10% average improvement with human captions). Accept the reproduction when the table reproduces the reported pattern: ~10% gain with human captions, no change with original images, decline with LLM-generated captions, and the two significant direct-image gains.

## Related tools

- **Python 3.11** (Runtime for the paired statistical comparison of per-scenario correctness vectors)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes the API interactions that produce the per-condition responses being compared)
- **OpenAI Python package** (GPT-3.5 semi-automated extraction of the selected MCQ option from each textual response before scoring)
- **Streamlit (1.33)** (Human evaluation app for reviewing the extracted options prior to correctness scoring and testing)

## Evaluation signals

- Per-scenario accuracy table reproduces the reported pattern: ~10% average gain with human captions, no change with original images, decline with LLM-generated captions
- Significant direct-image gains appear only for Claude3-Sonnet (+20.2%, p=0.001) and Llama3.2-11b (+8.7%, p=0.047)
- Each p-value is reported alongside its corresponding accuracy delta, per model, not pooled across models
- Every scored response traces back to a reviewed extracted option (GPT-3.5 extraction followed by human review in the Streamlit 1.33 app)
- Denominator per cell equals 138 image-based questions for every VLM and scenario

## Limitations

- Limited number of experimental runs; no average and SD reported to assess model consistency
- Information loss from evaluating responses as simply correct or incorrect
- Prompt/settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs, so per-model response differences feeding the paired tests may partly reflect prompt fit
- Benchmark limited to gastroenterology questions; gains may not generalize to other specialties

## Evidence
