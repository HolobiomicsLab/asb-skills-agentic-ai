---
name: benchmark-result-visualization
description: Use when per-question model correctness data (e.g., GPT-4/GPT-3.5 responses
  scored against an ACG-SA answer key) must be aggregated into accuracy percentages
  and visualized as grouped bar charts matching a paper figure such as Fig. 1 or Fig.
  7a.
when_to_use_negative:
- The task is scoring or extracting model options from raw free-text responses rather
  than visualizing already-scored correctness data (use the option-extraction/scoring
  step first).
- The analysis targets image-containing question subsets with captions or direct image
  inputs (VLM Experiment 2 scenarios), which require a different figure/comparison
  design than the text-only accuracy bars.
- The goal is statistical re-analysis (confidence intervals, significance of deltas)
  rather than reproducing reported figure values; the paper reports limited runs with
  no average and SD for consistency.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python 3.11
  role: Implementation language for loading responses, aggregating accuracy, and rendering
    the bar chart with pandas
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- name: OpenAI Python package
  role: Interface for the GPT-3.5 option-extraction step that converts free-text responses
    into scorable option letters before aggregation and plotting
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes LLM API interactions when re-querying GPT-3.5 for the parameter-optimization
    configurations summarized in Fig. 1
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:f6ecaa39fd5ca37bc97a2f8b765c5771842962cae064c3ad0e02adf3867aa3dc
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# benchmark-result-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Renders LLM/VLM benchmark accuracy results into publication-style summary figures (e.g., grouped bar charts of model accuracy by exam edition or prompt setting) paired with machine-readable CSV tables, mirroring the figures of the Gastroenterology LLM/VLM benchmarking paper.

## When to use

Use when per-question model correctness data (e.g., GPT-4/GPT-3.5 responses scored against an ACG-SA answer key) must be aggregated into accuracy percentages and visualized as grouped bar charts matching a paper figure such as Fig. 1 or Fig. 7a.

## When NOT to use

- The task is scoring or extracting model options from raw free-text responses rather than visualizing already-scored correctness data (use the option-extraction/scoring step first).
- The analysis targets image-containing question subsets with captions or direct image inputs (VLM Experiment 2 scenarios), which require a different figure/comparison design than the text-only accuracy bars.
- The goal is statistical re-analysis (confidence intervals, significance of deltas) rather than reproducing reported figure values; the paper reports limited runs with no average and SD for consistency.

## Inputs

- Stored model responses (free-text answers) for the ACG-SA 2021/2022/2023 text-only question sets from the LLM-VLM-in-Gastroenterology repository deposit
- Official ACG-SA answer key for the 300-question editions (162 text-only and 138 image-inclusive questions in the 2022 edition)
- Per-configuration accuracy results from the Experiment Zero parameter-optimization grid (structured vs free-text output, prompt variants, temperature, max-token)

## Outputs

- fig7a_gpt_accuracy.csv — 6-row summary table with model, edition, and accuracy fields
- fig7a_gpt_accuracy.png — grouped bar chart of model accuracy across exam editions matching the paper's Fig. 7a layout
- parameter_optimization_results.csv — per-setting accuracy table with deltas versus baseline for the Fig. 1 / Supplementary Tables 3-6 rendering

## How to apply

Load the stored model responses (e.g., GPT-4-0613 and GPT-3.5-0125 answers to the 2021, 2022, and 2023 ACG-SA text-only question sets) from the Sdamirsa/LLM-VLM-in-Gastroenterology deposit with Python 3.11 and pandas. Extract the selected option letter from each free-text response using the paper's semi-automated GPT-3.5-based extraction step, then score each response correct/incorrect against the official ACG-SA answer key. Aggregate per-question correctness into accuracy percentages per model-edition pair (Fig. 7a targets: GPT-4-0613 71.59%, 65.38%, 70.35%; GPT-3.5-0125 44.57%, 46.30%, 53.49%) or per prompt/temperature/max-token configuration (Fig. 1 / Supplementary Tables 3-6). Write the summary values to a CSV such as fig7a_gpt_accuracy.csv with model, edition, and accuracy fields, and render a grouped bar chart (e.g., fig7a_gpt_accuracy.png) matching the layout of the corresponding paper figure. The rationale is that the benchmark's findings — parameter effects like structured output gains of 5-10% and the +5.00% (95% CI 1.11-7.78%) combined-prompt effect — are communicated only through these figures, so faithful aggregation and rendering are required for validation.

## Related tools

- **Python 3.11** (Implementation language for loading responses, aggregating accuracy, and rendering the bar chart with pandas) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
- **OpenAI Python package** (Interface for the GPT-3.5 option-extraction step that converts free-text responses into scorable option letters before aggregation and plotting)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes LLM API interactions when re-querying GPT-3.5 for the parameter-optimization configurations summarized in Fig. 1)

## Examples

```
import pandas as pd; df = pd.read_csv('fig7a_gpt_accuracy.csv'); assert len(df) == 6; df.groupby(['model','edition']).accuracy.plot(kind='bar')
```

## Evaluation signals

- Each computed accuracy equals the reported Fig. 7a value within 0.1 percentage points (GPT-4-0613: 71.59/65.38/70.35; GPT-3.5-0125: 44.57/46.30/53.49).
- fig7a_gpt_accuracy.csv contains exactly 6 rows with model and edition fields present and non-empty.
- The combined-prompt effect reproduces as +5.00% (95% CI 1.11-7.78%) and answer-and-justify as +4.44% in the parameter-optimization summary.
- Structured output configurations show a 5-10% accuracy gain over free-text responses before rendering.
- The rendered grouped bar chart visually matches the layout of the paper's Fig. 7a (models grouped by exam edition).

## Limitations

- Only a limited number of experimental runs were performed; no average and SD were reported to assess model consistency, so the figures show point estimates without dispersion.
- Quantized/local models excluded for incoherent responses (Meditron-7b-Q16, HippoMistral-7b-Q16) will be absent from any reproduced figure.
- Evaluating responses as simply correct or incorrect loses information, and MCQ accuracy does not capture hallucinations or real-world usage, constraining what the visualized metric means.
- The GitHub README states dataset and performance details are withheld pending publication ('This will be published after the paper is published'), so inputs may not be fully available from the repository alone.

## Evidence

- [other] GPT-4-0613 scored 71.59%, 65.38%, and 70.35% and GPT-3.5-0125 scored 44.57%, 46.30%, and 53.49% on the 2021, 2022, and 2023 ACG-SA text-only question sets, respectively (Fig. 7a).: [evidence span withheld — anchor status: not_found]
- [discussion] we used the GPT-3.5 API to extract the selected option from the LLM textual response: "we used the GPT-3.5 API to extract the selected option from the LLM textual response"
- [other] Structured output generation enhanced performance by 5–10%, and the best combined prompt (all 5 techniques) increased performance by 5.00% (95% CI: 1.11% to 7.78%): [evidence span withheld — anchor status: not_found]
- [discussion] Another limitation of our study is the limited number of experimental runs.: "Another limitation of our study is the limited number of experimental runs."
- [discussion] We used Python version 3.11 for API interactions: "We used Python version 3.11 for API interactions"
