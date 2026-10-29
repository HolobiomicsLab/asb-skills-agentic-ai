---
name: llm-inference-latency-benchmarking
description: Use when comparing computational efficiency of multiple LLM APIs on an
  identical generation task, e.g. ranking models by total processing time over a fixed
  number of prompt runs.
when_to_use_negative:
- The goal is accuracy/quality comparison (e.g. MAE against caloric targets) rather
  than runtime — use the USDA-vs-target calorie comparison instead.
- The task is a single generation per model; total-time-per-10-outputs benchmarking
  requires a fixed, repeated run count for fair comparison.
- Models are evaluated on local hardware where wall-clock time conflates hardware
  differences rather than API-side computational demands.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: NutriGen
  role: LLM framework whose meal-plan generation prompt is the fixed workload timed
    10 times per model
  repo: github.com/SamanKhamesian/NutriGen
- name: GPT-3.5 Turbo
  role: Evaluated model; among the fastest in processing time
- name: Gemini 2.0 Flash
  role: Evaluated model; among the fastest via google.generativeai 0.8.4
- name: Claude 3.5 Haiku
  role: Evaluated model; among the fastest via anthropic 0.45.2
- name: Llama 3.1 8B
  role: Evaluated model; accessed via llamaapi 0.1.36
- name: DeepSeek V3
  role: Evaluated model; exhibited the longest runtime, indicating higher computational
    demands
- name: Python
  role: Python 3.10 runtime executing the Jupyter Notebook timing harness
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:2876361248441202eacae16ebe1e747bff32006341edd67b1d6847c1b3058bc1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Claude 3.5 Haiku
  - DeepSeek V3
  - GPT-3.5 Turbo
  - Gemini 2.0 Flash
  - Llama 3.1 8B
  - NutriGen
  - Python
  - anthropic
  - google.generativeai
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# llm-inference-latency-benchmarking

## Summary

Benchmark the computational efficiency of LLMs by measuring the total processing time each model needs to generate 10 outputs of the same structured task (here, NutriGen meal plans), then aggregate and compare per-model runtimes as a bar chart.

## When to use

Use when comparing computational efficiency of multiple LLM APIs on an identical generation task, e.g. ranking models by total processing time over a fixed number of prompt runs.

## When NOT to use

- The goal is accuracy/quality comparison (e.g. MAE against caloric targets) rather than runtime — use the USDA-vs-target calorie comparison instead.
- The task is a single generation per model; total-time-per-10-outputs benchmarking requires a fixed, repeated run count for fair comparison.
- Models are evaluated on local hardware where wall-clock time conflates hardware differences rather than API-side computational demands.

## Inputs

- NutriGen prompt (personalized meal-plan generation) run against each LLM provider API
- Valid API keys for each model provider (GPT-3.5 Turbo, Gemini 2.0 Flash, Claude 3.5 Haiku, Llama 3.1 8B, DeepSeek V3)
- requirements.txt with pinned dependencies under Python 3.10

## Outputs

- Per-model total processing time for generating 10 outputs (timing table)
- Bar-chart comparison of total processing time per model (figures/time_comparison.png)
- Qualitative ranking of models by computational efficiency

## How to apply

Set up the pinned Python 3.10 environment from requirements.txt (anthropic 0.45.2, google.generativeai 0.8.4, llamaapi 0.1.36, openai 1.42.0, numpy 2.2.2, pandas 2.2.3) and insert valid provider API keys in place of the "KEY" placeholders in the Jupyter Notebook. For each model (e.g. GPT-3.5 Turbo, Gemini 2.0 Flash, Claude 3.5 Haiku, Llama 3.1 8B, DeepSeek V3), run the same prompt 10 times and record the total processing time required to generate the 10 outputs. Aggregate the per-model totals into a timing table and plot a bar-chart comparison (figures/time_comparison.png). Interpret results in light of model size/optimization: the paper expects smaller optimized models (GPT-3.5 Turbo, Gemini 2.0 Flash, Claude 3.5 Haiku) to be fastest, while larger models like DeepSeek V3 run significantly longer, reflecting higher computational demands.

## Related tools

- **NutriGen** (LLM framework whose meal-plan generation prompt is the fixed workload timed 10 times per model) — github.com/SamanKhamesian/NutriGen
- **GPT-3.5 Turbo** (Evaluated model; among the fastest in processing time)
- **Gemini 2.0 Flash** (Evaluated model; among the fastest via google.generativeai 0.8.4)
- **Claude 3.5 Haiku** (Evaluated model; among the fastest via anthropic 0.45.2)
- **Llama 3.1 8B** (Evaluated model; accessed via llamaapi 0.1.36)
- **DeepSeek V3** (Evaluated model; exhibited the longest runtime, indicating higher computational demands)
- **Python** (Python 3.10 runtime executing the Jupyter Notebook timing harness)

## Examples

```
pip install -r requirements.txt && jupyter notebook NutriGen.ipynb  # set each "KEY" placeholder, then run the prompt 10x per model and record total processing time
```

## Evaluation signals

- The reproduced bar chart (figures/time_comparison.png) matches the paper's qualitative finding: GPT-3.5 Turbo, Gemini 2.0 Flash, and Claude 3.5 Haiku complete fastest.
- DeepSeek V3 shows the longest runtime among evaluated models, consistent with the paper's report that it 'was significantly slower in processing time'.
- Each model is run exactly 10 times with the same prompt, and total processing time is recorded consistently across models.
- Timing values are aggregated into a per-model table before plotting, with no missing models in the comparison.

## Limitations

- Processing times reflect API-side and network behavior at run time; runtimes can vary, so the comparison is qualitative (ranking) rather than exact.
- Runtime ranking should not be conflated with accuracy: DeepSeek V3 was slowest and least accurate (MAE 10.45%), but fast models like GPT-3.5 Turbo were not the most accurate (1.55% error for Llama 3.1 8B vs 3.68% for GPT-3.5 Turbo).
- Requires valid API keys for each provider; execution fails without replacing the "KEY" placeholders.

## Evidence

- [methods] Environment setup: [evidence span withheld — anchor status: not_found]
