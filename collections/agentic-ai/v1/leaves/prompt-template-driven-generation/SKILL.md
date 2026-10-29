---
name: prompt-template-driven-generation
description: Use when the goal is to elicit structured, constraint-adherent outputs
  (e.g., meal plans matching a specified calorie target) from one or more LLMs via
  a shared prompt template across provider SDKs.
when_to_use_negative:
- When the task requires exact, verifiable nutrient computation from a food composition
  database rather than LLM-generated estimates — LLM outputs need MAE validation because
  adherence varies by model (1.55% to 10.45%).
- When low-latency or high-accuracy calorie targeting is critical and only slow, high-error
  models (e.g., DeepSeek V3 at 10.45% MAE, significantly slower processing) are available.
- When dietary recommendations must account for real-world constraints such as ingredient
  availability and minimal user input that the underlying LLM prompt does not encode.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3071
tools:
- name: NutriGen
  role: LLM-based framework whose Jupyter Notebook holds the prompt template and dietary
    profiles used to generate personalized meal plans
  repo: https://github.com/SamanKhamesian/NutriGen
- name: openai
  role: Python SDK client (v1.42.0) used to call GPT-3.5 Turbo and other OpenAI-compatible
    models with the filled prompt template
- name: anthropic
  role: Python SDK (v0.45.2) used to call Claude models (e.g., Claude 3.5 Haiku) with
    the prompt template
- name: google.generativeai
  role: Python SDK (v0.8.4) used to call Gemini models (e.g., Gemini 2.0 Flash) with
    the prompt template
- name: llamaapi
  role: Python SDK (v0.1.36) used to call Llama models (e.g., Llama 3.1 8B) with the
    prompt template
- name: numpy
  role: Numerical computation of average total calories and MAE / MAE % between generated
    plans and caloric targets
- name: pandas
  role: Assembling per-model MAE and MAE % results into the Table II mirror and writing
    table_ii_mae_results.csv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:56f1fe337afbb68b4a095d505d6495a0778bea9467e653a7372d8cc2c6554c8c
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
  - numpy
  - openai
  - pandas
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# prompt-template-driven-generation

## Summary

Generates personalized meal plans by filling a prompt template with user-defined dietary preferences, caloric targets, and nutrition-database context, then querying LLMs (e.g., GPT-3.5 Turbo, Llama 3.1 8B) via their SDK clients to produce structured plans whose total calories can be validated against targets.

## When to use

Use when the goal is to elicit structured, constraint-adherent outputs (e.g., meal plans matching a specified calorie target) from one or more LLMs via a shared prompt template across provider SDKs.

## When NOT to use

- When the task requires exact, verifiable nutrient computation from a food composition database rather than LLM-generated estimates — LLM outputs need MAE validation because adherence varies by model (1.55% to 10.45%).
- When low-latency or high-accuracy calorie targeting is critical and only slow, high-error models (e.g., DeepSeek V3 at 10.45% MAE, significantly slower processing) are available.
- When dietary recommendations must account for real-world constraints such as ingredient availability and minimal user input that the underlying LLM prompt does not encode.

## Inputs

- Simulated dietary profiles (user-defined dietary preferences and constraints, including target caloric values)
- Prompt template from the NutriGen Jupyter Notebook (with "KEY" placeholder for API keys)
- Personalized nutrition database grounded in USDA nutrition database references
- requirements.txt pinning Python 3.10 dependencies

## Outputs

- Structured meal plans generated per model per input profile (three plans per profile in the evaluation)
- Average total calories per input per model for comparison against targets
- Per-model MAE and MAE % values (e.g., table_ii_mae_results.csv mirroring Table II)
- Processing-time measurements for 10 outputs per model

## How to apply

Install the Python 3.10 dependencies (openai==1.42.0, anthropic==0.45.2, google.generativeai==0.8.4, llamaapi==0.1.36, numpy, pandas) via 'pip install -r requirements.txt'. Load the simulated dietary profiles and the prompt template from the NutriGen Jupyter Notebook in github.com/SamanKhamesian/NutriGen, replacing the "KEY" placeholder with a valid provider API key before execution. Build the prompt by grounding it in a personalized nutrition database (e.g., USDA nutrition database references) and the user's dietary preferences and caloric target, then call each model through its SDK client to generate multiple plans per input (three per profile per model in the evaluation; 10 outputs per model for timing). After generation, compute the average total calories per input per model and compare against the specified target to quantify adherence as MAE % via Equation 2, since the article reports Llama 3.1 8B at 1.55%, GPT-3.5 Turbo at 3.68%, and DeepSeek V3 at 10.45%. The rationale is that prompt engineering plus a nutrition database lets LLMs incorporate reliable nutritional references while remaining flexible.

## Related tools

- **NutriGen** (LLM-based framework whose Jupyter Notebook holds the prompt template and dietary profiles used to generate personalized meal plans) — https://github.com/SamanKhamesian/NutriGen
- **openai** (Python SDK client (v1.42.0) used to call GPT-3.5 Turbo and other OpenAI-compatible models with the filled prompt template)
- **anthropic** (Python SDK (v0.45.2) used to call Claude models (e.g., Claude 3.5 Haiku) with the prompt template)
- **google.generativeai** (Python SDK (v0.8.4) used to call Gemini models (e.g., Gemini 2.0 Flash) with the prompt template)
- **llamaapi** (Python SDK (v0.1.36) used to call Llama models (e.g., Llama 3.1 8B) with the prompt template)
- **numpy** (Numerical computation of average total calories and MAE / MAE % between generated plans and caloric targets)
- **pandas** (Assembling per-model MAE and MAE % results into the Table II mirror and writing table_ii_mae_results.csv)

## Examples

```
pip install -r requirements.txt && jupyter notebook  # then replace "KEY" with a valid API key and execute the NutriGen notebook to generate meal plans
```

## Evaluation signals

- Reproduced MAE % per model aligns with reported figures within tolerance: Llama 3.1 8B ≈ 1.55%, GPT-3.5 Turbo ≈ 3.68%, DeepSeek V3 ≈ 10.45%.
- table_ii_mae_results.csv contains an MAE and MAE % row for each of the 10 evaluated models.
- Average total calories per input per model tracks the user-specified caloric targets, as visualized in the usda_vs_target comparison figure.
- Prompt template executed only after "KEY" placeholders are replaced with valid API keys; runs complete without authentication errors across all SDK clients.
- Each model produces the expected number of outputs (three meal plans per input profile; 10 outputs per model for the timing measurement).

## Limitations

- Accuracy varies sharply across models: DeepSeek V3 exhibited a higher MAE of 10.45% and was significantly slower in processing time than GPT-3.5 Turbo and Llama 3.1 8B.
- LLM-generated calorie totals are estimates; they must be compared against specified target values rather than assumed correct.
- Valid API keys are required for each provider; the code will not run with the "KEY" placeholder in place.
- Existing dietary recommendation approaches (which this method addresses) can lack adaptability, ignore ingredient availability, and require extensive user input — constraints the prompt template must encode explicitly.

## Evidence
