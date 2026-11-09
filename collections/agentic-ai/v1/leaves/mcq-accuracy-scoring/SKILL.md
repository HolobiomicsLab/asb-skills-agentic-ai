---
name: mcq-accuracy-scoring
description: Use when you have raw LLM/VLM free-text responses to multiple-choice
  questions (e.g., ACG-SA 2022 text-only or image-inclusive items) plus an answer
  key, and need per-model accuracy values.
when_to_use_negative:
- When the goal is to assess justification quality, hallucination, or clinical reasoning
  depth — the article notes MCQ scoring 'may exhibit hallucinations in its response'
  even when the correct option is selected
- When responses cannot be mapped to a single selectable option (e.g., models with
  incoherent output like Meditron-7b-Q16 and HippoMistral-7b-Q16, which were excluded
  for inability to provide coherent responses)
- When a non-MCQ validation task is needed to reflect real-world LLM usage — the article
  cites the lack of a more suitable validation dataset as a limitation
edam_operation: http://edamontology.org/operation_0306
tools:
- name: OpenAI Python package
  role: Runs the GPT-3.5 API extraction step that pulls the selected option from each
    LLM textual response before scoring
- name: Streamlit (1.33)
  role: Human evaluation application for reviewing the semi-automated option extraction
    prior to binary scoring
- name: Python 3.11
  role: Scripting environment (with pandas) for merging responses with the ACG-SA
    answer key and computing per-model/per-stratum accuracy
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:4528a5e9bddcbdf70f1082603fd59f61d74292c1ce577c6608bf0a573cb1e6c8
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
  - LangChain (0.1.17 and 0.2.4)
  - Llama 3.2-11b (8-bit)
  - Llama 3.3-70b
  - OpenAI Python package
  - Phi3-14b (6-bit)
  - Python
  - Python 3.11
  - Qwen 2.5-72b
  - Streamlit (1.33)
  - o1-preview
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# mcq-accuracy-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Semi-automated scoring pipeline for medical MCQ benchmarking: extract the selected multiple-choice option from each LLM/VLM textual response with a GPT-3.5 extraction step via the OpenAI Python package, apply human review in a Streamlit app, and score responses as binary correct/incorrect against the ACG-SA answer key to compute accuracy.

## When to use

Use when you have raw LLM/VLM free-text responses to multiple-choice questions (e.g., ACG-SA 2022 text-only or image-inclusive items) plus an answer key, and need per-model accuracy values.

## When NOT to use

- When the goal is to assess justification quality, hallucination, or clinical reasoning depth — the article notes MCQ scoring 'may exhibit hallucinations in its response' even when the correct option is selected
- When responses cannot be mapped to a single selectable option (e.g., models with incoherent output like Meditron-7b-Q16 and HippoMistral-7b-Q16, which were excluded for inability to provide coherent responses)
- When a non-MCQ validation task is needed to reflect real-world LLM usage — the article cites the lack of a more suitable validation dataset as a limitation

## Inputs

- LLM/VLM raw free-text responses to ACG-SA MCQs (2022 edition; 162 text-only and 138 image-inclusive questions)
- ACG-SA answer key for the scored question set
- Question metadata (e.g., human test-taker difficulty quartile labels Q1–Q4, image-inclusive flags)

## Outputs

- Per-model accuracy values (proportion of correctly selected options) per condition, e.g., accuracy CSVs like parameter_optimization_results.csv or quantization_comparison.csv
- Stratified accuracy tables (e.g., per difficulty quartile: GPT-4o 88% Q4 / 74% Q2-Q3 / 58.7% Q1)
- Human-reviewed extracted option labels per response

## How to apply

First run the GPT-3.5 API extraction step to pull the selected option (e.g., A–E) out of each model's free-text response, then perform human review of the semi-automated extraction in the Streamlit (1.33) evaluation app to catch extraction errors. Score each reviewed response as correct or incorrect against the ACG-SA answer key (e.g., the 2022 ACG-SA key over the 162 text-only and 138 image-inclusive questions), treating scoring as binary rather than partial-credit. Compute accuracy as the proportion of correctly selected options per model per condition (output format, prompt variant, image scenario, difficulty quartile), merging question metadata such as human test-taker difficulty quartiles (Q1–Q4) for stratified analyses. The binary protocol deliberately discards justification text; the article flags this information loss as a known trade-off of the approach.

## Related tools

- **OpenAI Python package** (Runs the GPT-3.5 API extraction step that pulls the selected option from each LLM textual response before scoring)
- **Streamlit (1.33)** (Human evaluation application for reviewing the semi-automated option extraction prior to binary scoring)
- **Python 3.11** (Scripting environment (with pandas) for merging responses with the ACG-SA answer key and computing per-model/per-stratum accuracy) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology

## Examples

```
df = pd.merge(responses, key, on='question_id'); accuracy = (df['extracted_option'] == df['correct_option']).mean()
```

## Evaluation signals

- Every response in the scored table has exactly one extracted option label and a binary correct/incorrect value after human review
- Recomputed accuracy matches published per-model values (e.g., o1-preview 82.0%, Claude3.5-Sonnet 74.0%, Llama3.3-70b 65.7%, Qwen-2.5-72b 61.0%) within rounding tolerance
- Stratified reproduction recovers GPT-4o quartile accuracies of 88% (Q4), 74% (Q2-Q3), and 58.7% (Q1)
- Parameter-optimization scoring reproduces the combined-prompt delta of +5.00% (95% CI 1.11–7.78%) and answer-and-justify delta of +4.44%
- Questions known to be inaccessible behind the ACG-SA paywall are treated as unseen by models, per the paper's contamination rationale

## Limitations

- Binary correct/incorrect evaluation causes information loss; an LLM may select the correct option yet hallucinate in its justification
- Option extraction from free text is semi-automated and requires human review; extraction errors would propagate into accuracy estimates
- Limited number of experimental runs means no average/SD is reported to assess model consistency
- Model knowledge cutoffs changed during experimentation without documented notification, potentially affecting scored accuracy

## Evidence
