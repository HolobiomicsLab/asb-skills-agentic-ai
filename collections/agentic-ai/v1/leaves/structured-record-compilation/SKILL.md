---
name: structured-record-compilation
description: Use when you have per-model meal plan outputs across multiple evaluation
  inputs and need to flag outright generation failures, structural defects, and disclaimer
  presence in a tabular record.
when_to_use_negative:
- The analysis goal is quantitative accuracy (MAE against caloric targets) rather
  than structural/qualitative output auditing — use the paper's USDA-vs-target comparison
  instead.
- Only one model's outputs are available and no cross-model failure matrix is needed
  — the compilation is designed for multi-model, multi-input comparison.
- Outputs have not yet been generated (notebook not run or API keys missing) — there
  is nothing to audit or compile.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3379
- http://edamontology.org/topic_3071
tools:
- name: NutriGen
  role: Framework whose Jupyter Notebook generates the meal plan outputs that are
    audited and compiled into the structured record
  repo: https://github.com/SamanKhamesian/NutriGen
- name: pandas
  role: Used in Python 3.10 to build the failure/disclaimer table and save it as qualitative_failures.csv
- name: GPT-3.5 Turbo
  role: One of the six models whose outputs are inspected for failure modes and disclaimer
    presence
- name: Claude 3.5 Haiku
  role: Model whose outputs are inspected; failed on input 5 and produced single plans
    on inputs 6/8/9/10
- name: DeepSeek V3
  role: Model whose outputs are inspected; failed to generate plans for inputs 4–7
- name: Llama 3.1 8B
  role: Model whose outputs are inspected; failed on input 8
- name: Gemini 1.5 Pro
  role: Model whose outputs are inspected; omitted fat and protein values from all
    plans
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:21e37d730176071390af6b58931aaf2353482fa4f3524ca21b396e9460b40367
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3379
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# structured-record-compilation

## Summary

A method for auditing LLM-generated meal plan outputs (from the NutriGen framework) for qualitative failures and compiling them into a structured pandas table (qualitative_failures.csv) with per-input failure types and dietary-disclaimer flags.

## When to use

Use when you have per-model meal plan outputs across multiple evaluation inputs and need to flag outright generation failures, structural defects, and disclaimer presence in a tabular record.

## When NOT to use

- The analysis goal is quantitative accuracy (MAE against caloric targets) rather than structural/qualitative output auditing — use the paper's USDA-vs-target comparison instead.
- Only one model's outputs are available and no cross-model failure matrix is needed — the compilation is designed for multi-model, multi-input comparison.
- Outputs have not yet been generated (notebook not run or API keys missing) — there is nothing to audit or compile.

## Inputs

- Meal plan outputs per model (6 LLMs) for 10 evaluation inputs, produced by the NutriGen Jupyter Notebook
- Model list: GPT-3.5 Turbo, Claude 3.5 Haiku, DeepSeek V3, Llama 3.1 8B, Gemini 1.5 Pro, Gemini 2.0 Flash
- NutriGen Jupyter Notebook with valid API keys (openai==1.42.0, anthropic==0.45.2, google.generativeai==0.8.4, llamaapi==0.1.36)

## Outputs

- qualitative_failures.csv — structured table with columns for model, input index, failure type, and disclaimer presence
- Per-model disclaimer flags (yes/no) across the 10 inputs

## How to apply

Re-run the NutriGen Jupyter Notebook (replacing "KEY" with a valid API key) to obtain meal plans from each model — GPT-3.5 Turbo, Claude 3.5 Haiku, DeepSeek V3, Llama 3.1 8B, Gemini 1.5 Pro, Gemini 2.0 Flash — across the 10 evaluation inputs. Inspect each model-input pair for outright failures (e.g., DeepSeek V3 on inputs 4–7, Claude 3.5 Haiku on input 5, Llama 3.1 8B on input 8) and structural defects (Claude 3.5 Haiku producing only one plan on inputs 6/8/9/10; Gemini 1.5 Pro omitting fat and protein values from all plans). Separately audit each output for the presence of a dietary disclaimer and record it per model. Compile all entries into a pandas (pandas==2.2.3, Python 3.10) DataFrame with columns model, input index, failure type, and disclaimer presence, then save as qualitative_failures.csv. Validate the record against the paper's accuracy context — DeepSeek V3's 10.45% MAE versus Llama 3.1 8B (1.55%) and GPT-3.5 Turbo (3.68%) — to ensure failure-mode entries are consistent.

## Related tools

- **NutriGen** (Framework whose Jupyter Notebook generates the meal plan outputs that are audited and compiled into the structured record) — https://github.com/SamanKhamesian/NutriGen
- **pandas** (Used in Python 3.10 to build the failure/disclaimer table and save it as qualitative_failures.csv)
- **GPT-3.5 Turbo** (One of the six models whose outputs are inspected for failure modes and disclaimer presence)
- **Claude 3.5 Haiku** (Model whose outputs are inspected; failed on input 5 and produced single plans on inputs 6/8/9/10)
- **DeepSeek V3** (Model whose outputs are inspected; failed to generate plans for inputs 4–7)
- **Llama 3.1 8B** (Model whose outputs are inspected; failed on input 8)
- **Gemini 1.5 Pro** (Model whose outputs are inspected; omitted fat and protein values from all plans)

## Examples

```
df.to_csv('qualitative_failures.csv', index=False)  # after auditing model outputs across the 10 inputs
```

## Evaluation signals

- Record enumerates DeepSeek V3 failures exactly at inputs 4, 5, 6, 7 and Claude 3.5 Haiku failure at input 5.
- Record notes Claude 3.5 Haiku single-plan outputs on inputs 6, 8, 9, 10 and Gemini 1.5 Pro missing fat and protein values on all plans.
- Llama 3.1 8B failure on input 8 is present in the table.
- Every model-input pair carries a disclaimer presence flag (yes/no), with no missing entries across 6 models × 10 inputs.
- qualitative_failures.csv is produced with columns for model, input index, failure type, and disclaimer presence.

## Limitations

- DeepSeek V3 failed outright on 4 of 10 inputs, so its rows cannot support accuracy or structural comparisons for those inputs.
- Gemini 1.5 Pro's plans lack fat and protein values, limiting macro-level structural checks.
- Some models (Claude 3.5 Haiku) returned fewer plans than requested on certain inputs, so record completeness depends on detecting this defect.
- Disclaimer presence is a binary qualitative flag and must be re-audited if prompts or models change.

## Evidence
