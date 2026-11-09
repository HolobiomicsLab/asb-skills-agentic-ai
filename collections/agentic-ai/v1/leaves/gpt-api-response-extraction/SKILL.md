---
name: gpt-api-response-extraction
description: Use when you have free-text LLM responses to medical MCQs (e.g., GPT-3.5
  or GPT-4 answers to 2022 ACG-SA questions) and need to convert each textual response
  into a discrete selected option for accuracy scoring.
when_to_use_negative:
- Responses are already structured as a clean single-letter option (no extraction
  needed — score directly against the answer key)
- You need evaluation beyond binary correctness (e.g., hallucination or error-source
  analysis) — the article notes information loss when evaluating responses as simply
  correct or incorrect
edam_operation: http://edamontology.org/operation_0306
tools:
- name: GPT-3.5
  role: Semi-automated extractor of the selected option from LLM textual responses
- name: Python 3.11
  role: Host language for the extraction, scoring, and pandas-based summary workflow
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes the GPT-3.5 API interactions during extraction
- name: OpenAI Python package
  role: Client library used to communicate with the OpenAI models
- name: Streamlit (1.33)
  role: Human-evaluation app used to adjudicate extraction disagreements
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:35cc9b2e7fc0698656351a0056ea49926a381fd10e2427c0843d5e328098ec66
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
  - LangChain (0.1.17 and 0.2.4)
  - OpenAI Python package
  - Python
  - Python 3.11
  - Streamlit (1.33)
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gpt-api-response-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A semi-automated pipeline that uses the GPT-3.5 API to extract the selected MCQ option from an LLM's free-text response, followed by human review of extraction disagreements, enabling correct/incorrect scoring of model outputs on the 2022 ACG-SA gastroenterology questions.

## When to use

Use when you have free-text LLM responses to medical MCQs (e.g., GPT-3.5 or GPT-4 answers to 2022 ACG-SA questions) and need to convert each textual response into a discrete selected option for accuracy scoring.

## When NOT to use

- Responses are already structured as a clean single-letter option (no extraction needed — score directly against the answer key)
- You need evaluation beyond binary correctness (e.g., hallucination or error-source analysis) — the article notes information loss when evaluating responses as simply correct or incorrect

## Inputs

- Raw free-text LLM response logs from GPT-3.5/GPT-4 API runs (e.g., three separate GPT-3.5 run outputs from the repository)
- 2022 ACG-SA answer key (300 questions) for downstream correct/incorrect scoring
- Prompt configuration metadata (simple 'just QA' prompt vs. multi-turn variants such as QA followed by 'are you sure?')

## Outputs

- Per-question extracted selected option (A/B/C/D/...) for each run
- Correct/incorrect scoring per question against the ACG-SA answer key
- Per-run overall accuracy values feeding run-to-run consistency analysis (e.g., consistency_summary.csv)

## How to apply

Load the LLM run outputs (e.g., the three GPT-3.5 API runs from the LLM-VLM-in-Gastroenterology repository) with Python 3.11 and pandas, then call the GPT-3.5 API to extract the selected option from each textual response, standardizing interactions with LangChain (0.1.17/0.2.4) and communicating via the OpenAI Python package. Flag responses where extraction is ambiguous and adjudicate only those disagreements through human review, supported by a lightweight review app built with Streamlit (1.33). Once a single selected option is assigned per question, score it against the 2022 ACG-SA answer key as correct/incorrect and compute per-run accuracy. The rationale is that free-text MCQ answers are not machine-scoreable directly, while fully manual extraction does not scale across 300 questions per ACG edition and multiple runs; the semi-automated design keeps the human in the loop only where the extractor is uncertain.

## Related tools

- **GPT-3.5** (Semi-automated extractor of the selected option from LLM textual responses)
- **Python 3.11** (Host language for the extraction, scoring, and pandas-based summary workflow)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes the GPT-3.5 API interactions during extraction)
- **OpenAI Python package** (Client library used to communicate with the OpenAI models)
- **Streamlit (1.33)** (Human-evaluation app used to adjudicate extraction disagreements)

## Examples

```
import pandas as pd; acc = [pd.read_csv(f"gpt35_run{i}.csv").correct.mean() for i in range(1,4)]; pd.DataFrame({'acc': acc}).to_csv('consistency_summary.csv')
```

## Evaluation signals

- Every question in each run has exactly one extracted option after human review — no unanswered/ambiguous entries remain
- Reproduced GPT-3.5 run-to-run accuracy variation under the fixed-seed, temperature-zero, simple-prompt configuration falls within approximately ±5 percentage points
- Scored per-run accuracies are reproducible against the 2022 ACG-SA answer key (300 questions) when re-running the scoring code
- Extraction disagreements are a small minority of responses, and all disagreements receive documented human review

## Limitations

- Extraction settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs; per-model prompt configuration is suggested as more effective
- Binary correct/incorrect evaluation loses information — an LLM may select the correct option yet hallucinate in its response
- Only a limited number of experimental runs were performed; no average and SD were reported to fully assess model consistency

## Evidence
