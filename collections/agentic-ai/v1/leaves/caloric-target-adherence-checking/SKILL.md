---
name: caloric-target-adherence-checking
description: Use when evaluating a prompt-driven meal-plan generator's outputs against
  user-defined calorie/macronutrient targets, or when benchmarking multiple LLMs for
  dietary-adherence accuracy.
when_to_use_negative:
- The goal is generating meal plans rather than auditing their caloric adherence —
  use the prompt-rendering/generation skill instead
- Meal-plan outputs are unstructured free text lacking parseable portion sizes, recipes,
  and per-plan calorie totals, so no target comparison can be computed
- Evaluating models against macronutrient-only targets with no caloric target specified
  — the paper's error metrics are defined against user-defined caloric targets
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3063
- http://edamontology.org/topic_3379
tools:
- name: NutriGen
  role: Framework whose Jupyter Notebook pipeline generates the meal-plan completions
    and supports the calorie-target comparison
  repo: github.com/SamanKhamesian/NutriGen
- name: Llama 3.1 8B
  role: Best-performing model evaluated for caloric-target adherence (1.55% error)
  repo: github.com/SamanKhamesian/NutriGen
- name: GPT-3.5 Turbo
  role: Best-performing model evaluated for caloric-target adherence (3.68% error)
    via the openai client
  repo: github.com/SamanKhamesian/NutriGen
- name: DeepSeek V3
  role: Comparison model exhibiting a higher MAE of 10.45% and slower processing time,
    illustrating the accuracy/efficiency benchmarking use of this skill
- name: pandas
  role: Pinned dependency (pandas==2.2.3) used in the Python 3.10 environment for
    computing average total nutrition comparisons
  repo: github.com/SamanKhamesian/NutriGen
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:1e8b47626653a52f9b26c969cd54de0a32964c833ee5b84c07a28ba4969908f6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DeepSeek V3
  - GPT-3.5 Turbo
  - Llama 3.1 8B
  - NutriGen
  - Python
  - llamaapi
  - openai
  - pandas
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3063
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# caloric-target-adherence-checking

## Summary

Quantifies how closely LLM-generated meal plans match user-defined caloric targets by comparing average total nutrition (calories) per input against the specified target values, reporting percentage errors such as the 1.55% (Llama 3.1 8B) and 3.68% (GPT-3.5 Turbo) best results from NutriGen.

## When to use

Use when evaluating a prompt-driven meal-plan generator's outputs against user-defined calorie/macronutrient targets, or when benchmarking multiple LLMs for dietary-adherence accuracy.

## When NOT to use

- The goal is generating meal plans rather than auditing their caloric adherence — use the prompt-rendering/generation skill instead
- Meal-plan outputs are unstructured free text lacking parseable portion sizes, recipes, and per-plan calorie totals, so no target comparison can be computed
- Evaluating models against macronutrient-only targets with no caloric target specified — the paper's error metrics are defined against user-defined caloric targets

## Inputs

- Structured JSON meal-plan records (meal_plan.) with three meal plans, portion sizes, recipes, and per-plan total calories
- User-defined caloric target values per input (from prompt.txt constraints)
- Raw LLM completions from models such as GPT-3.5 Turbo, Llama 3.1 8B, or DeepSeek V3

## Outputs

- Percentage error (or MAE) of average total calories vs. user-defined target per model
- Comparison of average total nutrition against target values (USDA vs target analysis)
- Total processing time to generate 10 outputs per model

## How to apply

After parsing the LLM completion into a structured JSON record holding the three meal plans with portion sizes, recipes, and per-plan total calories, compute the average total nutrition (calories) for each input and compare it against the specified target values. Express adherence as a percentage error; in NutriGen the best models (Llama 3.1 8B and GPT-3.5 Turbo) achieved errors of 1.55% and 3.68%, so deviations within that band signal correct application. Also measure total processing time required to generate 10 outputs per model, since efficiency (e.g. DeepSeek V3's slowness and 10.45% MAE) is part of the evaluation. Validation gate: meal_plan. must contain exactly three meal plans with portion sizes and recipes, and its total calories should deviate from the caloric target by no more than the paper's best reported errors.

## Related tools

- **NutriGen** (Framework whose Jupyter Notebook pipeline generates the meal-plan completions and supports the calorie-target comparison) — github.com/SamanKhamesian/NutriGen
- **Llama 3.1 8B** (Best-performing model evaluated for caloric-target adherence (1.55% error)) — github.com/SamanKhamesian/NutriGen
- **GPT-3.5 Turbo** (Best-performing model evaluated for caloric-target adherence (3.68% error) via the openai client) — github.com/SamanKhamesian/NutriGen
- **DeepSeek V3** (Comparison model exhibiting a higher MAE of 10.45% and slower processing time, illustrating the accuracy/efficiency benchmarking use of this skill)
- **pandas** (Pinned dependency (pandas==2.2.3) used in the Python 3.10 environment for computing average total nutrition comparisons) — github.com/SamanKhamesian/NutriGen

## Examples

```
jupyter notebook NutriGen.ipynb  # replace "KEY" with a valid API key, then compare average total calories per input against the specified target values
```

## Evaluation signals

- meal_plan. parses as valid JSON and contains exactly three meal plans, each with portion sizes and recipes
- Total calories of the generated plan deviate from the user-defined caloric target by no more than the paper's best reported errors (1.55% for Llama 3.1 8B, 3.68% for GPT-3.5 Turbo)
- Average total nutrition per input is compared against the specified target values, with deviations quantified as percentage error/MAE
- Processing time is measured over a consistent workload: total time to generate 10 outputs per model, enabling cross-model efficiency comparison
- Nutritional values are grounded in a reliable reference (USDA nutrition database) rather than unverified model recall

## Limitations

- DeepSeek V3 struggled with accuracy (MAE of 10.45%) and was significantly slower than GPT-3.5 Turbo and Llama 3.1 8B, so adherence quality is highly model-dependent
- Validation thresholds are tied to the paper's specific best-reported errors; other models or targets may fall outside the 1.55%/3.68% band without indicating a broken pipeline
- A valid API key must replace the "KEY" placeholder in the notebook before execution, otherwise no completions are produced to evaluate

## Evidence

- [abstract] Llama 3.1 8B and GPT-3.5 Turbo achieve the lowest percentage errors of 1.55% and 3.68%, respectively, producing meal plans that closely align with user-defined caloric targets: "Llama 3.1 8B and GPT-3.5 Turbo achieve the lowest percentage errors of 1.55% and 3.68%, respectively"
- [methods] Comparison of average total nutrition against targets: "comparing the average total nutrition for each input against the specified target values"
- [methods] Processing-time measurement protocol: "we measured the total processing time required to generate 10 outputs"
- [abstract] DeepSeek V3 failure mode: "it struggled with accuracy, exhibiting a higher MAE of 10.45%, and was significantly slower in processing time compared to models like GPT-3.5 Turbo and Llama 3.1 8B"
