---
name: dietary-prompt-template-construction
description: Use when generating LLM personalized meal plans that must honor explicit
  user constraints (calorie targets, macronutrient distributions, portion sizes) and
  be validated against a defined caloric target.
when_to_use_negative:
- The task requires true database retrieval or exact nutritional computation rather
  than LLM-generated estimates grounded in a referenced database (USDA values are
  referenced, not queried)
- DeepSeek V3 or other slower/less accurate models are the only available backends
  and strict calorie-target adherence is required
- The user cannot provide explicit numeric constraints (calories, macros), since the
  prompt mechanism encodes them as explicit constraints
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: NutriGen
  role: Framework whose prompt-engineering mechanism defines the modular prompt template
    and personalized nutrition database grounding
  repo: https://github.com/SamanKhamesian/NutriGen
- name: openai
  role: Python client (pinned at 1.42.0) used to send the rendered prompt to GPT-3.5
    Turbo
  repo: https://github.com/SamanKhamesian/NutriGen
- name: llamaapi
  role: Python client (pinned at 0.1.36) used to send the rendered prompt to Llama
    3.1 8B
  repo: https://github.com/SamanKhamesian/NutriGen
- name: GPT-3.5 Turbo
  role: Top-performing backend model; best reported calorie error 3.68%
- name: Llama 3.1 8B
  role: Top-performing backend model; best reported calorie error 1.55%
- name: USDA nutrition database
  role: Reliable nutritional reference incorporated into the prompt to ground meal
    plan nutritional values
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:e4664eeba62cb4bc949cd9d70c15191df12ad6d847a90c755c4c279a5f9f2914
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/SamanKhamesian/NutriGen
  tools:
  - GPT-3.5 Turbo
  - Llama 3.1 8B
  - NutriGen
  - Python
  - USDA nutrition database
  - llamaapi
  - openai
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# dietary-prompt-template-construction

## Summary

Constructs a modular structured prompt P = {I_current, T_task, O_output} for LLM-based personalized meal plan generation, encoding user calorie, macronutrient, and sugar constraints with an output indicator requesting meal plans with portion sizes, recipes, and USDA-grounded nutritional values.

## When to use

Use when generating LLM personalized meal plans that must honor explicit user constraints (calorie targets, macronutrient distributions, portion sizes) and be validated against a defined caloric target.

## When NOT to use

- The task requires true database retrieval or exact nutritional computation rather than LLM-generated estimates grounded in a referenced database (USDA values are referenced, not queried)
- DeepSeek V3 or other slower/less accurate models are the only available backends and strict calorie-target adherence is required
- The user cannot provide explicit numeric constraints (calories, macros), since the prompt mechanism encodes them as explicit constraints

## Inputs

- Sample user profile (I_current): dietary preferences, restrictions, and nutritional targets
- Task description (T_task) embedding calorie, protein, and sugar constraints
- Output indicator (O_output) specifying three meal plans with portion sizes and recipes

## Outputs

- prompt.txt — the rendered structured prompt text
- meal_plan — structured JSON record with three meal plans, portion sizes, recipes, and per-plan total calories

## How to apply

Compose the prompt by concatenating three modules: (1) the current input — the sample user profile; (2) a task description embedding the user's calorie, protein, and sugar constraints as explicit limits; and (3) an output indicator requesting three meal plans with portion sizes and recipes, with nutritional values grounded in the USDA nutrition database. Save the rendered text as prompt.txt, then send it to a top-performing model (GPT-3.5 Turbo via the openai client, or Llama 3.1 8B via llamaapi) — the paper reports these achieve the lowest percentage errors of 3.68% and 1.55% against caloric targets, versus 10.45% MAE for DeepSeek V3. Parse the raw completion into a structured JSON record (meal_plan) holding the three plans with per-plan total calories, then validate calorie deviation against the user-defined target.

## Related tools

- **NutriGen** (Framework whose prompt-engineering mechanism defines the modular prompt template and personalized nutrition database grounding) — https://github.com/SamanKhamesian/NutriGen
- **openai** (Python client (pinned at 1.42.0) used to send the rendered prompt to GPT-3.5 Turbo) — https://github.com/SamanKhamesian/NutriGen
- **llamaapi** (Python client (pinned at 0.1.36) used to send the rendered prompt to Llama 3.1 8B) — https://github.com/SamanKhamesian/NutriGen
- **GPT-3.5 Turbo** (Top-performing backend model; best reported calorie error 3.68%)
- **Llama 3.1 8B** (Top-performing backend model; best reported calorie error 1.55%)
- **USDA nutrition database** (Reliable nutritional reference incorporated into the prompt to ground meal plan nutritional values)

## Examples

```
pip install -r requirements.txt  # then run the NutriGen notebook with 'KEY' replaced by a valid API key to render prompt.txt and capture meal_plan JSON
```

## Evaluation signals

- meal_plan must contain exactly three meal plans, each with portion sizes and recipes
- Total calories per plan must deviate from the user-defined caloric target by no more than the paper's best reported errors (≤1.55% for Llama 3.1 8B, ≤3.68% for GPT-3.5 Turbo)
- prompt.txt must contain all three modules: current input (user profile), task description with explicit calorie/protein/sugar constraints, and output indicator
- Generated plans should be structured and practical, referencing USDA nutrition-database-grounded values
- Model choice matters: expect substantially larger errors (~10.45% MAE) and slower processing with DeepSeek V3

## Limitations

- Accuracy is model-dependent: DeepSeek V3 exhibited a higher MAE of 10.45% and was significantly slower than GPT-3.5 Turbo and Llama 3.1 8B
- A valid LLM API key must replace the 'KEY' placeholder before execution; the pipeline requires Python 3.10 with pinned dependencies (anthropic 0.45.2, google.generativeai 0.8.4, llamaapi 0.1.36, openai 1.42.0, numpy 2.2.2, pandas 2.2.3)
- Nutritional values are LLM-generated and grounded by reference to the USDA database rather than retrieved from it, so outputs are approximations

## Evidence
