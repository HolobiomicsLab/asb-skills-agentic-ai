---
name: mean-absolute-error-computation
description: Use when evaluating how closely generated meal plans (or other numeric
  predictions) adhere to user-specified target values and you need per-model MAE /
  MAE % for comparison tables.
when_to_use_negative:
- The comparison goal is processing-time benchmarking rather than caloric adherence
  — use the paper's separate timing analysis (total processing time for 10 outputs
  per model) instead
- The predictions are not numeric values comparable against a target (e.g., free-text
  dietary advice with no calorie totals), since MAE requires numeric actual-vs-target
  pairs
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: NutriGen
  role: Framework whose generated meal plans supply the actual total calories compared
    against targets
  repo: github.com/SamanKhamesian/NutriGen
- name: numpy
  role: Computes MAE and MAE % between average total calories and specified targets
    (Equation 2)
- name: pandas
  role: Assembles per-model MAE / MAE % into a Table II-style table and writes it
    to CSV
- name: Llama 3.1 8B
  role: Best-adherence model (MAE % 1.55%) used as validation reference for the computed
    errors
- name: GPT-3.5 Turbo
  role: Second-best-adherence model (MAE % 3.68%) used as validation reference
- name: DeepSeek V3
  role: High-error model (MAE % 10.45%) used as lower-bound validation reference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:5986a583fc036cd9624ba5c0885c13e26926e31223e757a082d44b129b64738b
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
  - numpy
  - openai
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# mean-absolute-error-computation

## Summary

Computes the mean absolute error (MAE) and MAE % between average total calories of LLM-generated meal plans and user-specified caloric targets, as used in NutriGen to rank the adherence of 10 models (e.g., Llama 3.1 8B at 1.55%, GPT-3.5 Turbo at 3.68%, DeepSeek V3 at 10.45%).

## When to use

Use when evaluating how closely generated meal plans (or other numeric predictions) adhere to user-specified target values and you need per-model MAE / MAE % for comparison tables.

## When NOT to use

- The comparison goal is processing-time benchmarking rather than caloric adherence — use the paper's separate timing analysis (total processing time for 10 outputs per model) instead
- The predictions are not numeric values comparable against a target (e.g., free-text dietary advice with no calorie totals), since MAE requires numeric actual-vs-target pairs

## Inputs

- Per-model generated meal plans (3 per input profile) with total calories per plan
- User-specified caloric target values for each simulated dietary profile
- Model identifiers for each of the 10 evaluated LLMs (GPT-3.5 Turbo, Llama 3.1 8B, DeepSeek V3, Claude 3.5 Haiku, Gemini 2.0 Flash, etc.)

## Outputs

- Per-model MAE and MAE % values (Table II structure)
- CSV file of per-model MAE / MAE % results (e.g., table_ii_mae_results.csv)
- Per-input average total calories per model for target-vs-actual comparison

## How to apply

Generate multiple meal plans per input profile per model (three meal plans per input in the source task; the paper measured processing over 10 outputs per model), then average the total calories per input per model. Compute the MAE between these averages and the user-specified caloric target per Equation 2, and express it as MAE % so results are comparable across targets. Assemble per-model MAE and MAE % rows into a table mirroring Table II and validate against reference values (Llama 3.1 8B 1.55%, GPT-3.5 Turbo 3.68%, DeepSeek V3 10.45%) within tolerance. Implement with numpy and pandas on Python 3.10, and record results to a CSV such as table_ii_mae_results.csv.

## Related tools

- **NutriGen** (Framework whose generated meal plans supply the actual total calories compared against targets) — github.com/SamanKhamesian/NutriGen
- **numpy** (Computes MAE and MAE % between average total calories and specified targets (Equation 2))
- **pandas** (Assembles per-model MAE / MAE % into a Table II-style table and writes it to CSV)
- **Llama 3.1 8B** (Best-adherence model (MAE % 1.55%) used as validation reference for the computed errors)
- **GPT-3.5 Turbo** (Second-best-adherence model (MAE % 3.68%) used as validation reference)
- **DeepSeek V3** (High-error model (MAE % 10.45%) used as lower-bound validation reference)

## Examples

```
mae = np.mean(np.abs(df.groupby(['model','input'])['total_calories'].mean() - df.groupby(['model','input'])['target_calories'].first())); df_out.to_csv('table_ii_mae_results.csv')
```

## Evaluation signals

- Reproduced MAE % for Llama 3.1 8B is 1.55%, GPT-3.5 Turbo is 3.68%, and DeepSeek V3 is 10.45%, each within a stated tolerance
- Output CSV contains one row with MAE and MAE % for each of the 10 evaluated models
- Averaging is done over three meal plans per input profile per model before differencing against the caloric target
- MAE is computed between average total calories and the user-specified target per Equation 2, and MAE % is the same quantity expressed as a percentage
- Table structure mirrors Table II of the paper (model, MAE, MAE %)

## Limitations

- Errors are computed against user-specified caloric targets only; other nutritional dimensions (e.g., macros) are not covered by this metric in the source workflow
- DeepSeek V3 showed both high error (10.45% MAE) and significantly slower processing time, so model ranking on MAE alone does not capture cost
- Results depend on API key access and valid SDK calls to each provider; invalid or missing keys will prevent reproduction

## Evidence
