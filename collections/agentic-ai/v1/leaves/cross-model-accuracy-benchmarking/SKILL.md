---
name: cross-model-accuracy-benchmarking
description: Use when comparing several LLMs' outputs against quantitative user-specified
  targets (e.g., calories) and you need per-model MAE/MAE % metrics to rank adherence,
  as in reproducing Table II of the NutriGen paper.
when_to_use_negative:
- Evaluating meal plans on dimensions other than numeric caloric adherence (e.g.,
  ingredient availability, user-friendliness) — this skill only measures MAE against
  caloric targets
- Benchmarking models on processing time rather than accuracy — the paper measures
  time separately (10 outputs per model) and it should not be conflated with MAE %
- You lack valid API keys for the providers — the workflow requires replacing "KEY"
  with a valid API key before execution
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: NutriGen
  role: Framework whose Jupyter Notebook provides the dietary profiles and prompt
    template used to generate meal plans from each LLM
  repo: github.com/SamanKhamesian/NutriGen
- name: GPT-3.5 Turbo
  role: Evaluated LLM benchmarked for caloric-target adherence (reported MAE % 3.68%)
- name: Llama 3.1 8B
  role: Evaluated LLM benchmarked for caloric-target adherence (reported MAE % 1.55%)
- name: DeepSeek V3
  role: Evaluated LLM benchmarked for caloric-target adherence (reported MAE % 10.45%)
- name: numpy
  role: Numerical computation of MAE between actual and target total calories
- name: pandas
  role: Assembling per-model MAE and MAE % values into a Table II-mirroring results
    table and writing CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:8cc17285cb31aa5e896e1965939ba7adcac3f5f1643605e3d37f1f282e74c3ce
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cross-model-accuracy-benchmarking

## Summary

Benchmark multiple LLMs on how closely their generated meal plans adhere to user-specified caloric targets by computing per-model MAE and MAE % (Table II of NutriGen), enabling direct cross-model accuracy comparison in personalized nutrition generation.

## When to use

Use when comparing several LLMs' outputs against quantitative user-specified targets (e.g., calories) and you need per-model MAE/MAE % metrics to rank adherence, as in reproducing Table II of the NutriGen paper.

## When NOT to use

- Evaluating meal plans on dimensions other than numeric caloric adherence (e.g., ingredient availability, user-friendliness) — this skill only measures MAE against caloric targets
- Benchmarking models on processing time rather than accuracy — the paper measures time separately (10 outputs per model) and it should not be conflated with MAE %
- You lack valid API keys for the providers — the workflow requires replacing "KEY" with a valid API key before execution

## Inputs

- Simulated dietary profiles with user-specified caloric targets (from the NutriGen Jupyter Notebook)
- Prompt template from the NutriGen Jupyter Notebook
- Valid provider API keys inserted in place of the "KEY" placeholder
- Generated meal plans (3 per input profile per model) from the 10 evaluated LLMs

## Outputs

- Per-model average total calories per input profile
- Table II-style table with MAE and MAE % for each of the 10 models (e.g., table_ii_mae_results.csv)

## How to apply

Install the Python 3.10 dependencies from requirements.txt (openai==1.42.0, anthropic==0.45.2, google.generativeai==0.8.4, llamaapi==0.1.36, numpy==2.2.2, pandas==2.2.3). Load the simulated dietary profiles and prompt template from the provided Jupyter Notebook in the NutriGen repository, replacing the "KEY" placeholder with valid provider API keys. Call each of the 10 evaluated models (e.g., GPT-3.5 Turbo, Llama 3.1 8B, DeepSeek V3, Claude 3.5 Haiku, Gemini 2.0 Flash) via their SDK clients to generate meal plans per input profile. Compute the average total calories per input per model and the MAE between actual total calories and the specified caloric target (Equation 2), expressing it as MAE % with numpy and pandas. Assemble the per-model MAE and MAE % into a table mirroring Table II and write it to CSV (e.g., table_ii_mae_results.csv).

## Related tools

- **NutriGen** (Framework whose Jupyter Notebook provides the dietary profiles and prompt template used to generate meal plans from each LLM) — github.com/SamanKhamesian/NutriGen
- **GPT-3.5 Turbo** (Evaluated LLM benchmarked for caloric-target adherence (reported MAE % 3.68%))
- **Llama 3.1 8B** (Evaluated LLM benchmarked for caloric-target adherence (reported MAE % 1.55%))
- **DeepSeek V3** (Evaluated LLM benchmarked for caloric-target adherence (reported MAE % 10.45%))
- **numpy** (Numerical computation of MAE between actual and target total calories)
- **pandas** (Assembling per-model MAE and MAE % values into a Table II-mirroring results table and writing CSV)

## Examples

```
pip install -r requirements.txt && jupyter notebook
```

## Evaluation signals

- Reproduced MAE % for Llama 3.1 8B is 1.55% within tolerance
- Reproduced MAE % for GPT-3.5 Turbo is 3.68% within tolerance
- Reproduced MAE % for DeepSeek V3 is 10.45% within tolerance
- Output CSV contains a row with MAE and MAE % for each of the 10 evaluated models
- Per-input average total calories per model are computed from three generated meal plans per input profile

## Limitations

- DeepSeek V3 struggled with accuracy (MAE of 10.45%) and was significantly slower in processing time compared to models like GPT-3.5 Turbo and Llama 3.1 8B
- Results require valid provider API keys; models are accessed through third-party SDKs (openai, anthropic, google.generativeai, llamaapi), so reported figures depend on model versions available at evaluation time
- The metric only captures caloric-target adherence; other user-defined dietary preferences and constraints are not quantified by MAE %

## Evidence
