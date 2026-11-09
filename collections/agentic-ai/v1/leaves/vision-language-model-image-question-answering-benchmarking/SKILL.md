---
name: vision-language-model-image-question-answering-benchmarking
description: Use when benchmarking VLMs on multiple-choice questions that contain
  clinical images, to quantify whether image content, captions, or text alone drive
  accuracy.
when_to_use_negative:
- Questions are text-only (162 non-image 2022 ACG-SA questions) — use the LLM benchmarking
  workflow instead; image-inclusive questions were 'Excluded' in the LLM arm
- Evaluating free-text reasoning, hallucination, or documentation quality — binary
  correct/incorrect MCQ scoring loses this information
- A single model's general clinical reasoning is being tested rather than a controlled
  comparison of image-input modalities
edam_operation: null
edam_topics: []
tools:
- name: Claude 3.5 Sonnet
  role: Proprietary VLM evaluated across the four image-input scenarios; showed a
    significant +20.2% gain (p=0.001) with direct image input
- name: Llama 3.2-11b (8-bit)
  role: Open-source VLM (Meta-llama/Llama-3.2-11B-Vision-Instruct) evaluated under
    all four scenarios; +8.7% (p=0.047) with direct image input
  repo: https://huggingface.co/Meta-llama/Llama-3.2-11B-Vision-Instruct
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes VLM API interactions for the benchmark QA prompt
- name: OpenAI Python package
  role: Runs the GPT-3.5 extraction of the selected MCQ option from VLM textual responses
- name: Streamlit (1.33)
  role: Human evaluation app for reviewing semi-automated option extraction
  repo: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:2554b296cae5385796a34b9a9e7f88329b45f785a24e430921fe9fd9b21093e5
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

# vision-language-model-image-question-answering-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking protocol for evaluating vision-language models (VLMs) on image-based gastroenterology MCQs across four input scenarios: no-image baseline, VLM-generated captions, direct image input, and human-generated captions, revealing that human captions improve accuracy (~10%) while direct image input rarely does.

## When to use

Use when benchmarking VLMs on multiple-choice questions that contain clinical images, to quantify whether image content, captions, or text alone drive accuracy.

## When NOT to use

- Questions are text-only (162 non-image 2022 ACG-SA questions) — use the LLM benchmarking workflow instead; image-inclusive questions were 'Excluded' in the LLM arm
- Evaluating free-text reasoning, hallucination, or documentation quality — binary correct/incorrect MCQ scoring loses this information
- A single model's general clinical reasoning is being tested rather than a controlled comparison of image-input modalities

## Inputs

- 138 image-inclusive MCQs from the 2022 ACG Self-Assessment (ACG-SA)
- Answer images associated with each image-based question
- Human-generated image captions
- VLM-generated (LLM-generated) image captions
- Answer key for correct/incorrect scoring
- Benchmark QA prompt configuration (temperature, max-token, prompt variant)

## Outputs

- Per-VLM, per-scenario accuracy table over the 138 image-based questions
- Paired statistical test p-values comparing each image scenario to the no-image baseline
- Extracted selected options per model response with human-review status

## How to apply

Load the 138 image-inclusive questions, their answer images, and human-generated captions for the 2022 ACG Self-Assessment (ACG-SA) from github.com/Sdamirsa/LLM-VLM-in-Gastroenterology. Query each VLM (e.g., Claude3-Sonnet and the 8-bit Llama 3.2-11b) under the no-image baseline using the benchmark QA prompt via Python 3.11 with LangChain (0.1.17 and 0.2.4) to standardize API interactions, then run the three image-informed conditions: LLM-generated captions, human-generated captions, and direct image input. Extract the selected MCQ option from each textual response with the semi-automated GPT-3.5 extraction pipeline (via the OpenAI Python package) followed by human review in a Streamlit (1.33) evaluation app. Score each response correct/incorrect against the answer key, compute accuracy per VLM per scenario, and perform paired statistical tests comparing each image scenario to the no-image baseline; accept the reproduction when the table shows ~10% improvement with human captions, no change with original images, decline with LLM-generated captions, and significant direct-image gains (Claude3-Sonnet +20.2%, p=0.001; Llama3.2-11b +8.7%, p=0.047).

## Related tools

- **Claude 3.5 Sonnet** (Proprietary VLM evaluated across the four image-input scenarios; showed a significant +20.2% gain (p=0.001) with direct image input)
- **Llama 3.2-11b (8-bit)** (Open-source VLM (Meta-llama/Llama-3.2-11B-Vision-Instruct) evaluated under all four scenarios; +8.7% (p=0.047) with direct image input) — https://huggingface.co/Meta-llama/Llama-3.2-11B-Vision-Instruct
- **LangChain (0.1.17 and 0.2.4)** (Standardizes VLM API interactions for the benchmark QA prompt)
- **OpenAI Python package** (Runs the GPT-3.5 extraction of the selected MCQ option from VLM textual responses)
- **Streamlit (1.33)** (Human evaluation app for reviewing semi-automated option extraction) — https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology

## Examples

```
pip install "langchain==0.2.4" "streamlit==1.33" openai  # then query each VLM under baseline/caption/direct-image scenarios via LangChain and score 138 image MCQs
```

## Evaluation signals

- Per-scenario accuracy table reproduces the reported pattern: ~10% average improvement with human captions, unchanged accuracy with original images, decline with LLM-generated captions
- Direct-image gains are statistically significant only for Claude3-Sonnet (+20.2%, p=0.001) and Llama3.2-11b (+8.7%, p=0.047); other VLMs show no significant improvement over the no-image baseline
- Every VLM textual response yields a successfully extracted selected MCQ option (GPT-3.5 extraction plus human review in the Streamlit app) that can be scored against the answer key
- Accuracy is computed over exactly the 138 image-inclusive 2022 ACG-SA questions, matching the paper's question count
- Each image scenario is compared to its own no-image baseline via a paired statistical test rather than aggregate accuracy

## Limitations

- Prompt/settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs; per-model prompt configuration is suggested as more effective, and generalizability of the GPT-3.5-derived configuration to other LLMs is uncertain
- Limited number of experimental runs; no average and SD reported to assess model consistency
- Evaluating responses as simply correct or incorrect causes information loss; even correct option selection may accompany hallucinations, and MCQ evaluation does not capture real-world dynamic interactions
- Model knowledge cutoffs changed during experimentation without documented notification, and the benchmark is limited to gastroenterology questions

## Evidence
