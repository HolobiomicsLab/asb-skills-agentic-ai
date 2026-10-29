---
name: meal-plan-generation-prompt-execution
description: Use when benchmarking how quickly different LLMs generate personalized
  meal plans under a fixed prompt — e.g., reproducing the NutriGen per-model total
  processing-time ranking (10 outputs per model).
when_to_use_negative:
- When the goal is accuracy evaluation against caloric targets (MAE/percentage error
  vs USDA nutrition database) rather than runtime benchmarking
- When no valid API keys are available for the LLM providers, since the notebook requires
  replacing "KEY" placeholders before execution
- When evaluating dietary recommendation quality rather than comparing LLM computational
  efficiency
edam_topics:
- http://edamontology.org/topic_3376
tools:
- name: NutriGen
  role: Framework whose prompt is executed 10 times per model to generate personalized
    meal plans
  repo: https://github.com/SamanKhamesian/NutriGen
- name: GPT-3.5 Turbo
  role: Evaluated LLM; among the fastest processing times
- name: Gemini 2.0 Flash
  role: Evaluated LLM; among the fastest processing times
- name: Claude 3.5 Haiku
  role: Evaluated LLM; among the fastest processing times
- name: Llama 3.1 8B
  role: Evaluated LLM accessed via llamaapi 0.1.36
- name: DeepSeek V3
  role: Evaluated LLM; exhibits the longest runtime, indicating higher computational
    demands
- name: Python 3.10
  role: Runtime environment for the NutriGen Jupyter Notebook
  repo: https://github.com/SamanKhamesian/NutriGen
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:f6b5ac052deda27d1dd42141257391e3ea0f9f0c42ac569a6d2504367abd491b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SamanKhamesian/NutriGen
  tools:
  - Claude 3.5 Haiku
  - DeepSeek V3
  - GPT-3.5 Turbo
  - Gemini 2.0 Flash
  - Llama 3.1 8B
  - NutriGen
  - Python
  - Python 3.10
  - anthropic
  - google.generativeai
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# meal-plan-generation-prompt-execution

## Summary

Executes the NutriGen prompt via LLM APIs (GPT-3.5 Turbo, Gemini 2.0 Flash, Claude 3.5 Haiku, Llama 3.1 8B, DeepSeek V3) to generate 10 personalized meal-plan outputs per model and record per-model total processing time for a computational-efficiency comparison.

## When to use

Use when benchmarking how quickly different LLMs generate personalized meal plans under a fixed prompt — e.g., reproducing the NutriGen per-model total processing-time ranking (10 outputs per model).

## When NOT to use

- When the goal is accuracy evaluation against caloric targets (MAE/percentage error vs USDA nutrition database) rather than runtime benchmarking
- When no valid API keys are available for the LLM providers, since the notebook requires replacing "KEY" placeholders before execution
- When evaluating dietary recommendation quality rather than comparing LLM computational efficiency

## Inputs

- NutriGen prompt (personalized meal-plan request with user-defined caloric target and dietary constraints)
- Valid API keys for each model provider (OpenAI, Anthropic, Google, LlamaAPI, DeepSeek)
- Pinned Python 3.10 environment from requirements.txt
- NutriGen Jupyter Notebook from github.com/SamanKhamesian/NutriGen

## Outputs

- Per-model total processing time for 10 generated meal-plan outputs
- Timing table aggregating total processing time per model
- Bar-chart comparison of total processing time per model (figures/time_comparison.png)
- 10 meal-plan outputs per LLM

## How to apply

Set up a Python 3.10 environment with the pinned dependencies (anthropic 0.45.2, google.generativeai 0.8.4, llamaapi 0.1.36, openai 1.42.0, numpy 2.2.2, pandas 2.2.3) via pip install -r requirements.txt. Open the provided NutriGen Jupyter Notebook and replace every "KEY" placeholder with a valid API key for each model provider before execution. For each of the five models (GPT-3.5 Turbo, Gemini 2.0 Flash, Claude 3.5 Haiku, Llama 3.1 8B, DeepSeek V3), run the NutriGen prompt 10 times and record the total processing time required to generate the 10 outputs, following the paper's method of 'measuring the total processing time required to generate 10 outputs'. Aggregate per-model totals into a timing table and plot a bar-chart comparison (figures/time_comparison.png). The rationale is that runtime differences reveal computational demands: smaller optimized models complete fastest while larger models like DeepSeek V3 are significantly slower.

## Related tools

- **NutriGen** (Framework whose prompt is executed 10 times per model to generate personalized meal plans) — https://github.com/SamanKhamesian/NutriGen
- **GPT-3.5 Turbo** (Evaluated LLM; among the fastest processing times)
- **Gemini 2.0 Flash** (Evaluated LLM; among the fastest processing times)
- **Claude 3.5 Haiku** (Evaluated LLM; among the fastest processing times)
- **Llama 3.1 8B** (Evaluated LLM accessed via llamaapi 0.1.36)
- **DeepSeek V3** (Evaluated LLM; exhibits the longest runtime, indicating higher computational demands)
- **Python 3.10** (Runtime environment for the NutriGen Jupyter Notebook) — https://github.com/SamanKhamesian/NutriGen

## Examples

```
pip install -r requirements.txt && jupyter notebook
```

## Evaluation signals

- Each model produced exactly 10 meal-plan outputs from the NutriGen prompt before timing stops
- GPT-3.5 Turbo, Gemini 2.0 Flash, and Claude 3.5 Haiku rank fastest in the timing table, matching the paper's reported ranking
- DeepSeek V3 shows the longest total runtime in the bar chart (figures/time_comparison.png), consistent with its reported higher computational demand
- Timing table totals match the values plotted in figures/time_comparison.png
- All "KEY" placeholders were replaced with valid API keys so every model completes without authentication errors

## Limitations

- DeepSeek V3 was significantly slower in processing time than GPT-3.5 Turbo and Llama 3.1 8B and struggled with accuracy (MAE 10.45%), highlighting the need for further optimization
- Timing results depend on live API availability and valid per-provider keys; the notebook will not execute with "KEY" placeholders
- Runtime comparisons reflect total time for 10 outputs under the paper's conditions and may vary with provider load or API changes

## Evidence
