---
name: llm-meal-plan-generation
description: Use when the task is to generate personalized meal plans from dietary
  profiles via LLM prompting and quantify adherence to user-defined caloric targets
  or USDA nutrition references.
when_to_use_negative:
- The task requires verified clinical dietary advice or regulated nutrition counseling
  — NutriGen outputs are LLM-generated plans evaluated for accuracy, not clinical
  recommendations
- No LLM API keys are available; the framework cannot execute without replacing the
  "KEY" placeholder
- The goal is general nutritional-database lookup rather than personalized plan generation
  — direct USDA FoodData Central queries are more appropriate
edam_operation: http://edamontology.org/operation_3946
tools:
- name: NutriGen
  role: Framework whose Jupyter Notebook orchestrates prompt composition and meal
    plan generation
  repo: github.com/SamanKhamesian/NutriGen
- name: openai
  role: SDK client for calling GPT-3.5 Turbo to generate meal plan completions
  repo: github.com/SamanKhamesian/NutriGen
- name: llamaapi
  role: SDK client for calling Llama 3.1 8B to generate meal plan completions
  repo: github.com/SamanKhamesian/NutriGen
- name: pandas
  role: Sum per-item reported calories and assemble MAE/error-matrix tables
  repo: github.com/SamanKhamesian/NutriGen
- name: USDA nutrition database
  role: Reliable nutritional reference grounding the prompt's calorie values and validation
    of generated plans
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_001
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:bfaad3e8cd00b28b300438f821cb1c93e830c6098a2c7ca8837d0ac6e85860c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - DeepSeek V3
  - GPT-3.5 Turbo
  - Llama 3.1 8B
  - NutriGen
  - Python
  - USDA nutrition database
  - anthropic
  - google.generativeai
  - llamaapi
  - numpy
  - openai
  - pandas
  edam_operation: http://edamontology.org/operation_3946
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# llm-meal-plan-generation

## Summary

Generate personalized meal plans by prompting LLMs (GPT-3.5 Turbo, Llama 3.1 8B, DeepSeek V3, etc.) through the NutriGen framework, grounding nutritional values in a USDA-nutrition-database-referenced prompt with user-defined calorie, protein, and sugar constraints, then evaluating the plans against caloric targets (MAE %) and USDA reference values.

## When to use

Use when the task is to generate personalized meal plans from dietary profiles via LLM prompting and quantify adherence to user-defined caloric targets or USDA nutrition references.

## When NOT to use

- The task requires verified clinical dietary advice or regulated nutrition counseling — NutriGen outputs are LLM-generated plans evaluated for accuracy, not clinical recommendations
- No LLM API keys are available; the framework cannot execute without replacing the "KEY" placeholder
- The goal is general nutritional-database lookup rather than personalized plan generation — direct USDA FoodData Central queries are more appropriate

## Inputs

- Simulated dietary user profiles (10 inputs) with user-defined caloric and macronutrient targets
- NutriGen prompt template (Jupyter Notebook) with "KEY" placeholder
- Provider API keys for openai, anthropic, google.generativeai, llamaapi
- USDA FoodData Central reference calorie values for meal items

## Outputs

- Generated meal plans (three per input per model) with portion sizes, recipes, and per-plan total calories (e.g., meal_plan.)
- Rendered prompt text (prompt.txt)
- table_ii_mae_results.csv: per-model MAE and MAE % against caloric targets
- table1_error_matrix.csv: 10x10 models-x-inputs percentage-error matrix with '-' placeholders for failed generations

## How to apply

Set up Python 3.10 with the pinned dependencies (anthropic==0.45.2, google.generativeai==0.8.4, llamaapi==0.1.36, openai==1.42.0, numpy==2.2.2, pandas==2.2.3) from the SamanKhamesian/NutriGen repository, replacing the "KEY" placeholder in the Jupyter Notebook with a valid provider API key. Compose the structured prompt P = {I_current, T_task, O_output}: concatenate the user profile (current input), a task description embedding calorie/macronutrient/portion-size constraints, and an output indicator requesting meal plans with portion sizes, recipes, and nutritional values grounded in the USDA nutrition database. Send the rendered prompt to each of the evaluated models (e.g., GPT-3.5 Turbo via the openai client, Llama 3.1 8B via llamaapi, DeepSeek V3) three times per input, recording any failures to produce a plan. Parse completions into structured records, sum per-item reported calories with pandas, compute MAE % against the user-defined caloric target per Equation 2, and compare averaged totals against USDA FoodData Central reference values. Expect the best models (Llama 3.1 8B at 1.55%, GPT-3.5 Turbo at 3.68% MAE) to validate the pipeline; DeepSeek V3 should reproduce ~10.45% MAE and markedly slower processing times.

## Related tools

- **NutriGen** (Framework whose Jupyter Notebook orchestrates prompt composition and meal plan generation) — github.com/SamanKhamesian/NutriGen
- **openai** (SDK client for calling GPT-3.5 Turbo to generate meal plan completions) — github.com/SamanKhamesian/NutriGen
- **llamaapi** (SDK client for calling Llama 3.1 8B to generate meal plan completions) — github.com/SamanKhamesian/NutriGen
- **pandas** (Sum per-item reported calories and assemble MAE/error-matrix tables) — github.com/SamanKhamesian/NutriGen
- **USDA nutrition database** (Reliable nutritional reference grounding the prompt's calorie values and validation of generated plans)

## Examples

```
pip install -r requirements.txt && jupyter notebook NutriGen.ipynb  # replace "KEY" with a valid provider API key before execution
```

## Evaluation signals

- Reproduced MAE % aligns with reported figures: Llama 3.1 8B at 1.55%, GPT-3.5 Turbo at 3.68%, DeepSeek V3 at 10.45%, within tolerance
- Error matrix is 10x10 (models x inputs) with '-' placeholders wherever a model failed to generate a plan
- Each structured meal-plan record contains exactly three meal plans with portion sizes and recipes
- table_ii_mae_results.csv contains a row with MAE and MAE % for each of the 10 evaluated models
- Smaller/optimized models (GPT-3.5 Turbo, Gemini 2.0 Flash, Claude 3.5 Haiku) complete 10 generations fastest, consistent with the reported processing-time comparison

## Limitations

- DeepSeek V3 struggled with accuracy (MAE of 10.45%) and was significantly slower than GPT-3.5 Turbo and Llama 3.1 8B
- Models may fail to produce a plan for some inputs; these failures must be recorded (as '-' in the error matrix) rather than treated as zero error
- Errors are measured against user-defined caloric targets and USDA references; macronutrient or micronutrient fidelity beyond calories is not the primary metric

## Evidence
