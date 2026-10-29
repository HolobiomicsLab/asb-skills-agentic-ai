---
name: meal-plan-calorie-aggregation
description: Use when you have LLM-generated meal plans for multiple user inputs and
  need to quantify how well each model's average total calories match user-defined
  caloric targets against USDA reference values.
when_to_use_negative:
- The analysis goal is model processing-time comparison rather than caloric accuracy
  — use the separate time-per-model evaluation instead.
- Only a single model or a single user input is available, so no models-x-inputs error
  matrix can be assembled.
- Meal items lack USDA FoodData Central reference entries, making percentage error
  against USDA reference values impossible to compute.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: NutriGen
  role: Framework that generates the personalized meal plans whose total calories
    are aggregated and scored
  repo: https://github.com/SamanKhamesian/NutriGen
- name: pandas
  role: Sums per-item reported calories for every generated meal plan and assembles
    the error matrix
- name: GPT-3.5 Turbo
  role: Evaluated LLM; reference best-case accuracy of 3.68% percentage error
- name: Llama 3.1 8B
  role: Evaluated LLM; reference best-case accuracy of 1.55% percentage error
- name: DeepSeek V3
  role: Evaluated LLM; reference worst-case MAE of 10.45% used as a contrast when
    validating the matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:8733b549ffcce1b3236116a65992c3fa7ac7589d4229dcc9f902855cb70c5b88
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SamanKhamesian/NutriGen
  tools:
  - DeepSeek V3
  - GPT-3.5 Turbo
  - Llama 3.1 8B
  - NutriGen
  - Python
  - anthropic
  - google.generativeai
  - llamaapi
  - openai
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# meal-plan-calorie-aggregation

## Summary

Aggregate per-item reported calories from LLM-generated personalized meal plans (NutriGen), compare the summed totals against user-defined caloric targets using USDA nutrition database reference values, and compute per-model, per-input percentage errors such as the 1.55% and 3.68% reported for Llama 3.1 8B and GPT-3.5 Turbo.

## When to use

Use when you have LLM-generated meal plans for multiple user inputs and need to quantify how well each model's average total calories match user-defined caloric targets against USDA reference values.

## When NOT to use

- The analysis goal is model processing-time comparison rather than caloric accuracy — use the separate time-per-model evaluation instead.
- Only a single model or a single user input is available, so no models-x-inputs error matrix can be assembled.
- Meal items lack USDA FoodData Central reference entries, making percentage error against USDA reference values impossible to compute.

## Inputs

- LLM-generated personalized meal plans (three replicates per model per user input) from the NutriGen framework
- 10 simulated user inputs with user-defined caloric targets
- USDA FoodData Central reference calorie values for the meal items appearing in the plans

## Outputs

- Per-model, per-input average percentage error of total calories versus the caloric target
- 10x10 error matrix (models x inputs) with '-' placeholders for failed generations, written to table1_error_matrix.csv
- USDA-vs-target comparison of average total calories per input per model (as in figures/usda_vs_target.png)

## How to apply

For each of the 10 simulated user inputs, prompt each LLM (e.g., GPT-3.5 Turbo, Llama 3.1 8B, DeepSeek V3) three times through the NutriGen framework to generate a personalized meal plan, recording any failure to produce a plan. Sum the per-item reported calories of every generated plan with pandas, then retrieve USDA FoodData Central reference calorie values for the meal items appearing in the plans. Compute the percentage error of each summed plan against its user-defined caloric target and average over the three replicates for every model-input pair. Assemble the 10x10 error matrix (models x inputs), inserting '-' wherever a model failed to generate a plan; the best-performing models should reproduce the reported lowest percentage errors of 1.55% (Llama 3.1 8B) and 3.68% (GPT-3.5 Turbo), with DeepSeek V3 showing a higher MAE of 10.45%.

## Related tools

- **NutriGen** (Framework that generates the personalized meal plans whose total calories are aggregated and scored) — https://github.com/SamanKhamesian/NutriGen
- **pandas** (Sums per-item reported calories for every generated meal plan and assembles the error matrix)
- **GPT-3.5 Turbo** (Evaluated LLM; reference best-case accuracy of 3.68% percentage error)
- **Llama 3.1 8B** (Evaluated LLM; reference best-case accuracy of 1.55% percentage error)
- **DeepSeek V3** (Evaluated LLM; reference worst-case MAE of 10.45% used as a contrast when validating the matrix)

## Examples

```
pip install -r requirements.txt && jupyter notebook NutriGen.ipynb  # with "KEY" replaced by a valid provider API key
```

## Evaluation signals

- The assembled error matrix is exactly 10x10 (models x inputs).
- Failed generations appear as '-' placeholders rather than numeric errors or zeros.
- Best-performing models reproduce the reported lowest percentage errors of 1.55% and 3.68%.
- DeepSeek V3 errors are markedly higher (around its reported 10.45% MAE) than GPT-3.5 Turbo and Llama 3.1 8B.
- Each model-input error is an average over three replicate meal-plan generations.

## Limitations

- Some models failed to produce a plan for certain inputs; these cells must be marked with a dash rather than an error value.
- DeepSeek V3 struggled with accuracy (MAE 10.45%) and was significantly slower than GPT-3.5 Turbo and Llama 3.1 8B, so runtime and retries may be an issue.
- Accuracy is measured against user-defined caloric targets with USDA reference values; it does not assess micronutrients or other dietary dimensions.

## Evidence
