---
name: retrieval-augmented-meal-plan-generation
description: Use when evaluating whether retrieving USDA-referenced nutritional values
  for meal items into LLM prompts reduces MAE versus caloric targets compared to prompts
  without retrieved values, in the NutriGen framework.
when_to_use_negative:
- The goal is to cite published empirical evidence that RAG reduces MAE — the paper
  only proposes RAG as a future extension and reports no such experiment, only the
  non-RAG baseline errors of 1.55%, 3.68%, and 10.45%.
- Evaluation targets micronutrient-level accuracy rather than the paper's metric of
  average total calories against user-defined caloric targets.
- No USDA nutrition database lookup or valid API keys ('KEY' placeholders) are available
  for the LLM clients.
edam_operation: http://edamontology.org/operation_2422
tools:
- name: NutriGen
  role: LLM-based framework whose prompt template P and personalized nutrition database
    host the RAG retrieval and meal-plan generation
  repo: github.com/SamanKhamesian/NutriGen
- name: Llama 3.1 8B
  role: Evaluated LLM achieving the lowest non-RAG baseline percentage error (1.55%)
    used as the reference point for RAG-vs-baseline comparison
- name: GPT-3.5 Turbo
  role: Evaluated LLM with the second-lowest non-RAG baseline percentage error (3.68%)
    and among the fastest processing times
- name: DeepSeek V3
  role: Evaluated LLM with the highest non-RAG baseline MAE (10.45%) and significantly
    slower processing time, included to test whether RAG compensates for weak accuracy
- name: Python
  role: Python 3.10 runtime for implementing Q(x), the prompt template, API calls,
    and MAE computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:1e17aaf024317898563387b3cfc3924c9b75da3485f2862a1883937425bc9f00
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# retrieval-augmented-meal-plan-generation

## Summary

Implements the NutriGen RAG extension M_i = LLM(Q(x), P), where a retrieval function Q(x) injects USDA-referenced calories and macronutrients for each meal item into the prompt template P before LLM generation. The rationale is to test whether grounding LLM meal plans in the USDA nutrition database reduces MAE against caloric targets relative to the non-RAG baseline (Llama 3.1 8B 1.55%, GPT-3.5 Turbo 3.68%, DeepSeek V3 10.45%).

## When to use

Use when evaluating whether retrieving USDA-referenced nutritional values for meal items into LLM prompts reduces MAE versus caloric targets compared to prompts without retrieved values, in the NutriGen framework.

## When NOT to use

- The goal is to cite published empirical evidence that RAG reduces MAE — the paper only proposes RAG as a future extension and reports no such experiment, only the non-RAG baseline errors of 1.55%, 3.68%, and 10.45%.
- Evaluation targets micronutrient-level accuracy rather than the paper's metric of average total calories against user-defined caloric targets.
- No USDA nutrition database lookup or valid API keys ('KEY' placeholders) are available for the LLM clients.

## Inputs

- User dietary profiles with user-defined caloric targets and dietary preferences/constraints (NutriGen input format)
- USDA nutrition database (USDA-referenced calories and macronutrients per food item)
- Prompt template P for the NutriGen framework
- requirements.txt dependencies (anthropic==0.45.2, google.generativeai==0.8.4, llamaapi==0.1.36, openai==1.42.0, numpy==2.2.2, pandas==2.2.3)

## Outputs

- mae_comparison.csv: RAG vs non-RAG baseline MAE per model (Llama 3.1 8B, GPT-3.5 Turbo, DeepSeek V3)
- figures/usda_vs_target.png: average total calories of generated meal plans vs target values per input
- 10 generated meal-plan outputs per model with and without injected USDA-referenced values

## How to apply

Load the user dietary profiles with their caloric targets together with the USDA nutrition database, and implement the retrieval function Q(x) that looks up USDA-referenced calories and macronutrients for each meal item. Build the prompt template P within NutriGen that injects the retrieved USDA values alongside user-defined dietary preferences and constraints, then call Llama 3.1 8B, GPT-3.5 Turbo, and DeepSeek V3 (via the anthropic==0.45.2, google.generativeai==0.8.4, and OpenAI-compatible clients in Python 3.10) to generate 10 outputs per model. Compute the mean absolute percentage error between the average total calories of the generated meal plans and the specified target values, following the paper's evaluation protocol of comparing average total nutrition per input against target values. Repeat the same generation with retrieval disabled (prompts without injected USDA values) to obtain the non-RAG baseline MAE. Accept the RAG extension only if the augmented MAE is lower than the non-RAG baseline for each model, using the paper's reported percentage errors (1.55%, 3.68%, 10.45%) as reference points. Note that the source paper only proposes RAG as a future extension and reports no such experiment, so this comparison must be run fresh.

## Related tools

- **NutriGen** (LLM-based framework whose prompt template P and personalized nutrition database host the RAG retrieval and meal-plan generation) — github.com/SamanKhamesian/NutriGen
- **Llama 3.1 8B** (Evaluated LLM achieving the lowest non-RAG baseline percentage error (1.55%) used as the reference point for RAG-vs-baseline comparison)
- **GPT-3.5 Turbo** (Evaluated LLM with the second-lowest non-RAG baseline percentage error (3.68%) and among the fastest processing times)
- **DeepSeek V3** (Evaluated LLM with the highest non-RAG baseline MAE (10.45%) and significantly slower processing time, included to test whether RAG compensates for weak accuracy)
- **Python** (Python 3.10 runtime for implementing Q(x), the prompt template, API calls, and MAE computation)

## Examples

```
pip install -r requirements.txt && jupyter notebook  # then set "KEY" to a valid API key before executing
```

## Evaluation signals

- RAG-augmented MAE is strictly lower than the non-RAG baseline MAE for each of the three models, otherwise the extension is rejected
- Non-RAG baseline percentage errors fall near the paper's reported values (Llama 3.1 8B 1.55%, GPT-3.5 Turbo 3.68%, DeepSeek V3 10.45%), validating the reproduction
- mae_comparison.csv contains one RAG and one baseline MAE row per model with no missing model entries
- figures/usda_vs_target.png shows average total calories per input plotted against the specified target values
- Exactly 10 generated outputs per model per condition, matching the paper's protocol of measuring over 10 outputs

## Limitations

- The paper only proposes RAG as a future extension and acknowledges its known practical challenges; it provides no experiment showing RAG reduces MAE versus the non-RAG baseline in Table I
- DeepSeek V3 exhibited a higher MAE of 10.45% and was significantly slower in processing time, so RAG may not recover its accuracy
- All errors are percentage errors against user-defined caloric targets only; the protocol does not validate other nutrients
- The sample is small (10 outputs per model) and requires valid API keys, so results may vary with API versions and nondeterministic generation

## Evidence
