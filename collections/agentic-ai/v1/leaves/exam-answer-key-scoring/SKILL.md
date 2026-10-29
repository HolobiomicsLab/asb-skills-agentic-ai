---
name: exam-answer-key-scoring
description: Use when stored free-text LLM answers to MCQ exam sets (e.g., ACG-SA
  2021/2022/2023) must be converted into binary correctness scores and accuracy summaries
  against an official answer key.
when_to_use_negative:
- Evaluating image-inclusive ACG-SA questions, which the paper's text-only scoring
  excluded
- Assessing response quality beyond option selection (e.g., hallucinations or justification
  logic), since correct/incorrect scoring loses that information
- Datasets whose official answer key is not accessible — the paper notes ACG-SA answers
  are restricted by a paywall, so scoring cannot proceed without the key
tools:
- name: Python 3.11
  role: Runtime for loading stored responses with pandas and aggregating per-question
    correctness into accuracy percentages
  repo: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: OpenAI Python package
  role: Used for GPT-3.5 API calls in the semi-automated option-extraction step of
    the evaluation pipeline
  repo: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardized the LLM API interactions whose stored responses are subsequently
    scored
  repo: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:297fcb89e2452c018c577cd30ce910b2608e78c8bbb36404c02b4817917f4af2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  repo_url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  tools:
  - LangChain (0.1.17 and 0.2.4)
  - OpenAI Python package
  - Python
  - Python 3.11
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# exam-answer-key-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores free-text LLM responses to ACG Self-Assessment (ACG-SA) multiple-choice questions by extracting the selected option letter and judging it correct/incorrect against the official answer key, yielding per-question correctness and model-edition accuracy percentages.

## When to use

Use when stored free-text LLM answers to MCQ exam sets (e.g., ACG-SA 2021/2022/2023) must be converted into binary correctness scores and accuracy summaries against an official answer key.

## When NOT to use

- Evaluating image-inclusive ACG-SA questions, which the paper's text-only scoring excluded
- Assessing response quality beyond option selection (e.g., hallucinations or justification logic), since correct/incorrect scoring loses that information
- Datasets whose official answer key is not accessible — the paper notes ACG-SA answers are restricted by a paywall, so scoring cannot proceed without the key

## Inputs

- Stored free-text LLM responses to the 162 text-only ACG-SA questions per edition (2021, 2022, 2023) from the Sdamirsa/LLM-VLM-in-Gastroenterology deposit
- Official ACG-SA answer keys for the 2021, 2022, and 2023 editions

## Outputs

- fig7a_gpt_accuracy.csv with 6 rows (model, edition, accuracy) covering GPT-4-0613 and GPT-3.5-0125 across three exam years
- fig7a_gpt_accuracy.png grouped bar chart matching the paper's Fig. 7a layout
- Per-question binary correctness labels (correct/incorrect) for each model-edition pair

## How to apply

Load the stored model responses (e.g., GPT-4-0613 and GPT-3.5-0125 on the 2021, 2022, and 2023 ACG-SA text-only question sets) from the LLM-VLM-in-Gastroenterology repository deposit using Python 3.11 with pandas. Mirror the paper's semi-automated evaluation pipeline: use the GPT-3.5 API to extract the selected option from each LLM textual response, then apply human review as a check on extraction. Score each extracted option as correct or incorrect against the official ACG-SA answer key, aggregating per-question correctness into accuracy percentages per model-edition pair (target: GPT-4-0613 at 71.59%/65.38%/70.35% and GPT-3.5-0125 at 44.57%/46.30%/53.49% on the 2021/2022/2023 editions). Write the six-value (model, edition, accuracy) summary to CSV and optionally render a grouped bar chart matching the paper's Fig. 7a layout. The rationale is that MCQ accuracy against a paywalled answer key is the benchmark metric used to detect performance degradation on newer exam editions as an indirect sensitivity analysis.

## Related tools

- **Python 3.11** (Runtime for loading stored responses with pandas and aggregating per-question correctness into accuracy percentages) — https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **OpenAI Python package** (Used for GPT-3.5 API calls in the semi-automated option-extraction step of the evaluation pipeline) — https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **LangChain (0.1.17 and 0.2.4)** (Standardized the LLM API interactions whose stored responses are subsequently scored) — https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology

## Examples

```
python score_acg_sa.py --responses repo/responses/ --answer-key acg_sa_key_2021.csv --out fig7a_gpt_accuracy.csv --chart fig7a_gpt_accuracy.png
```

## Evaluation signals

- Each computed accuracy equals the reported Fig. 7a value within 0.1 percentage points (GPT-4-0613: 71.59/65.38/70.35; GPT-3.5-0125: 44.57/46.30/53.49 for 2021/2022/2023)
- fig7a_gpt_accuracy.csv contains exactly 6 rows with non-empty model and edition fields
- Option-letter extraction from free-text responses succeeds for the expected proportion of the 162 text-only questions per edition (human review catches extraction failures)
- Per-question correctness is binary and every scored question maps to exactly one answer-key entry
- Rendered bar chart reproduces the grouped model-x-edition layout of the paper's Fig. 7a

## Limitations

- Semi-automated GPT-3.5-based extraction may mis-parse free-text responses; human review is required, and evaluating responses as simply correct or incorrect causes information loss
- Model knowledge cutoffs changed during experimentation without documented notification, which could confound cross-edition comparisons
- The limited number of experimental runs means no average and SD are reported to assess model consistency

## Evidence
