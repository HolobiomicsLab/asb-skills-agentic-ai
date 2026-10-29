---
name: llm-benchmark-reproducibility
description: Use when reproducing or validating LLM accuracy figures across multiple
  exam editions (e.g., GPT-4-0613 and GPT-3.5-0125 on ACG-SA 2021–2023 text-only MCQ
  sets), or when checking that a stored-response benchmark matches reported numbers.
when_to_use_negative:
- You are scoring image-inclusive questions — this skill applies only to text-only
  question sets; image-based questions require the VLM workflow (no-image, human caption,
  VLM caption, or direct image scenarios).
- You need per-run variance or consistency estimates — the paper reports limited experimental
  runs with no average and SD, so this skill cannot establish model consistency.
- You lack access to the ACG-SA answer keys or stored responses — examination access
  is restricted by a paywall, so the scoring step cannot be executed.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Python 3.11
  role: Environment used for API interactions and response processing; load stored
    responses and aggregate accuracies with pandas
  repo: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: OpenAI Python package
  role: Used to call the GPT-3.5 API for semi-automated extraction of the selected
    option letter from free-text LLM responses
  repo: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardized interactions with the GPT models whose stored responses are scored
    in the sensitivity analysis
  repo: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:81f2f32afe223e13a633df9203c0b7ba0fd11bc8db0523045a62ce2a525449e5
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# llm-benchmark-reproducibility

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the paper's GPT model-generation and exam-year sensitivity analysis (Fig. 7a) by scoring stored GPT-4-0613 and GPT-3.5-0125 responses against official ACG-SA answer keys for the 2021, 2022, and 2023 text-only question sets, and validating computed accuracies against the reported values (GPT-4-0613: 71.59%/65.38%/70.35%; GPT-3.5-0125: 44.57%/46.30%/53.49%).

## When to use

Use when reproducing or validating LLM accuracy figures across multiple exam editions (e.g., GPT-4-0613 and GPT-3.5-0125 on ACG-SA 2021–2023 text-only MCQ sets), or when checking that a stored-response benchmark matches reported numbers.

## When NOT to use

- You are scoring image-inclusive questions — this skill applies only to text-only question sets; image-based questions require the VLM workflow (no-image, human caption, VLM caption, or direct image scenarios).
- You need per-run variance or consistency estimates — the paper reports limited experimental runs with no average and SD, so this skill cannot establish model consistency.
- You lack access to the ACG-SA answer keys or stored responses — examination access is restricted by a paywall, so the scoring step cannot be executed.

## Inputs

- Stored GPT-4-0613 free-text responses for ACG-SA 2021, 2022, and 2023 text-only question sets (from the LLM-VLM-in-Gastroenterology repository deposit)
- Stored GPT-3.5-0125 free-text responses for ACG-SA 2021, 2022, and 2023 text-only question sets
- Official ACG-SA answer keys for the 2021, 2022, and 2023 editions (300 questions per edition)

## Outputs

- fig7a_gpt_accuracy.csv — 6 rows with model, edition, and accuracy fields
- fig7a_gpt_accuracy.png — grouped bar chart matching the layout of the paper's Fig. 7a

## How to apply

Load the stored GPT-4-0613 and GPT-3.5-0125 free-text responses for the ACG-SA 2021, 2022, and 2023 text-only question sets from the LLM-VLM-in-Gastroenterology deposit using Python 3.11 with pandas. Mirror the paper's semi-automated evaluation by extracting the selected option letter from each free-text response (the paper used the GPT-3.5 API for this extraction step with human review) and score each response as correct or incorrect against the official ACG-SA answer key. Aggregate per-question correctness into accuracy percentages for each of the six model-edition pairs, targeting the reported values GPT-4-0613: 71.59%, 65.38%, 70.35% and GPT-3.5-0125: 44.57%, 46.30%, 53.49%. Write the six-value (model, edition, accuracy) summary to fig7a_gpt_accuracy.csv and render a grouped bar chart as fig7a_gpt_accuracy.png matching the layout of the paper's Fig. 7a. The year-over-year comparison serves as an indirect sensitivity analysis: degradation on the recently published 2023 edition could suggest inclusion of previous datasets in model training.

## Related tools

- **Python 3.11** (Environment used for API interactions and response processing; load stored responses and aggregate accuracies with pandas) — https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **OpenAI Python package** (Used to call the GPT-3.5 API for semi-automated extraction of the selected option letter from free-text LLM responses) — https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **LangChain (0.1.17 and 0.2.4)** (Standardized interactions with the GPT models whose stored responses are scored in the sensitivity analysis) — https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology

## Examples

```
import pandas as pd; df = pd.read_csv('stored_gpt_responses.csv'); acc = df.groupby(['model','edition'])['correct'].mean().mul(100).round(2).reset_index(); acc.to_csv('fig7a_gpt_accuracy.csv', index=False)
```

## Evaluation signals

- Each computed accuracy must equal the reported Fig. 7a value within 0.1 percentage points: GPT-4-0613 at 71.59% (2021), 65.38% (2022), 70.35% (2023) and GPT-3.5-0125 at 44.57% (2021), 46.30% (2023 editions), 53.49% (2023).
- fig7a_gpt_accuracy.csv contains exactly 6 rows (2 models x 3 editions) with non-empty model and edition fields.
- Every stored free-text response yields a parseable option letter (via the GPT-3.5-based extraction mirroring the paper's semi-automated pipeline with human review).
- Grouped bar chart fig7a_gpt_accuracy.png visually matches the paper's Fig. 7a layout with six bars grouped by model and edition.
- Per-edition accuracy is computed only over the text-only question subset (162 text-only questions in the 2022 ACG-SA main evaluation), excluding image-inclusive questions.

## Limitations

- Limited number of experimental runs; no average and SD are reported, so model consistency cannot be assessed from this analysis.
- Access to the ACG-SA examinations and corresponding answers is restricted by a paywall, limiting third-party reproduction of the scoring step.
- Prompt/settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs; the GPT-3.5-derived configuration's generalizability is uncertain.
- Responses are evaluated as simply correct or incorrect, which loses information and does not capture hallucinations in the response text.

## Evidence
