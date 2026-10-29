---
name: llm-output-failure-mode-cataloging
description: Use when comparing multiple LLMs (e.g., GPT-3.5 Turbo, Claude 3.5 Haiku,
  DeepSeek V3, Gemini 1.5 Pro) on structured generation tasks and you need to catalog
  non-numeric failure modes that accuracy metrics like MAE miss.
when_to_use_negative:
- When the analysis goal is purely quantitative caloric-target error (percentage MAE
  against user-defined caloric targets) — use the accuracy-comparison workflow instead.
- When only a single model's outputs are available with no per-input, per-model comparison
  matrix — the catalog requires cross-model, per-input inspection.
- When outputs are not structured meal plans (e.g., free-form text tasks) — the failure
  taxonomy (missing plans, single-plan outputs, omitted fat/protein values, disclaimer
  presence) is specific to structured nutritional plan generation.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3393
tools:
- name: NutriGen
  role: Framework whose Jupyter Notebook produces the per-model, per-input meal plan
    outputs to be audited for failure modes
  repo: github.com/SamanKhamesian/NutriGen
- name: Claude 3.5 Haiku
  role: Evaluated model whose outputs exhibit failure on input 5 and single-plan outputs
    on inputs 6/8/9/10
- name: DeepSeek V3
  role: Evaluated model whose outputs failed on inputs 4–7 and showed higher MAE of
    10.45%
- name: Llama 3.1 8B
  role: Evaluated model whose outputs failed on input 8; achieved lowest error of
    1.55%
- name: GPT-3.5 Turbo
  role: Evaluated model (lowest error of 3.68%) used as a behavioral reference in
    the failure matrix
- name: Gemini 1.5 Pro
  role: Evaluated model that omitted fat and protein values from all plans
- name: pandas
  role: Used to compile failure entries and disclaimer flags into the structured qualitative_failures.csv
    table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:4f6119ceabc6ff5dba8706ac73edb28b48350552290665d0cba949ccd3eb7554
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3393
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# llm-output-failure-mode-cataloging

## Summary

A qualitative auditing procedure that inspects each LLM's generated meal plans (from the NutriGen Jupyter Notebook across 10 evaluation inputs) for structural failures—missing plans, single-plan outputs, omitted nutrient values—and records disclaimer presence per model, complementing accuracy metrics like MAE.

## When to use

Use when comparing multiple LLMs (e.g., GPT-3.5 Turbo, Claude 3.5 Haiku, DeepSeek V3, Gemini 1.5 Pro) on structured generation tasks and you need to catalog non-numeric failure modes that accuracy metrics like MAE miss.

## When NOT to use

- When the analysis goal is purely quantitative caloric-target error (percentage MAE against user-defined caloric targets) — use the accuracy-comparison workflow instead.
- When only a single model's outputs are available with no per-input, per-model comparison matrix — the catalog requires cross-model, per-input inspection.
- When outputs are not structured meal plans (e.g., free-form text tasks) — the failure taxonomy (missing plans, single-plan outputs, omitted fat/protein values, disclaimer presence) is specific to structured nutritional plan generation.

## Inputs

- Meal plan outputs per model (GPT-3.5 Turbo, Claude 3.5 Haiku, DeepSeek V3, Llama 3.1 8B, Gemini 1.5 Pro, Gemini 2.0 Flash) from the NutriGen Jupyter Notebook for 10 evaluation inputs
- NutriGen Jupyter Notebook from github.com/SamanKhamesian/NutriGen with API key placeholders ("KEY")
- requirements.txt dependencies (anthropic==0.45.2, google.generativeai==0.8.4, llamaapi==0.1.36, openai==1.42.0, numpy==2.2.2, pandas==2.2.3)

## Outputs

- qualitative_failures.csv — structured table with columns for model, input index, failure type, and disclaimer presence
- Per-model disclaimer flags
- Enumerated failure entries (e.g., DeepSeek V3 missing plans on inputs 4–7; Claude 3.5 Haiku failure on input 5 and single-plan outputs on inputs 6/8/9/10; Llama 3.1 8B failure on input 8; Gemini 1.5 Pro missing fat and protein values)

## How to apply

Load or re-run the NutriGen Jupyter Notebook (Python 3.10) to obtain meal plan outputs for each model across the 10 evaluation inputs, first replacing "KEY" with a valid API key. Inspect each model's output per input for outright generation failures (e.g., no plan produced), structural defects such as producing only one plan where multiple were requested, and omitted fields like fat or protein values. Separately audit every output for the presence of a dietary disclaimer and record it per model. Compile all entries into a pandas DataFrame with columns for model, input index, failure type, and disclaimer presence, then save it as qualitative_failures.csv. Cross-check the catalog against accuracy findings—e.g., DeepSeek V3's higher MAE of 10.45% versus Llama 3.1 8B's 1.55% and GPT-3.5 Turbo's 3.68%—so qualitative failures are interpreted alongside quantitative error.

## Related tools

- **NutriGen** (Framework whose Jupyter Notebook produces the per-model, per-input meal plan outputs to be audited for failure modes) — github.com/SamanKhamesian/NutriGen
- **Claude 3.5 Haiku** (Evaluated model whose outputs exhibit failure on input 5 and single-plan outputs on inputs 6/8/9/10)
- **DeepSeek V3** (Evaluated model whose outputs failed on inputs 4–7 and showed higher MAE of 10.45%)
- **Llama 3.1 8B** (Evaluated model whose outputs failed on input 8; achieved lowest error of 1.55%)
- **GPT-3.5 Turbo** (Evaluated model (lowest error of 3.68%) used as a behavioral reference in the failure matrix)
- **Gemini 1.5 Pro** (Evaluated model that omitted fat and protein values from all plans)
- **pandas** (Used to compile failure entries and disclaimer flags into the structured qualitative_failures.csv table)

## Examples

```
python -c "import pandas as pd; df = pd.DataFrame(rows, columns=['model','input_index','failure_type','disclaimer']); df.to_csv('qualitative_failures.csv', index=False)"
```

## Evaluation signals

- The record enumerates DeepSeek V3 failures at inputs 4, 5, 6, and 7 and Llama 3.1 8B failure at input 8, matching the reported finding.
- Claude 3.5 Haiku entries include both the input-5 failure and single-plan outputs on inputs 6, 8, 9, and 10.
- Every Gemini 1.5 Pro plan entry carries a structural-defect flag for missing fat and protein values.
- Each model row includes a binary disclaimer-presence flag, and the table schema contains model, input index, failure type, and disclaimer columns.
- The saved qualitative_failures.csv loads cleanly in pandas (Python 3.10) with no malformed rows.

## Limitations

- Qualitative failure detection depends on manual inspection of notebook outputs; outright non-generation (as with DeepSeek V3) and subtle omissions (as with Gemini 1.5 Pro's missing fat and protein values) require different inspection strategies.
- Failure modes are specific to the 10 evaluation inputs and the six tested models; they may not generalize to other models, prompts, or inputs.
- Re-running requires valid API keys for each provider; without them the notebook cannot reproduce the raw outputs to audit.

## Evidence

- [other] DeepSeek V3 failed to generate meal plans for inputs 4, 5, 6, and 7, Claude 3.5 Haiku failed on input 5 (and produced only one plan for inputs 6, 8, 9, 10), Llama 3.1 8B failed on input 8: [evidence span withheld — anchor status: not_found]
- [other] Gemini 1.5 Pro omitted fat and protein values from all plans: [evidence span withheld — anchor status: not_found]
