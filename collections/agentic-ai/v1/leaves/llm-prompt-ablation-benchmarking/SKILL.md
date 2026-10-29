---
name: llm-prompt-ablation-benchmarking
description: Use when tuning LLM answering settings (structured vs free-text output,
  prompt variant, temperature, max tokens) for MCQ clinical-reasoning benchmarks,
  or when reproducing the paper's Fig. 1 parameter-optimization results.
when_to_use_negative:
- The goal is whole-model benchmarking across LLMs/VLMs (Experiments 1–2), not tuning
  output format, prompt, temperature, or max-token settings for a single model
- The question set is image-inclusive; Experiment 0 parameter optimization was run
  on text-only questions with image-inclusive questions excluded
- Reusing the GPT-3.5-derived configuration on a different LLM without re-optimizing
  — the article warns its generalizability is untested
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: GPT-3.5
  role: Model queried across the settings grid, and also used in the extraction pipeline
    to pull the selected option from textual responses
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: OpenAI Python package
  role: API communication with GPT-3.5 for all configurations of the ablation grid
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardized LLM API interactions during the parameter sweep
- name: Python 3.11
  role: Runtime for the ablation workflow, scoring, and tabulation of results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:50554bef49cf942388acb7fd3cd6aaa47cd2da445de3592afaecd2ab831ca378
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  tools:
  - GPT-3.5
  - LangChain
  - LangChain (0.1.17 and 0.2.4)
  - OpenAI Python package
  - Python
  - Python 3.11
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# llm-prompt-ablation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the paper's Experiment 0 parameter-optimization ablation: sweep output-format, prompt-engineering, temperature, and max-token settings with GPT-3.5 on a 60-question ACG-SA gastroenterology MCQ subset to identify the best-performing model configuration.

## When to use

Use when tuning LLM answering settings (structured vs free-text output, prompt variant, temperature, max tokens) for MCQ clinical-reasoning benchmarks, or when reproducing the paper's Fig. 1 parameter-optimization results.

## When NOT to use

- The goal is whole-model benchmarking across LLMs/VLMs (Experiments 1–2), not tuning output format, prompt, temperature, or max-token settings for a single model
- The question set is image-inclusive; Experiment 0 parameter optimization was run on text-only questions with image-inclusive questions excluded
- Reusing the GPT-3.5-derived configuration on a different LLM without re-optimizing — the article warns its generalizability is untested

## Inputs

- 60-question ACG-SA gastroenterology MCQ subset with answer key (from the project repository)
- Grid of model settings: output-format variants (structured vs free-text), prompt variants (Just QA, answer-and-justify, combined 5-technique prompt), temperature values, max-token limits
- Access credentials for the GPT-3.5 API via the OpenAI Python package

## Outputs

- parameter_optimization_results.csv with per-setting accuracies
- Summary figure reproducing Fig. 1 (parameter-optimization results)
- Accuracy deltas vs baseline per factor (structured output, prompt variant, temperature, max tokens) with 95% CIs

## How to apply

Load the 60-question ACG-SA sample and answer key from github.com/Sdamirsa/LLM-VLM-in-Gastroenterology using Python 3.11 with LangChain (0.1.17 and 0.2.4) and the OpenAI Python package. Query GPT-3.5 across the grid of variants: structured vs free-text output format; prompt variants including Just QA, answer-and-justification, QA followed by 'are you sure?', QA + 'answer following question', and the best combined prompt (all 5 techniques); plus temperature and max-token settings. Extract the selected option from each free-text response via the GPT-3.5 extraction pipeline (semi-automated, with human review) and score accuracy per configuration against the answer key. Compare accuracy deltas versus baseline: structured output generation gives +5–10%, the best combined prompt +5.00% (95% CI 1.11–7.78%), and answer-and-justification +4.44% (2.78–6.11%), the largest single-technique gain. Tabulate per-setting accuracies, render the summary figure matching Fig. 1 and Supplementary Tables 3–6, and save parameter_optimization_results.csv.

## Related tools

- **GPT-3.5** (Model queried across the settings grid, and also used in the extraction pipeline to pull the selected option from textual responses) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **OpenAI Python package** (API communication with GPT-3.5 for all configurations of the ablation grid)
- **LangChain (0.1.17 and 0.2.4)** (Standardized LLM API interactions during the parameter sweep)
- **Python 3.11** (Runtime for the ablation workflow, scoring, and tabulation of results)

## Examples

```
python run_parameter_optimization.py --model gpt-3.5 --prompts just_qa,answer_justify,combined --sweep temperature,max_tokens --output parameter_optimization_results.csv
```

## Evaluation signals

- Reproduced best-combined-prompt effect equals +5.00% with 95% CI 1.11–7.78%
- Reproduced answer-and-justification single-technique effect equals +4.44% (95% CI 2.78–6.11%)
- Structured output generation yields a 5–10% accuracy gain over free-text responses
- Per-configuration accuracies scored against the ACG-SA answer key and tabulated consistently with Fig. 1 and Supplementary Tables 3–6
- parameter_optimization_results.csv is produced with one accuracy row per settings combination

## Limitations

- The prompt/settings configuration was optimized only for OpenAI (GPT-3.5) and reused for other LLMs; per-model prompt configuration is suggested as more effective and generalizability is uncertain
- Limited number of experimental runs (three GPT-3.5 API runs in the consistency experiment); no average and SD reported to assess model consistency
- Semi-automated extraction plus human review may introduce information loss when responses are scored simply as correct or incorrect

## Evidence
