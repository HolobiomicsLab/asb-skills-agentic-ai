---
name: local-hardware-llm-deployment
description: Use when benchmarking quantized open-source LLMs (e.g., GGUF 4/6/8-bit
  weights) on MCQ datasets under local compute constraints, or comparing quantized-vs-full-precision
  accuracy deltas.
when_to_use_negative:
- Evaluating proprietary/API-only models (o1-preview, Claude3.5-Sonnet, GPT-3.5) that
  cannot be quantized or deployed locally
- Benchmarking VLMs on image-inclusive questions — this skill covers text-only inference
  after image questions are excluded
- Assessing the impact of quantization on model reasoning quality beyond binary correct/incorrect
  MCQ scoring — the article notes this impact has not been explored
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3376
- http://edamontology.org/topic_3474
tools:
- name: Llama 3.2-11b (8-bit)
  role: Best-performing locally deployed quantized model on text-only ACG-SA questions
    (51.7% accuracy)
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: Phi3-14b (6-bit)
  role: Second-best local-hardware quantized model (48.7% accuracy), GGUF weights
    from Bartowski/Phi-3-medium-128k-instruct-GGUF
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: Python 3.11
  role: Runtime for local inference and API interactions
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes LLM interaction/prompting across locally deployed models
- name: OpenAI Python package
  role: GPT-3.5 API used for semi-automated extraction of selected options from model
    responses
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:067c54fc09ebd2c94c716b83f2e181513ddfcc8baa035f9f75f9aaf91ef41984
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  tools:
  - LangChain (0.1.17 and 0.2.4)
  - Llama 3.2-11b (8-bit)
  - OpenAI Python package
  - Phi3-14b (6-bit)
  - Python
  - Python 3.11
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3376
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# local-hardware-llm-deployment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Deploys quantized open-source LLM weights (GGUF format, 4–8 bit) locally for benchmarking on medical MCQs, enabling comparison of quantized vs full-precision accuracy and identification of best local-hardware models.

## When to use

Use when benchmarking quantized open-source LLMs (e.g., GGUF 4/6/8-bit weights) on MCQ datasets under local compute constraints, or comparing quantized-vs-full-precision accuracy deltas.

## When NOT to use

- Evaluating proprietary/API-only models (o1-preview, Claude3.5-Sonnet, GPT-3.5) that cannot be quantized or deployed locally
- Benchmarking VLMs on image-inclusive questions — this skill covers text-only inference after image questions are excluded
- Assessing the impact of quantization on model reasoning quality beyond binary correct/incorrect MCQ scoring — the article notes this impact has not been explored

## Inputs

- 2022 ACG-SA text-only question set (162 questions) with answer key, image-inclusive questions excluded
- Quantized GGUF model weights (e.g., 8-bit Llama 3.2-11b, 6-bit Phi3-14b, medicineLLM-7b-Q8, OpenBioLLM-8b-Q8, meditron-70b-Q4)
- Standardized QA prompt and GPT-3.5-optimized settings (temperature, max-token)
- Full-precision counterpart accuracies for delta comparison

## Outputs

- Per-question model responses with extracted selected options
- Accuracy scores per quantized model (e.g., 8-bit Llama 3.2-11b 51.7%, 6-bit Phi3-14b 48.7%)
- Quantized-vs-full-precision comparison table (e.g., quantization_comparison.csv)

## How to apply

Load the text-only question set (162 questions from the 2022 ACG-SA, image-inclusive questions excluded) and answer key, then deploy quantized GGUF weights locally — e.g., 8-bit Llama 3.2-11b and 6-bit Phi3-14b (Bartowski/Phi-3-medium-128k-instruct-GGUF) — running inference with the standardized QA prompt and the temperature/max-token settings optimized in the GPT-3.5 parameter-optimization experiment, orchestrated via Python 3.11 and LangChain (0.1.17 and 0.2.4). Apply the same pipeline to medically fine-tuned quantized models (medicineLLM-7b-Q8, OpenBioLLM-8b-Q8, meditron-70b-Q4). Extract the selected option from each free-text response with the GPT-3.5 API via the OpenAI Python package, followed by human review of the semi-automated extraction. Score accuracy as the proportion of correctly selected options and compile a comparison table pairing each quantized model with its full-precision counterpart (e.g., Llama3-8b 43.3% full-precision vs 31% at 8-bit) to quantify quantization loss.

## Related tools

- **Llama 3.2-11b (8-bit)** (Best-performing locally deployed quantized model on text-only ACG-SA questions (51.7% accuracy)) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **Phi3-14b (6-bit)** (Second-best local-hardware quantized model (48.7% accuracy), GGUF weights from Bartowski/Phi-3-medium-128k-instruct-GGUF) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **Python 3.11** (Runtime for local inference and API interactions)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes LLM interaction/prompting across locally deployed models)
- **OpenAI Python package** (GPT-3.5 API used for semi-automated extraction of selected options from model responses)

## Evaluation signals

- Reproduced accuracy of 8-bit Llama 3.2-11b matches 51.7% and 6-bit Phi3-14b matches 48.7%
- Quantized-vs-full-precision deltas fall within reported ranges (e.g., Llama3-8b dropping from 43.3% to 31% at 8-bit); flag any model deviating materially for expert review
- Fine-tuned quantized baselines reproduce: medicineLLM-7b-Q8 27%, OpenBioLLM-8b-Q8 29%, meditron-70b-Q4 29.7%
- Image-inclusive questions are excluded so the evaluated set is exactly the 162 text-only questions from 2022 ACG-SA
- Every textual response has an extracted option confirmed by human review of the GPT-3.5 semi-automated extraction

## Limitations

- Prompt/settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs; per-model prompt configuration is suggested as more effective and generalizability is uncertain
- Quantized models in local environments are a nascent field, with many libraries and modules still in beta development
- Meditron-7b-Q16 and HippoMistral-7b-Q16 were excluded for inability to provide coherent responses on >50% of questions
- Limited number of experimental runs; no average and SD reported to assess model consistency

## Evidence
