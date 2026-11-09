---
name: usda-nutrition-reference-matching
description: Use when evaluating how accurately LLM-generated meal plans match a caloric
  target, and you need per-plan error quantified against USDA nutrition database reference
  values.
when_to_use_negative:
- The goal is measuring processing/latency time of model output generation rather
  than nutritional accuracy
- No USDA reference values are available for the meal items in the generated plans
- Plans were not generated through a reproducible framework with recorded failures,
  so '-' placeholders and replicate averaging cannot be applied
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3068
tools:
- name: NutriGen
  role: Framework that generates the personalized meal plans whose calories are validated
    against USDA references
  repo: github.com/SamanKhamesian/NutriGen
- name: pandas
  role: Sums per-item reported calories for each generated meal plan and assembles
    the error matrix
- name: GPT-3.5 Turbo
  role: Evaluated model achieving the lowest percentage error of 3.68% against caloric
    targets
- name: Llama 3.1 8B
  role: Evaluated model achieving the lowest percentage error of 1.55% against caloric
    targets
- name: DeepSeek V3
  role: Evaluated model used as an accuracy/latency baseline, exhibiting a higher
    MAE of 10.45%
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:f3366eeb136984502fe66a8753933f406f75d5f201c2ebe317772df459844bf2
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
  - http://edamontology.org/topic_3068
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# usda-nutrition-reference-matching

## Summary

Validates LLM-generated meal plans by summing per-item reported calories with pandas and comparing them against USDA FoodData Central reference values and user-defined caloric targets, yielding a percentage-error matrix across models and inputs.

## When to use

Use when evaluating how accurately LLM-generated meal plans match a caloric target, and you need per-plan error quantified against USDA nutrition database reference values.

## When NOT to use

- The goal is measuring processing/latency time of model output generation rather than nutritional accuracy
- No USDA reference values are available for the meal items in the generated plans
- Plans were not generated through a reproducible framework with recorded failures, so '-' placeholders and replicate averaging cannot be applied

## Inputs

- LLM-generated meal plans (10 simulated user inputs x models x 3 replicates) produced via the NutriGen Jupyter Notebook
- USDA FoodData Central reference calorie values for meal items appearing in the plans
- User-defined caloric target values for each of the 10 inputs

## Outputs

- 10x10 percentage-error matrix (models x inputs) with '-' placeholders for failed generations, written to table1_error_matrix.csv
- Per-model average percentage error against caloric targets (e.g., MAE)

## How to apply

For each simulated user input, prompt each LLM (GPT-3.5 Turbo, Llama 3.1 8B, DeepSeek V3, etc.) three times through the NutriGen framework to generate a personalized meal plan, recording any failure to produce a plan. Sum the per-item reported calories for every generated plan using pandas, then retrieve USDA FoodData Central reference calorie values for the meal items appearing in the plans. Compute the percentage error of each summed plan against the user-defined caloric target for its input, and average over the three replicates for every model-input pair. Assemble the results as a 10x10 error matrix (models x inputs), inserting '-' wherever a model failed to generate a plan, and write it to table1_error_matrix.csv. The best-performing models should reproduce the reported lowest percentage errors of 1.55% (Llama 3.1 8B) and 3.68% (GPT-3.5 Turbo), while weaker models such as DeepSeek V3 exhibit a higher MAE around 10.45%.

## Related tools

- **NutriGen** (Framework that generates the personalized meal plans whose calories are validated against USDA references) — github.com/SamanKhamesian/NutriGen
- **pandas** (Sums per-item reported calories for each generated meal plan and assembles the error matrix)
- **GPT-3.5 Turbo** (Evaluated model achieving the lowest percentage error of 3.68% against caloric targets)
- **Llama 3.1 8B** (Evaluated model achieving the lowest percentage error of 1.55% against caloric targets)
- **DeepSeek V3** (Evaluated model used as an accuracy/latency baseline, exhibiting a higher MAE of 10.45%)

## Examples

```
pip install -r requirements.txt  # then execute the NutriGen Jupyter Notebook with 'KEY' replaced by valid provider API keys
```

## Evaluation signals

- Verify the error matrix is exactly 10x10 (models x inputs) with '-' placeholders wherever a model failed to generate a plan
- Best-performing models reproduce the reported lowest percentage errors of 1.55% (Llama 3.1 8B) and 3.68% (GPT-3.5 Turbo)
- DeepSeek V3's average error should approximate the reported MAE of 10.45%
- Each model-input cell is the mean percentage error over three replicate generations
- Summed plan calories are cross-checked against USDA FoodData Central reference values before error computation

## Limitations

- Models can fail to produce a meal plan entirely; these cases are marked with '-' and contribute no error value
- DeepSeek V3 was significantly slower in processing time and less accurate (MAE 10.45%) than GPT-3.5 Turbo and Llama 3.1 8B, so latency must be budgeted
- Accuracy depends on LLMs incorporating reliable references like the USDA nutrition database via prompt engineering; without it, reported calories may deviate substantially

## Evidence
