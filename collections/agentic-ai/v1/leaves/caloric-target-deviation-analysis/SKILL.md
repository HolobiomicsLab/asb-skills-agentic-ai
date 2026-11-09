---
name: caloric-target-deviation-analysis
description: Use when evaluating LLM-generated meal plans against user-defined caloric
  targets, e.g. reproducing the NutriGen Table II comparison across models like Llama
  3.1 8B, GPT-3.5 Turbo, or DeepSeek V3.
when_to_use_negative:
- The goal is to evaluate retrieval-augmented accuracy against USDA reference values
  (RAG extension) rather than adherence to user-specified caloric targets — the paper
  proposes RAG only as future work and provides no baseline for that comparison.
- The target metric is processing time or throughput of model generation rather than
  caloric accuracy, where the paper instead reports total processing time to generate
  10 outputs per model.
- Evaluating macronutrient-level or micronutrient-level accuracy — the paper's evaluation
  compares only average total nutrition (calories) against specified target values.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NutriGen
  role: Framework providing the prompt template and simulated dietary profiles used
    to generate meal plans whose caloric adherence is evaluated
  repo: github.com/SamanKhamesian/NutriGen
- name: numpy
  role: Computes mean absolute error and MAE % between generated average total calories
    and target values
- name: pandas
  role: Assembles per-model MAE and MAE % values into the Table II results table (table_ii_mae_results.csv)
- name: Llama 3.1 8B
  role: Best-performing model whose 1.55% MAE serves as the validation reference for
    the lowest-error result
- name: GPT-3.5 Turbo
  role: Second-best model whose 3.68% MAE serves as a validation reference point
- name: DeepSeek V3
  role: Worst-accuracy model whose 10.45% MAE defines the upper bound of expected
    deviation
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:dc1f670ce4e21f28ee49fe7d316aa228e0aaf6eabb2bfe77df673f6f0c59d8aa
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# caloric-target-deviation-analysis

## Summary

Quantifies how closely LLM-generated meal plans adhere to user-specified caloric targets by computing mean absolute error (MAE and MAE %) between the average total calories of generated plans and target values, as used to rank 10 models in the NutriGen evaluation.

## When to use

Use when evaluating LLM-generated meal plans against user-defined caloric targets, e.g. reproducing the NutriGen Table II comparison across models like Llama 3.1 8B, GPT-3.5 Turbo, or DeepSeek V3.

## When NOT to use

- The goal is to evaluate retrieval-augmented accuracy against USDA reference values (RAG extension) rather than adherence to user-specified caloric targets — the paper proposes RAG only as future work and provides no baseline for that comparison.
- The target metric is processing time or throughput of model generation rather than caloric accuracy, where the paper instead reports total processing time to generate 10 outputs per model.
- Evaluating macronutrient-level or micronutrient-level accuracy — the paper's evaluation compares only average total nutrition (calories) against specified target values.

## Inputs

- Simulated dietary profiles with user-specified caloric targets (loaded from the NutriGen Jupyter Notebook prompt template)
- Generated meal-plan outputs (10 per model) from LLMs called via openai, anthropic, google.generativeai, and llamaapi SDK clients
- Pandas DataFrame of per-input, per-model average total calories and target values

## Outputs

- table_ii_mae_results.csv with MAE and MAE % rows for each of the 10 evaluated models
- Per-model MAE % values comparable to reported figures (Llama 3.1 8B 1.55%, GPT-3.5 Turbo 3.68%, DeepSeek V3 10.45%)

## How to apply

For each dietary profile and each model, generate multiple meal-plan outputs (the NutriGen evaluation used 10 outputs per model via the project's Jupyter Notebook with valid API keys replacing the "KEY" placeholders). Average the total calories per input per model, then compute the MAE between actual average total calories and the specified caloric target per Equation 2, expressed as MAE % using numpy and pandas. Assemble per-model MAE and MAE % into a table mirroring Table II and validate against the paper's reference points: Llama 3.1 8B at 1.55%, GPT-3.5 Turbo at 3.68%, and DeepSeek V3 at 10.45%. Lower MAE % indicates better adherence to user-defined caloric targets; models with the lowest percentage errors are judged best-performing.

## Related tools

- **NutriGen** (Framework providing the prompt template and simulated dietary profiles used to generate meal plans whose caloric adherence is evaluated) — github.com/SamanKhamesian/NutriGen
- **numpy** (Computes mean absolute error and MAE % between generated average total calories and target values)
- **pandas** (Assembles per-model MAE and MAE % values into the Table II results table (table_ii_mae_results.csv))
- **Llama 3.1 8B** (Best-performing model whose 1.55% MAE serves as the validation reference for the lowest-error result)
- **GPT-3.5 Turbo** (Second-best model whose 3.68% MAE serves as a validation reference point)
- **DeepSeek V3** (Worst-accuracy model whose 10.45% MAE defines the upper bound of expected deviation)

## Examples

```
import pandas as pd; df = pd.read_csv('generated_totals.csv'); mae = (df['avg_total_calories'] - df['target_calories']).abs().mean(); mae_pct = mae / df['target_calories'].mean() * 100
```

## Evaluation signals

- The reproduced MAE % for Llama 3.1 8B falls within tolerance of 1.55% and GPT-3.5 Turbo within tolerance of 3.68%.
- DeepSeek V3 reproduces a MAE of approximately 10.45%, notably higher than the other reference models.
- The output CSV contains a row with MAE and MAE % for each of the 10 evaluated models.
- MAE is computed as the mean absolute difference between average total calories per input and the specified caloric target, consistent with Equation 2.
- The MAE % ranking places Llama 3.1 8B and GPT-3.5 Turbo as the two lowest-error models among the evaluated set.

## Limitations

- The evaluation covers only caloric adherence (average total nutrition vs. target values); it does not validate macronutrient or micronutrient accuracy.
- The reported figures depend on API model versions (e.g., GPT-3.5 Turbo, Claude 3.5 Haiku, Gemini 2.0 Flash) that may change over time, affecting reproducibility.
- The paper only proposes RAG-based retrieval of USDA values as a future extension with known practical challenges; no experimental evidence exists that retrieval reduces MAE against USDA reference values.
- DeepSeek V3 was significantly slower in processing time, so large-scale replication of 10 outputs per model across all 10 models may be time-consuming for larger models.

## Evidence
