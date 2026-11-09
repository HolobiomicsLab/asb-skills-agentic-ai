---
name: runtime-comparison-visualization-across-models
description: Use when you must compare computational/processing-time efficiency across
  several LLM APIs on the same prompt workload and verify which optimized smaller
  models are fastest versus a slow heavyweight like DeepSeek V3.
when_to_use_negative:
- Your goal is accuracy of generated meal plans (caloric MAE vs. targets), not runtime
  — use the USDA vs. target comparison analysis instead
- You only evaluate a single model with no cross-model comparison, since the skill's
  output is a comparative ranking/figure
- No API access to the model providers is available, since timing requires live generation
  of 10 outputs per model
edam_operation: http://edamontology.org/operation_0337
tools:
- name: NutriGen
  role: Framework whose prompt is executed 10 times per model to measure total processing
    time
  repo: github.com/SamanKhamesian/NutriGen
- name: GPT-3.5 Turbo
  role: Evaluated model; one of the fastest runtimes against which others are compared
- name: Gemini 2.0 Flash
  role: Evaluated model; fast optimized runtime in the comparison
- name: Claude 3.5 Haiku
  role: Evaluated model; fast optimized runtime in the comparison
- name: Llama 3.1 8B
  role: Evaluated model accessed via llamaapi 0.1.36
- name: DeepSeek V3
  role: Evaluated model; baseline slow runtime demonstrating higher computational
    demands
- name: anthropic
  role: Python client (pinned 0.45.2) used to call Claude 3.5 Haiku during timing
- name: google.generativeai
  role: Python client (pinned 0.8.4) used to call Gemini 2.0 Flash during timing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:81fc27607dc1f8e8cd33a10a97c91416fce932c7b3d5abbb8f8eb9dc81c1a94c
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# runtime-comparison-visualization-across-models

## Summary

Benchmark the computational efficiency of multiple LLMs (GPT-3.5 Turbo, Gemini 2.0 Flash, Claude 3.5 Haiku, Llama 3.1 8B, DeepSeek V3) by timing 10 NutriGen meal-plan generations per model, aggregating totals, and producing a bar-chart comparison (figures/time_comparison.png).

## When to use

Use when you must compare computational/processing-time efficiency across several LLM APIs on the same prompt workload and verify which optimized smaller models are fastest versus a slow heavyweight like DeepSeek V3.

## When NOT to use

- Your goal is accuracy of generated meal plans (caloric MAE vs. targets), not runtime — use the USDA vs. target comparison analysis instead
- You only evaluate a single model with no cross-model comparison, since the skill's output is a comparative ranking/figure
- No API access to the model providers is available, since timing requires live generation of 10 outputs per model

## Inputs

- NutriGen Jupyter Notebook with 'KEY' API-key placeholders for each model provider
- Valid API keys for OpenAI (GPT-3.5 Turbo), Google (Gemini 2.0 Flash), Anthropic (Claude 3.5 Haiku), LlamaAPI (Llama 3.1 8B), and DeepSeek V3
- requirements.txt with pinned versions (anthropic 0.45.2, google.generativeai 0.8.4, llamaapi 0.1.36, openai 1.42.0, numpy 2.2.2, pandas 2.2.3)

## Outputs

- Per-model total processing time for 10 meal-plan generations (timing table)
- Bar-chart figure figures/time_comparison.png comparing total processing time per model
- Verified model efficiency ranking (GPT-3.5 Turbo, Gemini 2.0 Flash, Claude 3.5 Haiku fastest; DeepSeek V3 slowest)

## How to apply

Install the pinned dependencies (anthropic 0.45.2, google.generativeai 0.8.4, llamaapi 0.1.36, openai 1.42.0, numpy 2.2.2, pandas 2.2.3) under Python 3.10, then open the NutriGen Jupyter Notebook and replace each 'KEY' placeholder with a valid provider API key. For each model, run the same NutriGen prompt 10 times and record the total processing time required to generate the 10 outputs, as the paper did. Aggregate per-model total times into a timing table (e.g., with pandas), then plot a bar chart of total processing time per model to figures/time_comparison.png. Judge correctness by checking the reproduced ranking matches the reported qualitative finding: smaller optimized models (GPT-3.5 Turbo, Gemini 2.0 Flash, Claude 3.5 Haiku) complete fastest while DeepSeek V3 is significantly slower due to higher computational demands.

## Related tools

- **NutriGen** (Framework whose prompt is executed 10 times per model to measure total processing time) — github.com/SamanKhamesian/NutriGen
- **GPT-3.5 Turbo** (Evaluated model; one of the fastest runtimes against which others are compared)
- **Gemini 2.0 Flash** (Evaluated model; fast optimized runtime in the comparison)
- **Claude 3.5 Haiku** (Evaluated model; fast optimized runtime in the comparison)
- **Llama 3.1 8B** (Evaluated model accessed via llamaapi 0.1.36)
- **DeepSeek V3** (Evaluated model; baseline slow runtime demonstrating higher computational demands)
- **anthropic** (Python client (pinned 0.45.2) used to call Claude 3.5 Haiku during timing)
- **google.generativeai** (Python client (pinned 0.8.4) used to call Gemini 2.0 Flash during timing)

## Examples

```
pip install -r requirements.txt; jupyter notebook  # open NutriGen notebook, replace "KEY" with valid API keys, run 10 outputs per model, save figures/time_comparison.png
```

## Evaluation signals

- Timing table contains exactly one total-time value per evaluated model (GPT-3.5 Turbo, Gemini 2.0 Flash, Claude 3.5 Haiku, Llama 3.1 8B, DeepSeek V3), each measured over 10 outputs
- Reproduced bar chart (figures/time_comparison.png) matches the paper's qualitative ranking: optimized smaller models fastest, DeepSeek V3 significantly slower
- All API calls succeed after replacing 'KEY' placeholders, so no model's timing is missing or defaulted
- Environment matches pinned versions (Python 3.10, anthropic 0.45.2, google.generativeai 0.8.4, llamaapi 0.1.36, openai 1.42.0, numpy 2.2.2, pandas 2.2.3) so timings are comparable

## Limitations

- Timing is qualitative/comparative: the paper reports relative ranking rather than statistically controlled benchmarking across hardware or network conditions
- DeepSeek V3 was significantly slower and also less accurate (MAE 10.45%), so runtime comparisons should not be conflated with accuracy findings
- Timing depends on live API access and valid keys for each provider; results are not reproducible without them

## Evidence
