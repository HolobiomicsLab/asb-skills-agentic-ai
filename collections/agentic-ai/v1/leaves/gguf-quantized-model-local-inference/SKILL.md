---
name: gguf-quantized-model-local-inference
description: Use when benchmarking medically or generally capable LLMs on MCQ question
  sets under local-hardware constraints, or when comparing quantized (4/6/8-bit) vs
  full-precision accuracy deltas.
when_to_use_negative:
- The question set contains image-inclusive items — this skill covers the text-only
  pipeline; image handling requires the VLM/captioning workflow (Experiment 2).
- A proprietary API model (e.g., o1-preview, Claude3.5-Sonnet) can be used and no
  local-hardware or data-privacy constraint motivates quantized local deployment.
- The target model is known to produce incoherent output when quantized — Meditron-7b-Q16
  and HippoMistral-7b-Q16 were excluded from the paper's analysis for incoherent responses
  on >50% of questions.
tools:
- name: Llama 3.2-11b (8-bit)
  role: Best-performing 8-bit quantized model run locally (51.7% accuracy)
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: Phi3-14b (6-bit)
  role: 6-bit GGUF quantized model run locally (48.7% accuracy), via Bartowski/Phi-3-medium-128k-instruct-GGUF
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: TheBloke/medicine-LLM-GGUF
  role: MedicineLLM-7b-Q8 weights (based on Llama-2) for the medically fine-tuned
    quantized comparison (27%)
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: bartowski/OpenBioLLM-Llama3-8B-GGUF
  role: OpenBioLLM-8b-Q8 weights (based on Llama-3) for the medically fine-tuned quantized
    comparison (29%)
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: Python 3.11
  role: Runtime for local inference and API interactions
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes prompt/API interactions across quantized models
- name: OpenAI Python package
  role: GPT-3.5 API extraction of the selected option from each textual response
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:e8a039e8ad0618b63cdc36e6c94db19243d7178bf4f82d33421354f5c94802d4
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
  - TheBloke/medicine-LLM-GGUF
  - bartowski/OpenBioLLM-Llama3-8B-GGUF
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# GGUF quantized model local inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs quantized GGUF weights (e.g., 8-bit Llama 3.2-11b, 6-bit Phi3-14b, medicineLLM-7b-Q8, OpenBioLLM-8b-Q8, meditron-70b-Q4) locally on the 2022 ACG-SA text-only gastroenterology MCQ set to measure how 4–8-bit compression affects accuracy relative to full-precision counterparts.

## When to use

Use when benchmarking medically or generally capable LLMs on MCQ question sets under local-hardware constraints, or when comparing quantized (4/6/8-bit) vs full-precision accuracy deltas.

## When NOT to use

- The question set contains image-inclusive items — this skill covers the text-only pipeline; image handling requires the VLM/captioning workflow (Experiment 2).
- A proprietary API model (e.g., o1-preview, Claude3.5-Sonnet) can be used and no local-hardware or data-privacy constraint motivates quantized local deployment.
- The target model is known to produce incoherent output when quantized — Meditron-7b-Q16 and HippoMistral-7b-Q16 were excluded from the paper's analysis for incoherent responses on >50% of questions.

## Inputs

- 2022 ACG-SA text-only question set (162 questions) with answer key, image-inclusive questions excluded
- GGUF quantized model weights (8-bit Llama 3.2-11b, 6-bit Phi3-14b, medicineLLM-7b-Q8, OpenBioLLM-8b-Q8, meditron-70b-Q4)
- Standardized QA prompt and GPT-3.5-optimized temperature and max-token settings
- Full-precision baseline accuracies for paired comparison

## Outputs

- quantization_comparison.csv pairing each quantized model with its full-precision counterpart accuracy
- Per-question textual responses and GPT-3.5-extracted selected options (human-reviewed)
- Accuracy scores (proportion correctly selected options), e.g., 8-bit Llama 3.2-11b 51.7%, 6-bit Phi3-14b 48.7%

## How to apply

Load the 2022 ACG-SA text-only question set (162 text-only questions after excluding the 138 image-inclusive items) and answer key from the LLM-VLM-in-Gastroenterology repository. Deploy the GGUF weights locally — e.g., Bartowski/Phi-3-medium-128k-instruct-GGUF (Phi-3-14b-Q6), TheBloke/medicine-LLM-GGUF (MedicineLLM-7b-Q8), bartowski/OpenBioLLM-Llama3-8B-GGUF (OpenBioLLM-8b-Q8), and Llama-3.2-11B-Vision-Instruct used at 8-bit — and run inference with the standardized QA prompt and the temperature/max-token settings optimized in the GPT-3.5 parameter-optimization experiment (Experiment 0), using Python 3.11 and LangChain (0.1.17/0.2.4) to standardize interactions. Score accuracy as the proportion of correctly selected options, extracting the selected option from each textual response with the GPT-3.5 API via the OpenAI Python package followed by human review. Pair each quantized model with its full-precision counterpart (e.g., Llama3-8b full-precision 43.3% vs 8-bit 31%) and save the comparison as quantization_comparison.csv.

## Related tools

- **Llama 3.2-11b (8-bit)** (Best-performing 8-bit quantized model run locally (51.7% accuracy)) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **Phi3-14b (6-bit)** (6-bit GGUF quantized model run locally (48.7% accuracy), via Bartowski/Phi-3-medium-128k-instruct-GGUF) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **TheBloke/medicine-LLM-GGUF** (MedicineLLM-7b-Q8 weights (based on Llama-2) for the medically fine-tuned quantized comparison (27%)) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **bartowski/OpenBioLLM-Llama3-8B-GGUF** (OpenBioLLM-8b-Q8 weights (based on Llama-3) for the medically fine-tuned quantized comparison (29%)) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **Python 3.11** (Runtime for local inference and API interactions)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes prompt/API interactions across quantized models)
- **OpenAI Python package** (GPT-3.5 API extraction of the selected option from each textual response)

## Evaluation signals

- 8-bit Llama 3.2-11b accuracy reproduces at ~51.7% and 6-bit Phi3-14b at ~48.7% on the 162 text-only 2022 ACG-SA questions
- Quantized-vs-full-precision deltas fall within reported ranges (e.g., Llama3-8b 43.3% → 31% after 8-bit quantization; most other quantized models comparable to full precision)
- Medically fine-tuned quantized results match reported values: medicineLLM 27%, OpenBioLLM 29%, meditron-70b-Q4 29.7%
- quantization_comparison.csv contains one full-precision/quantized pair per model with no missing rows
- Any model deviating materially from its reported accuracy is flagged for expert review before acceptance

## Limitations

- Prompt and settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs; per-model prompt configuration is suggested as more effective and generalizability of the GPT-3.5-derived configuration is uncertain
- Quantized-model local deployment is a nascent field with many libraries and modules still in beta development
- The impact of quantization on model reasoning has not been explored, and Meditron-7b-Q16/HippoMistral-7b-Q16 were excluded for incoherent responses
- The benchmark is limited to gastroenterology questions and may not generalize to all of medicine; limited runs mean no average/SD consistency estimate

## Evidence

- [other] Largest quantization loss: [evidence span withheld — anchor status: not_found]
