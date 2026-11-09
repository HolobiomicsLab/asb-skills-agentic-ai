---
name: quantization-accuracy-degradation-comparison
description: Use when comparing accuracy of quantized (Q4/Q6/Q8 GGUF) medical LLMs
  vs full-precision versions on a fixed MCQ benchmark to measure compression-induced
  degradation.
when_to_use_negative:
- The research question concerns vision-language model performance on image-containing
  questions — this skill is restricted to text-only MCQs with image-inclusive questions
  excluded
- The benchmark domain is not gastroenterology MCQs; the article notes the benchmark
  'is limited to questions pertaining to gastroenterology' and may not generalize
  to all of medicine
- The goal is to optimize prompt/model settings per model — the article notes settings
  were optimized only on GPT-3.5 and reused for other LLMs, so this skill is wrong
  for per-model configuration development
tools:
- name: Llama 3.2-11b (8-bit)
  role: Best-performing small quantized open-source model (51.7%); deployed locally
    as GGUF weights for inference over the 2022 ACG-SA text-only questions
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: Phi3-14b (6-bit)
  role: Second-best local-hardware quantized model (48.7%); deployed from Bartowski/Phi-3-medium-128k-instruct-GGUF
    weights
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: Python 3.11
  role: Runtime environment used for API interactions and the quantized-model inference
    pipeline
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes model interactions and prompts across full-precision and quantized
    LLMs
- name: OpenAI Python package
  role: Used to call the GPT-3.5 API for semi-automated extraction of the selected
    option from each model's textual response
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:fceebb2291f1769b96178f8d245032c494ae50faa7cab186d0df1318a243d606
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# quantization-accuracy-degradation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks quantized (GGUF) open-source LLMs against their full-precision counterparts on gastroenterology MCQs from the 2022 ACG-SA, quantifying accuracy loss from compression (e.g., Llama3-8b dropping from 43.3% to 31% after 8-bit quantization) and identifying which quantized models retain full-precision-level performance.

## When to use

Use when comparing accuracy of quantized (Q4/Q6/Q8 GGUF) medical LLMs vs full-precision versions on a fixed MCQ benchmark to measure compression-induced degradation.

## When NOT to use

- The research question concerns vision-language model performance on image-containing questions — this skill is restricted to text-only MCQs with image-inclusive questions excluded
- The benchmark domain is not gastroenterology MCQs; the article notes the benchmark 'is limited to questions pertaining to gastroenterology' and may not generalize to all of medicine
- The goal is to optimize prompt/model settings per model — the article notes settings were optimized only on GPT-3.5 and reused for other LLMs, so this skill is wrong for per-model configuration development

## Inputs

- 2022 ACG-SA text-only question set and answer key from the LLM-VLM-in-Gastroenterology repository (image-inclusive questions excluded)
- Quantized GGUF model weights (8-bit Llama 3.2-11b, 6-bit Phi3-14b from Bartowski/Phi-3-medium-128k-instruct-GGUF, medicineLLM-7b-Q8 from TheBloke/medicine-LLM-GGUF, OpenBioLLM-8b-Q8 from bartowski/OpenBioLLM-Llama3-8B-GGUF, meditron-70b-Q4)
- Reported full-precision accuracy values for each model to serve as comparison baselines
- Standardized QA prompt and GPT-3.5-optimized temperature/max-token settings from the parameter-optimization experiment

## Outputs

- quantization_comparison.csv — table pairing each quantized model with its full-precision counterpart and accuracy percentages (e.g., Llama3-8b 43.3% vs 31%; 8-bit Llama 3.2-11b 51.7%; 6-bit Phi3-14b 48.7%; medicineLLM 27%; OpenBioLLM 29%; meditron-70b-Q4 29.7%)
- Per-question selected-option extractions (GPT-3.5-parsed, human-reviewed) for each quantized model
- Quantized-vs-full-precision accuracy deltas per model, flagged when deviating materially from reported figures

## How to apply

Load the 2022 ACG-SA question set and answer key from the LLM-VLM-in-Gastroenterology repository, excluding image-inclusive questions (162 text-only questions remain). Deploy quantized GGUF weights locally — 8-bit Llama 3.2-11b, 6-bit Phi3-14b, plus medically fine-tuned medicineLLM-7b-Q8, OpenBioLLM-8b-Q8, and meditron-70b-Q4 — and run inference with the standardized QA prompt and the GPT-3.5-optimized settings (temperature and max-token) determined in the parameter-optimization experiment, using Python 3.11 and LangChain (0.1.17/0.2.4) for interaction standardization. Extract the selected option from each textual response via the GPT-3.5 API through the OpenAI Python package, followed by human review of the semi-automated extraction. Score accuracy as the proportion of correctly selected options and compile a table pairing each quantized model with its full-precision counterpart (e.g., Llama3-8b 43.3% full-precision vs 31% at 8-bit) and the local-hardware leaders (8-bit Llama 3.2-11b 51.7%, 6-bit Phi3-14b 48.7%). Rationale: quantization enables local deployment without protected-health-information exposure, but its effect on reasoning must be measured rather than assumed.

## Related tools

- **Llama 3.2-11b (8-bit)** (Best-performing small quantized open-source model (51.7%); deployed locally as GGUF weights for inference over the 2022 ACG-SA text-only questions) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **Phi3-14b (6-bit)** (Second-best local-hardware quantized model (48.7%); deployed from Bartowski/Phi-3-medium-128k-instruct-GGUF weights) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **Python 3.11** (Runtime environment used for API interactions and the quantized-model inference pipeline)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes model interactions and prompts across full-precision and quantized LLMs)
- **OpenAI Python package** (Used to call the GPT-3.5 API for semi-automated extraction of the selected option from each model's textual response)

## Evaluation signals

- Reproduced 8-bit Llama 3.2-11b accuracy matches 51.7% and 6-bit Phi3-14b matches 48.7%
- Llama3-8b 8-bit quantized accuracy reproduces ~31% vs 43.3% full-precision (the largest reported quantization-related degradation)
- Most other quantized-vs-full-precision deltas fall within the reported ranges, i.e., scores comparable to full-precision counterparts; flag any model with material deviation for expert review
- Fine-tuned quantized baselines reproduce: medicineLLM 27%, OpenBioLLM 29%, meditron-70b-Q4 29.7%
- quantization_comparison.csv contains one paired full-precision/quantized row per model with accuracy as proportion of correctly selected options, and extraction quality is human-reviewed after GPT-3.5 parsing

## Limitations

- GPT-3.5-optimized prompt/settings were reused for other LLMs; per-model configuration may be more effective, and generalizability of a configuration developed for one LLM to another is uncertain
- Quantized-model local deployment is a nascent field with many libraries and modules still in beta development
- The impact of quantization on model reasoning has not been explored, and limited experimental runs mean no average and SD are reported to assess consistency

## Evidence
