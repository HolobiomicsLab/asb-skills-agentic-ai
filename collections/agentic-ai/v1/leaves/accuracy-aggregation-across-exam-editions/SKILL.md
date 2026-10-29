---
name: accuracy-aggregation-across-exam-editions
description: Use when stored free-text LLM responses must be scored against an official
  MCQ answer key and summarized as accuracy per model and exam edition (e.g., ACG-SA
  2021/2022/2023) to test cross-edition consistency.
when_to_use_negative:
- The analysis goal is VLM performance on image-containing questions with human- or
  LLM-generated captions — this skill covers text-only MCQ accuracy aggregation only.
- You need consistency/variance estimates across repeated API runs; the paper's limited
  runs meant no average and SD were reported, so this skill produces single-run point
  accuracies.
- Responses lack extractable option letters and cannot be scored as correct/incorrect
  (cf. Meditron-7b-Q16 and HippoMistral-7b-Q16, excluded for incoherent responses
  on >50% of questions).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3379
- http://edamontology.org/topic_3474
tools:
- name: Python 3.11
  role: Language used for API interactions and data processing; pandas is used to
    load and aggregate the stored responses
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes interactions with the models when running/extracting responses
- name: OpenAI Python package
  role: Used for the GPT-3.5 semi-automated option-extraction step from free-text
    responses
- name: Sdamirsa/LLM-VLM-in-Gastroenterology
  role: Repository deposit supplying the stored model responses for the 2021–2023
    ACG-SA text-only question sets
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:90748f297f17ebbfabdb79f88b398a698a93857eb19d3750f9300f45a33a3c86
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  tools:
  - LangChain (0.1.17 and 0.2.4)
  - OpenAI Python package
  - Python
  - Python 3.11
  - Sdamirsa/LLM-VLM-in-Gastroenterology
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3379
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# accuracy-aggregation-across-exam-editions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-question correctness of LLM responses into accuracy percentages for each model-edition pair across multiple years of the ACG Self-Assessment examination, enabling a sensitivity analysis of whether model performance degrades on newer exam editions.

## When to use

Use when stored free-text LLM responses must be scored against an official MCQ answer key and summarized as accuracy per model and exam edition (e.g., ACG-SA 2021/2022/2023) to test cross-edition consistency.

## When NOT to use

- The analysis goal is VLM performance on image-containing questions with human- or LLM-generated captions — this skill covers text-only MCQ accuracy aggregation only.
- You need consistency/variance estimates across repeated API runs; the paper's limited runs meant no average and SD were reported, so this skill produces single-run point accuracies.
- Responses lack extractable option letters and cannot be scored as correct/incorrect (cf. Meditron-7b-Q16 and HippoMistral-7b-Q16, excluded for incoherent responses on >50% of questions).

## Inputs

- Stored GPT-4-0613 and GPT-3.5-0125 free-text responses for ACG-SA 2021, 2022, and 2023 text-only question sets (from the LLM-VLM-in-Gastroenterology repository deposit)
- Official ACG-SA answer keys for the 2021, 2022, and 2023 editions

## Outputs

- fig7a_gpt_accuracy.csv with 6 rows containing model, edition, and accuracy fields
- fig7a_gpt_accuracy.png grouped bar chart matching the paper's Fig. 7a layout

## How to apply

Load the stored model responses (e.g., GPT-4-0613 and GPT-3.5-0125 answers to the ACG-SA text-only question sets for 2021, 2022, and 2023) from the LLM-VLM-in-Gastroenterology repository deposit using Python 3.11 with pandas. Extract the selected option letter from each free-text response using the paper's semi-automated GPT-3.5-based option-extraction step, then score each response correct/incorrect against the official ACG-SA answer key. Aggregate per-question correctness into accuracy percentages for each of the six model-edition pairs, targeting the reported values GPT-4-0613: 71.59%, 65.38%, 70.35% and GPT-3.5-0125: 44.57%, 46.30%, 53.49%. Write the (model, edition, accuracy) rows to a CSV (fig7a_gpt_accuracy.csv) and render a grouped bar chart (fig7a_gpt_accuracy.png) matching the paper's Fig. 7a layout. The rationale is that performance degradation on the recently published 2023 dataset could suggest inclusion of previous exam datasets in model training data.

## Related tools

- **Python 3.11** (Language used for API interactions and data processing; pandas is used to load and aggregate the stored responses)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes interactions with the models when running/extracting responses)
- **OpenAI Python package** (Used for the GPT-3.5 semi-automated option-extraction step from free-text responses)
- **Sdamirsa/LLM-VLM-in-Gastroenterology** (Repository deposit supplying the stored model responses for the 2021–2023 ACG-SA text-only question sets) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology

## Examples

```
python score_acg_editions.py --models GPT-4-0613,GPT-3.5-0125 --editions 2021,2022,2023 --out fig7a_gpt_accuracy.csv --plot fig7a_gpt_accuracy.png
```

## Evaluation signals

- Each computed accuracy must equal the reported Fig. 7a value within 0.1 percentage points (GPT-4-0613: 71.59%, 65.38%, 70.35%; GPT-3.5-0125: 44.57%, 46.30%, 53.49%).
- The output CSV must contain exactly 6 rows, one per model-edition pair, with model and edition fields present.
- Each per-question response must resolve to a single extracted option letter and a binary correct/incorrect score against the official answer key (no unscored rows).
- The rendered bar chart must be a grouped layout with the two models compared across the three exam editions, matching the paper's Fig. 7a.
- Accuracy ordering should be consistent with the paper's trend analysis: any substantial degradation on the 2023 edition flags possible training-data inclusion of earlier editions.

## Limitations

- Only text-only questions (162 text-only vs 138 image-inclusive in the 2022 ACG-SA) are covered; image-inclusive questions are excluded from this aggregation.
- The option-extraction and evaluation step reduces responses to binary correct/incorrect, which the authors note causes potential information loss (e.g., hallucinations even when the correct option is selected).
- Limited number of experimental runs: no average and SD were reported, so model consistency cannot be assessed from a single aggregation.
- ACG-SA questions are paywalled, which the authors used to argue the exams are likely absent from LLM training data — the sensitivity-analysis interpretation rests on that assumption.

## Evidence

- [results] GPT-4-0613 scored 71.59%, 65.38%, and 70.35% and GPT-3.5-0125 scored 44.57%, 46.30%, and 53.49% on the 2021, 2022, and 2023 ACG-SA text-only question sets: [evidence span withheld — anchor status: not_found]
- [discussion] This served as an indirect sensitivity analysis, as any degradation in performance on the recently published 2023 dataset could suggest the inclusion of previous datasets: "This served as an indirect sensitivity analysis, as any degradation in performance on the recently published 2023 dataset could suggest the inclusion of previous datasets"
- [discussion] we used the GPT-3.5 API to extract the selected option from the LLM textual response: "we used the GPT-3.5 API to extract the selected option from the LLM textual response"
- [discussion] We used the 2021, 2022, and 2023 editions of the ACG-SA. Each edition has 300 questions: "We used the 2021, 2022, and 2023 editions of the ACG-SA. Each edition has 300 questions"
- [discussion] Another limitation of our study is the limited number of experimental runs.: "Another limitation of our study is the limited number of experimental runs."
