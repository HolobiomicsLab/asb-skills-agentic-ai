---
name: mean-absolute-error-benchmark-evaluation
description: Use when comparing generative models that produce structured outputs
  (e.g., meal plans with total calories) against numeric target values, and you need
  a per-model MAE/percentage-error ranking against a reference database such as USDA.
when_to_use_negative:
- When the goal is to empirically validate a RAG extension (M_i = LLM(Q(x), P)) against
  the non-RAG baseline — the paper only proposes RAG as future work with known practical
  challenges, so this skill provides no such evidence
- When outputs are not numeric targets comparable to a reference (e.g., free-text
  dietary advice without total-nutrition summaries)
- When the reference database is not USDA-referenced or targets are not user-defined
  caloric values, since percentage errors are computed specifically against those
  targets
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3077
tools:
- name: NutriGen
  role: LLM-based framework that generates the personalized meal plans whose caloric
    accuracy is benchmarked
  repo: https://github.com/SamanKhamesian/NutriGen
- name: Llama 3.1 8B
  role: Candidate model achieving the lowest percentage error of 1.55%
- name: GPT-3.5 Turbo
  role: Candidate model achieving the second-lowest percentage error of 3.68% and
    among the fastest processing times
- name: DeepSeek V3
  role: Candidate model exhibiting a higher MAE of 10.45% and significantly slower
    processing time
- name: Python 3.10
  role: Implementation language; dependencies installed via requirements.txt (anthropic==0.45.2,
    google.generativeai==0.8.4, openai==1.42.0, pandas==2.2.3)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:8ea122f1554fc06210708c345faf8d4f816a077bedf7521c8d6b16c2e6d158ff
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
  - Python 3.10
  - anthropic
  - google.generativeai
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3077
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mean-absolute-error-benchmark-evaluation

## Summary

Benchmark LLM-generated personalized meal plans by computing the mean absolute percentage error between each model's average total calories and user-defined caloric targets, using the USDA nutrition database as the nutritional reference. In the NutriGen paper this identifies Llama 3.1 8B (1.55%) and GPT-3.5 Turbo (3.68%) as the most accurate models and DeepSeek V3 (10.45%) as the least accurate.

## When to use

Use when comparing generative models that produce structured outputs (e.g., meal plans with total calories) against numeric target values, and you need a per-model MAE/percentage-error ranking against a reference database such as USDA.

## When NOT to use

- When the goal is to empirically validate a RAG extension (M_i = LLM(Q(x), P)) against the non-RAG baseline — the paper only proposes RAG as future work with known practical challenges, so this skill provides no such evidence
- When outputs are not numeric targets comparable to a reference (e.g., free-text dietary advice without total-nutrition summaries)
- When the reference database is not USDA-referenced or targets are not user-defined caloric values, since percentage errors are computed specifically against those targets

## Inputs

- User dietary profiles with user-defined caloric targets and dietary preferences/constraints
- Prompts for each LLM (Llama 3.1 8B, GPT-3.5 Turbo, DeepSeek V3) executed via Jupyter Notebook with API keys
- 10 generated meal-plan outputs per model
- USDA nutrition database as the nutritional reference

## Outputs

- Per-model mean absolute percentage error versus caloric targets (e.g., Llama 3.1 8B 1.55%, GPT-3.5 Turbo 3.68%, DeepSeek V3 10.45%)
- Comparison of average total calories per model against target values (figures/usda_vs_target.png)
- Total processing time for 10 outputs per model (figures/time_comparison.png)

## How to apply

Run each candidate LLM (here Llama 3.1 8B, GPT-3.5 Turbo, and DeepSeek V3 via the anthropic, google.generativeai, and OpenAI-compatible clients in Python 3.10) on the same user dietary profiles to generate 10 outputs per model. Compute the average total nutrition (calories) across the 10 outputs for each input, then calculate the mean absolute percentage error against the specified target values derived from USDA-referenced nutritional data. Optionally record the total processing time for the 10 outputs per model to jointly assess efficiency, since DeepSeek V3 was both less accurate (MAE 10.45%) and significantly slower than GPT-3.5 Turbo and Llama 3.1 8B. Rank models by MAE; in the NutriGen evaluation, models with the lowest percentage errors (1.55% and 3.68%) were judged to produce meal plans that closely align with user-defined caloric targets. Note that this skill evaluates the non-RAG baseline only: the paper proposes RAG retrieval of USDA values as future work and provides no experimental evidence that RAG reduces MAE relative to this baseline.

## Related tools

- **NutriGen** (LLM-based framework that generates the personalized meal plans whose caloric accuracy is benchmarked) — https://github.com/SamanKhamesian/NutriGen
- **Llama 3.1 8B** (Candidate model achieving the lowest percentage error of 1.55%)
- **GPT-3.5 Turbo** (Candidate model achieving the second-lowest percentage error of 3.68% and among the fastest processing times)
- **DeepSeek V3** (Candidate model exhibiting a higher MAE of 10.45% and significantly slower processing time)
- **Python 3.10** (Implementation language; dependencies installed via requirements.txt (anthropic==0.45.2, google.generativeai==0.8.4, openai==1.42.0, pandas==2.2.3))

## Examples

```
pip install -r requirements.txt  # then execute the provided Jupyter Notebook (replace "KEY" with a valid API key) to generate 10 outputs per model and compute MAE vs caloric targets
```

## Evaluation signals

- Percentage errors reproduce the reported baselines: Llama 3.1 8B at 1.55%, GPT-3.5 Turbo at 3.68%, DeepSeek V3 at 10.45%
- MAE is computed over 10 outputs per model, comparing average total nutrition for each input against the specified target values
- Processing-time results are consistent with the paper's finding that GPT-3.5 Turbo, Gemini 2.0 Flash, and Claude 3.5 Haiku complete fastest
- Generated meal plans remain aligned with user-defined dietary preferences and constraints, not just caloric targets
- Ranking is interpretable: smaller and optimized models are expected to be faster; DeepSeek V3 should rank worst on both accuracy and speed

## Limitations

- MAE here measures caloric-target adherence only; macronutrient-level accuracy against the USDA database is not experimentally evaluated
- No RAG vs non-RAG comparison exists: RAG retrieval of USDA values is proposed only as a future extension with known practical challenges
- DeepSeek V3's higher MAE (10.45%) and slower processing highlight that results do not transfer across model scale; further optimization was noted as needed
- Results depend on API availability and valid keys (replace "KEY" before execution); the repo provides no automated test harness beyond the Jupyter Notebook

## Evidence
