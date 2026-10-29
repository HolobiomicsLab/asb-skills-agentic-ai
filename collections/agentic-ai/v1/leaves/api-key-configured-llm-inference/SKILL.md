---
name: api-key-configured-llm-inference
description: 'Use when replicating the NutriGen pipeline: Python 3.10 dependencies
  are installed and you need to replace the ''KEY'' placeholder with valid provider
  API keys before executing the Jupyter Notebook to generate meal plans.'
when_to_use_negative:
- No valid provider API keys are available — the notebook cannot execute without replacing
  'KEY', so inference will fail
- The goal is offline or local-only model evaluation; the framework relies on hosted
  LLM APIs via the anthropic, openai, google.generativeai, and llamaapi clients
edam_operation: http://edamontology.org/operation_2422
tools:
- name: NutriGen
  role: LLM-based framework that generates personalized meal plans aligned with user-defined
    dietary preferences and constraints; invoked via the provided Jupyter Notebook
  repo: github.com/SamanKhamesian/NutriGen
- name: openai
  role: Python client for GPT-3.5 Turbo inference used in the evaluation
  repo: github.com/SamanKhamesian/NutriGen
- name: anthropic
  role: Python client for Claude model inference used in the evaluation
  repo: github.com/SamanKhamesian/NutriGen
- name: google.generativeai
  role: Python client for Gemini model inference used in the evaluation
  repo: github.com/SamanKhamesian/NutriGen
- name: llamaapi
  role: Python client for Llama 3.1 8B inference used in the evaluation
  repo: github.com/SamanKhamesian/NutriGen
- name: pandas
  role: Summing per-item reported calories of generated meal plans
  repo: github.com/SamanKhamesian/NutriGen
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/embc58623.2025.11253879
    title: 'NutriGen: Personalized Meal Plan Generator Leveraging Large Language Models
      to Enhance Dietary and Nutritional Adherence'
  source_content_hash: sha256:e14112b0eec24fba89cc4a88b36f74e36605c6f0bf7ce93c44230750c1509ba5
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
  - openai
  - pandas
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# api-key-configured-llm-inference

## Summary

Configure provider API keys and execute LLM inference (GPT-3.5 Turbo, Llama 3.1 8B, DeepSeek V3, etc.) through the NutriGen framework to generate personalized meal plans. This is the foundational setup that enables all downstream calorie-accuracy and processing-time evaluations.

## When to use

Use when replicating the NutriGen pipeline: Python 3.10 dependencies are installed and you need to replace the 'KEY' placeholder with valid provider API keys before executing the Jupyter Notebook to generate meal plans.

## When NOT to use

- No valid provider API keys are available — the notebook cannot execute without replacing 'KEY', so inference will fail
- The goal is offline or local-only model evaluation; the framework relies on hosted LLM APIs via the anthropic, openai, google.generativeai, and llamaapi clients

## Inputs

- Provider API keys (OpenAI, Anthropic, Google Generative AI, LlamaAPI) replacing the 'KEY' placeholder
- 10 simulated user inputs specifying dietary preferences and caloric targets
- requirements.txt with pinned versions: anthropic==0.45.2, google.generativeai==0.8.4, llamaapi==0.1.36, openai==1.42.0, numpy==2.2.2, pandas==2.2.3

## Outputs

- Generated personalized meal plans (10 inputs x 3 replicates per model, across ~10 LLMs)
- Records of models that failed to generate a plan (later rendered as '-' in the error matrix)
- Per-plan itemized nutrition data suitable for summing total calories with pandas

## How to apply

Set up a Python 3.10 environment and install pinned dependencies via 'pip install -r requirements.txt' (anthropic==0.45.2, google.generativeai==0.8.4, llamaapi==0.1.36, openai==1.42.0, numpy==2.2.2, pandas==2.2.3). Replace the placeholder 'KEY' in the code with a valid API key for each provider (OpenAI, Anthropic, Google Generative AI, Llama API) before execution. Then execute the provided Jupyter Notebook, which drives the NutriGen framework: for each of 10 simulated user inputs, each LLM is prompted three times to generate a personalized meal plan, with failures recorded. The resulting plans feed into downstream comparisons of average total calories against USDA reference values and user-defined caloric targets.

## Related tools

- **NutriGen** (LLM-based framework that generates personalized meal plans aligned with user-defined dietary preferences and constraints; invoked via the provided Jupyter Notebook) — github.com/SamanKhamesian/NutriGen
- **openai** (Python client for GPT-3.5 Turbo inference used in the evaluation) — github.com/SamanKhamesian/NutriGen
- **anthropic** (Python client for Claude model inference used in the evaluation) — github.com/SamanKhamesian/NutriGen
- **google.generativeai** (Python client for Gemini model inference used in the evaluation) — github.com/SamanKhamesian/NutriGen
- **llamaapi** (Python client for Llama 3.1 8B inference used in the evaluation) — github.com/SamanKhamesian/NutriGen
- **pandas** (Summing per-item reported calories of generated meal plans) — github.com/SamanKhamesian/NutriGen

## Examples

```
pip install -r requirements.txt  # then in the notebook: replace "KEY" with a valid API key and execute
```

## Evaluation signals

- Environment resolves Python 3.10 with all pinned dependencies from requirements.txt installed without version conflicts
- Each of the 10 inputs yields 3 replicate meal plans per model, or an explicit failure record if a plan could not be generated
- Best-performing models (Llama 3.1 8B, GPT-3.5 Turbo) reproduce reported lowest percentage errors of 1.55% and 3.68% against caloric targets
- Failed generations are traceable and can be rendered as '-' placeholders in the downstream 10x10 error matrix

## Limitations

- DeepSeek V3 exhibited a higher MAE of 10.45% and was significantly slower in processing time compared to GPT-3.5 Turbo and Llama 3.1 8B
- Some model-input pairs fail entirely to produce a meal plan, yielding missing cells ('-' in the error matrix)
- API keys are provider-specific and must be individually valid; a single invalid 'KEY' placeholder blocks execution

## Evidence

- [methods] Replace 'KEY' placeholder with API keys before running the NutriGen Jupyter Notebook under Python 3.10 with pinned dependency versions.: "To install the necessary dependencies, run:

sh
pip install -r requirements.txt"
- [methods] Execute the Jupyter Notebook with a valid API key to generate meal plans.: "To run the NutriGen framework and generate meal plans, execute the provided Jupyter Notebook and make sure to replace "KEY" with a valid API key in the code before execution."
- [readme] Pinned dependency versions from requirements.txt.: "* anthropic==0.45.2
* google.generativeai==0.8.4
* llamaapi==0.1.36
* openai==1.42.0
* numpy==2.2.2
* pandas==2.2.3"
- [abstract] NutriGen is an LLM framework for personalized meal plans aligned with user-defined dietary preferences and constraints.: "we introduce NutriGen, a framework based on large language models (LLM) designed to generate personalized meal plans"
