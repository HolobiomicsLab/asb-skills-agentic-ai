---
name: llm-prompt-engineering-for-dietary-constraints
description: Use when generating personalized meal plans with LLMs (Llama 3.1 8B,
  GPT-3.5 Turbo, DeepSeek V3) that must adhere to user-defined caloric targets and
  dietary constraints.
when_to_use_negative:
- When you need empirically validated RAG retrieval of USDA values — the paper only
  proposes RAG as a future extension with known practical challenges; no experiment
  shows it reduces MAE versus the non-RAG baseline
- When you need adaptability to real-world constraints such as food ingredient availability
  — this is a stated limitation of existing systems the paper aims to address, not
  a solved capability
- When generation latency matters and you require the fastest models — DeepSeek V3
  was significantly slower and less accurate (MAE 10.45%)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3379
tools:
- name: NutriGen
  role: LLM-based framework for generating personalized meal plans aligned with user-defined
    dietary preferences and constraints
  repo: github.com/SamanKhamesian/NutriGen
- name: Llama 3.1 8B
  role: Best-performing model, lowest percentage error of 1.55% vs caloric targets
  repo: github.com/SamanKhamesian/NutriGen
- name: GPT-3.5 Turbo
  role: Second-best model with 3.68% error; also among the fastest in processing time
  repo: github.com/SamanKhamesian/NutriGen
- name: DeepSeek V3
  role: Comparison model; higher MAE of 10.45% and significantly slower processing
  repo: github.com/SamanKhamesian/NutriGen
- name: USDA nutrition database
  role: Reliable nutritional reference incorporated via the personalized nutrition
    database and prompt engineering
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:f58dba39c7a2fcf5a69498104aa8fcb628012c9a6e882512ae507889038ff4e1
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# llm-prompt-engineering-for-dietary-constraints

## Summary

Prompt-engineering skill from the NutriGen framework: inject a personalized nutrition database of USDA-referenced values and user-defined dietary preferences/constraints into LLM prompts so generated meal plans align with target calories.

## When to use

Use when generating personalized meal plans with LLMs (Llama 3.1 8B, GPT-3.5 Turbo, DeepSeek V3) that must adhere to user-defined caloric targets and dietary constraints.

## When NOT to use

- When you need empirically validated RAG retrieval of USDA values — the paper only proposes RAG as a future extension with known practical challenges; no experiment shows it reduces MAE versus the non-RAG baseline
- When you need adaptability to real-world constraints such as food ingredient availability — this is a stated limitation of existing systems the paper aims to address, not a solved capability
- When generation latency matters and you require the fastest models — DeepSeek V3 was significantly slower and less accurate (MAE 10.45%)

## Inputs

- User-defined dietary preferences and constraints (e.g. caloric targets, restrictions)
- Personalized nutrition database with USDA-referenced nutritional values
- Valid API keys for the LLM providers (placeholder "KEY" in the code must be replaced)
- requirements.txt dependency set (Python 3.10)

## Outputs

- 10 structured meal-plan outputs per LLM model
- Average total nutrition (calories) per input compared against specified target values
- Model accuracy metrics: percentage errors / MAE vs caloric targets (e.g. 1.55%, 3.68%, 10.45%)
- Per-model total processing time for 10 outputs

## How to apply

Build a personalized nutrition database and leverage prompt engineering so the LLM can incorporate reliable nutritional references like the USDA nutrition database while keeping prompts flexible. Run NutriGen in Python 3.10 (dependencies include anthropic==0.45.2, google.generativeai==0.8.4, llamaapi==0.1.36, openai==1.42.0, numpy==2.2.2, pandas==2.2.3), replacing "KEY" with a valid API key before executing the provided Jupyter Notebook. Generate 10 outputs per model and evaluate by comparing the average total nutrition (calories) of each generated meal plan against the specified target values. Use the reported percentage errors as benchmarks: Llama 3.1 8B at 1.55% and GPT-3.5 Turbo at 3.68%; DeepSeek V3 showed a higher MAE of 10.45% and was significantly slower. Important caveat: RAG-based retrieval of USDA values is only proposed as future work — the paper provides no empirical evidence that retrieval reduces MAE versus the non-RAG baseline, so do not claim RAG-augmented gains without running that comparison yourself.

## Related tools

- **NutriGen** (LLM-based framework for generating personalized meal plans aligned with user-defined dietary preferences and constraints) — github.com/SamanKhamesian/NutriGen
- **Llama 3.1 8B** (Best-performing model, lowest percentage error of 1.55% vs caloric targets) — github.com/SamanKhamesian/NutriGen
- **GPT-3.5 Turbo** (Second-best model with 3.68% error; also among the fastest in processing time) — github.com/SamanKhamesian/NutriGen
- **DeepSeek V3** (Comparison model; higher MAE of 10.45% and significantly slower processing) — github.com/SamanKhamesian/NutriGen
- **USDA nutrition database** (Reliable nutritional reference incorporated via the personalized nutrition database and prompt engineering)

## Examples

```
pip install -r requirements.txt  # then in Python 3.10: replace "KEY" with a valid API key and execute the provided Jupyter Notebook to generate 10 meal plans per model
```

## Evaluation signals

- Percentage error between average total calories of generated meal plans and user-defined caloric targets is ≤ ~4% for top models (1.55% Llama 3.1 8B, 3.68% GPT-3.5 Turbo reported)
- Exactly 10 outputs are generated per model, matching the paper's evaluation protocol
- Average total nutrition per input is compared against the specified target values (usda_vs_target comparison)
- Total processing time per model is measured over the 10 outputs and falls within plausible API latency ranges (DeepSeek V3 expected to be significantly slower)
- Generated plans are structured, practical, and honor the user-defined dietary preferences and constraints stated in the prompt

## Limitations

- RAG retrieval of USDA-referenced values is proposed but never experimentally evaluated; no evidence it reduces MAE versus the non-RAG baseline reported in Table I
- DeepSeek V3 exhibited a higher MAE of 10.45% and significantly slower processing than GPT-3.5 Turbo and Llama 3.1 8B, highlighting the need for further optimization
- Evaluation is limited to matching caloric targets; real-world constraints such as food ingredient availability remain unaddressed
- Requires a valid API key ("KEY" placeholder must be replaced) and Python 3.10 with pinned dependencies

## Evidence

- [other] RAG not evaluated: [evidence span withheld — anchor status: not_found]
