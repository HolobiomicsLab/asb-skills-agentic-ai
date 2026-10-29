---
name: llm-response-option-extraction
description: Use when raw free-text LLM/VLM responses to multiple-choice questions
  must be mapped to a single scorable option label (A–E) before computing accuracy.
when_to_use_negative:
- The model output is already a single machine-readable option letter, so no extraction
  step is needed
- Responses are incoherent/uncodable for more than half the questions — the article's
  filter step instead excludes such models outright (e.g., Meditron-7b-Q16 and HippoMistral-7b-Q16)
- You need qualitative analysis of response content (justification quality, hallucination)
  rather than option identity — binary correct/incorrect extraction loses that information
tools:
- name: OpenAI Python package
  role: Calls the GPT-3.5 API to extract the selected option from each raw textual
    response
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes the API interaction and structured-output extraction configuration
    (temperature 0, max_token 50)
- name: Streamlit (1.33)
  role: Hosts the human evaluation application used to review extracted option labels
    against raw responses
- name: Python 3.11
  role: Runtime used for loading responses and driving the extraction pipeline
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:4f6c6d37c685581127238eedc9b4a6edeaf73c2b2a592b0dbd4babb029c27154
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
  - OpenAI Python package
  - Phi3-14b (6-bit)
  - Python
  - Python 3.11
  - Streamlit (1.33)
  - o1-preview
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# llm-response-option-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A semi-automated evaluation step that converts raw free-text LLM responses on gastroenterology ACG-SA MCQs into scorable selected options (A–E) by calling the GPT-3.5 API for extraction, followed by human review. It is required because LLM responses are free-form text that cannot be directly scored as correct/incorrect.

## When to use

Use when raw free-text LLM/VLM responses to multiple-choice questions must be mapped to a single scorable option label (A–E) before computing accuracy.

## When NOT to use

- The model output is already a single machine-readable option letter, so no extraction step is needed
- Responses are incoherent/uncodable for more than half the questions — the article's filter step instead excludes such models outright (e.g., Meditron-7b-Q16 and HippoMistral-7b-Q16)
- You need qualitative analysis of response content (justification quality, hallucination) rather than option identity — binary correct/incorrect extraction loses that information

## Inputs

- Raw free-text LLM/VLM responses to ACG-SA gastroenterology MCQs (e.g., from the 2022 ACG-SA experiment outputs)
- Answer key mapping questions to correct options A–E
- OpenAI API access to GPT-3.5

## Outputs

- extracted_options.csv — one normalized option label (A–E, or sentinel NOP/EOP/2OP/error) per LLM response
- Human-review audit results (e.g., 100-answer sample accuracy ~98.8%)
- Scorable correct/incorrect table used for downstream accuracy computation per model

## How to apply

Load the raw textual responses (e.g., from the Sdamirsa/LLM-VLM-in-Gastroenterology outputs for the 162 text-only and 138 image-inclusive 2022 ACG-SA questions) with Python 3.11. For each response, call the GPT-3.5 API via the OpenAI Python package, wrapped with LangChain (0.1.17/0.2.4) to standardize interaction, using temperature 0 and max_token 50 so each call deterministically maps one response to a single label: a valid option (A–E) or a sentinel (NOP, EOP, 2OP, error) when the answer is absent, ambiguous, or multiple. Collect the normalized labels per response into a table (extracted_options.csv). Rationale for the semi-automated design: fully automated extraction is error-prone, so validate by manually reviewing a sample of extracted labels against raw responses before accepting the file; the human review can be run in a Streamlit (1.33) evaluation app.

## Related tools

- **OpenAI Python package** (Calls the GPT-3.5 API to extract the selected option from each raw textual response)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes the API interaction and structured-output extraction configuration (temperature 0, max_token 50))
- **Streamlit (1.33)** (Hosts the human evaluation application used to review extracted option labels against raw responses)
- **Python 3.11** (Runtime used for loading responses and driving the extraction pipeline)

## Examples

```
openai.ChatCompletion.create(model='gpt-3.5-turbo', temperature=0, max_tokens=50, messages=[{'role':'user','content': llm_response + ' Select the chosen option A-E.'}])
```

## Evaluation signals

- Manual review of a 100-answer sample of extracted labels against raw responses reaches the paper-reported ~98.8% extraction accuracy
- Every raw response maps to exactly one output label: a single option A–E or one of the sentinels NOP, EOP, 2OP, error (no missing or multi-label rows)
- Rows in extracted_options.csv correspond one-to-one with raw responses and the ACG-SA answer key question IDs
- Downstream per-model accuracies computed from extracted labels reproduce reported values (e.g., o1-preview 82.0%, 8-bit Llama 3.2-11b 51.7%)
- Extraction calls are deterministic across reruns (temperature 0, max_token 50), yielding identical labels on repeated runs

## Limitations

- Information loss from evaluating responses as simply correct or incorrect — even a correctly selected option may co-occur with hallucinations in the response text
- Extraction config was optimized only for OpenAI (GPT-3.5) models and reused; generalizability of a configuration developed for one LLM to another is uncertain
- The extraction itself is LLM-mediated, so systematic extraction biases require the human review step to catch; the pipeline is semi-automated, not fully autonomous

## Evidence

- [other] setting structured-output extraction with temperature 0 and max_token 50 so each call deterministically maps one raw response to a single label (A-E): [evidence span withheld — anchor status: not_found]
- [other] manually review a 100-answer sample of the extracted labels against the raw responses and confirm the extraction accuracy reaches the paper-reported ~98.8%: [evidence span withheld — anchor status: not_found]
