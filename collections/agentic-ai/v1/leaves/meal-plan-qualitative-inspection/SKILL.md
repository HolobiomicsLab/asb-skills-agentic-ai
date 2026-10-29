---
name: meal-plan-qualitative-inspection
description: Use when comparing LLM-generated personalized meal plans beyond nutritional
  accuracy, to flag outright generation failures, structural defects, or omitted nutrients
  and disclaimers per model and input.
when_to_use_negative:
- The goal is quantitative accuracy reporting (MAE against caloric targets) rather
  than structural/qualitative output inspection
- Meal plan outputs have not been generated yet and API keys cannot be configured,
  since this skill audits actual generated plans
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: NutriGen
  role: Framework whose Jupyter Notebook generates the LLM meal plans that are inspected
    for qualitative failures
  repo: https://github.com/SamanKhamesian/NutriGen
- name: pandas
  role: Used in Python to compile failure-type and disclaimer entries into a structured
    table saved as qualitative_failures.csv
- name: DeepSeek V3
  role: Evaluated model whose outputs failed outright on inputs 4-7 and showed a higher
    MAE of 10.45%
- name: Claude 3.5 Haiku
  role: Evaluated model via the anthropic client; failed input 5 and produced only
    one plan on inputs 6/8/9/10
- name: Llama 3.1 8B
  role: Evaluated model via llamaapi; failed on input 8 while achieving the lowest
    error of 1.55%
- name: Gemini 1.5 Pro
  role: Evaluated model via google.generativeai; omitted fat and protein values from
    all plans
- name: GPT-3.5 Turbo
  role: Baseline model via openai client with 3.68% error, included in the qualitative
    comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:6a6006e884940acbe4b3bac29f8599a282d1795e2ffd5d7e12758a87043d64b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SamanKhamesian/NutriGen
  tools:
  - Claude 3.5 Haiku
  - DeepSeek V3
  - GPT-3.5 Turbo
  - Gemini 1.5 Pro
  - Llama 3.1 8B
  - NutriGen
  - Python
  - anthropic
  - google.generativeai
  - llamaapi
  - openai
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# meal-plan-qualitative-inspection

## Summary

Qualitative audit of LLM-generated NutriGen meal plans across the 10 evaluation inputs, checking for structural completeness (missing plans, single-plan outputs, omitted fat/protein values) and disclaimer presence per model, complementing quantitative MAE on caloric targets.

## When to use

Use when comparing LLM-generated personalized meal plans beyond nutritional accuracy, to flag outright generation failures, structural defects, or omitted nutrients and disclaimers per model and input.

## When NOT to use

- The goal is quantitative accuracy reporting (MAE against caloric targets) rather than structural/qualitative output inspection
- Meal plan outputs have not been generated yet and API keys cannot be configured, since this skill audits actual generated plans

## Inputs

- Meal plan outputs (one or more per model) for the 10 NutriGen evaluation inputs, generated from the provided Jupyter Notebook
- Per-input target caloric values from the evaluation inputs
- Model identifiers: GPT-3.5 Turbo, Claude 3.5 Haiku, DeepSeek V3, Llama 3.1 8B, Gemini 1.5 Pro, Gemini 2.0 Flash

## Outputs

- qualitative_failures.csv with columns for model, input index, failure type, and disclaimer presence
- Per-model list of qualitative anomalies (missing plans, single-plan outputs, omitted fat/protein values, missing dietary disclaimers)

## How to apply

Re-run the NutriGen Jupyter Notebook (Python 3.10, with anthropic==0.45.2, google.generativeai==0.8.4, openai==1.42.0, llamaapi==0.1.36) after replacing "KEY" with valid API keys to obtain meal plan outputs for each of the 10 evaluation inputs from GPT-3.5 Turbo, Claude 3.5 Haiku, DeepSeek V3, Llama 3.1 8B, Gemini 1.5 Pro, and Gemini 2.0 Flash. Inspect each model's per-input outputs for outright failures (e.g., DeepSeek V3 on inputs 4-7, Claude 3.5 Haiku on input 5, Llama 3.1 8B on input 8) and structural defects such as Claude 3.5 Haiku producing only one plan on inputs 6/8/9/10 or Gemini 1.5 Pro omitting fat and protein values from all plans. For each output also record whether a dietary disclaimer is present. Compile everything into a structured pandas DataFrame with columns model, input index, failure type, and disclaimer presence, then save it as qualitative_failures.csv. Sanity-check the qualitative flags against the paper's quantitative context: DeepSeek V3's higher MAE of 10.45% versus Llama 3.1 8B's 1.55% and GPT-3.5 Turbo's 3.68%.

## Related tools

- **NutriGen** (Framework whose Jupyter Notebook generates the LLM meal plans that are inspected for qualitative failures) — https://github.com/SamanKhamesian/NutriGen
- **pandas** (Used in Python to compile failure-type and disclaimer entries into a structured table saved as qualitative_failures.csv)
- **DeepSeek V3** (Evaluated model whose outputs failed outright on inputs 4-7 and showed a higher MAE of 10.45%)
- **Claude 3.5 Haiku** (Evaluated model via the anthropic client; failed input 5 and produced only one plan on inputs 6/8/9/10)
- **Llama 3.1 8B** (Evaluated model via llamaapi; failed on input 8 while achieving the lowest error of 1.55%)
- **Gemini 1.5 Pro** (Evaluated model via google.generativeai; omitted fat and protein values from all plans)
- **GPT-3.5 Turbo** (Baseline model via openai client with 3.68% error, included in the qualitative comparison)

## Examples

```
import pandas as pd; df = pd.read_csv('qualitative_failures.csv')
```

## Evaluation signals

- The failure record enumerates DeepSeek V3's failures on inputs 4, 5, 6, and 7 and Claude 3.5 Haiku's failure on input 5
- Claude 3.5 Haiku's single-plan outputs are flagged for inputs 6, 8, 9, and 10, and Llama 3.1 8B's failure on input 8 is recorded
- Gemini 1.5 Pro's missing fat and protein values are detected across all of its plans
- Each of the six models has a per-model disclaimer-presence flag in the table
- qualitative_failures.csv exists with the specified columns (model, input index, failure type, disclaimer presence) and is consistent with the paper's MAE ranking (DeepSeek V3 10.45% vs Llama 3.1 8B 1.55% and GPT-3.5 Turbo 3.68%)

## Limitations

- DeepSeek V3 was significantly slower in processing time and struggled with accuracy (higher MAE of 10.45%), so full re-generation for it is costly
- Findings are tied to the specific 10 evaluation inputs and the listed model versions; other inputs or model updates may shift failure modes
- Disclaimer audit depends on models voluntarily emitting a dietary disclaimer, which is not guaranteed or enforced by the framework

## Evidence
