---
name: option-combination-risk-categorization
description: Use when you have raw LLM chain-of-thought prediction outputs (e.g.,
  1748 predicted high-risk option combinations) and need to generalize them into ranked,
  human-readable risk categories of option combinations.
when_to_use_negative:
- Your goal is the prediction/fuzzing of option combinations itself (Constraint Extraction,
  Self Check, AutoCoT, Command Assembly) rather than post-hoc distillation of knowledge
  from prediction traces
- You need constraint relationships between individual options (conflicts/dependencies)
  — use the Constraint Extraction/Self Check pipeline instead
- Input traces are not CoT outputs from the High-Risk Combination Prediction step
  (e.g., they are command-assembly or file-generation logs)
edam_operation: http://edamontology.org/operation_0306
tools:
- name: GPT-4 Turbo
  role: LLM used both for the original prediction runs whose CoT is analyzed and for
    per-program summarization and cross-program synthesis/ranking of the 15 knowledge
    pieces
- name: Python
  role: Text parsing of the CoT outputs to extract prediction-related content and
    writing the ranked knowledge_table.csv
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: ProphetFuzz
  role: 'Source of the prediction traces: its High-Risk Combination Prediction component
    (llm_interface/predict.py, predictCombinations) generated the 1748 combinations
    analyzed'
  repo: https://github.com/NASP-THU/ProphetFuzz
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:5e1961f471f4b908545dbc985da6ce2632539d39ae6cd03331215483ae27367a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - GPT-4 Turbo
  - ProphetFuzz
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# option-combination-risk-categorization

## Summary

Distills a ranked table of 15 knowledge pieces for predicting high-risk command-line option combinations by summarizing GPT-4 Turbo chain-of-thought traces from ProphetFuzz's predictions across 52 evaluation programs, targeting security testers without security expertise.

## When to use

Use when you have raw LLM chain-of-thought prediction outputs (e.g., 1748 predicted high-risk option combinations) and need to generalize them into ranked, human-readable risk categories of option combinations.

## When NOT to use

- Your goal is the prediction/fuzzing of option combinations itself (Constraint Extraction, Self Check, AutoCoT, Command Assembly) rather than post-hoc distillation of knowledge from prediction traces
- You need constraint relationships between individual options (conflicts/dependencies) — use the Constraint Extraction/Self Check pipeline instead
- Input traces are not CoT outputs from the High-Risk Combination Prediction step (e.g., they are command-assembly or file-generation logs)

## Inputs

- Chain-of-thought (CoT) outputs from LLM prediction runs covering 1748 predicted high-risk option combinations
- Per-program prediction traces for 52 open-source programs from 40 packages (Groff man-page documentation as source)
- Prediction prompt structure (fourth step, High-Risk Combination Prediction) to delimit relevant trace content

## Outputs

- knowledge_table.csv with exactly 15 ranked rows: (knowledge item, supporting option-combination evidence)
- Top-three categories expected to match: Resource Management and Limits; Complex Data Processing; Output and Format Manipulation

## How to apply

Load the complete chain-of-thought (CoT) outputs recorded by GPT-4 Turbo during each ProphetFuzz prediction run, which produced 1748 predicted high-risk option combinations across the 52 evaluation programs from 40 packages. Parse only the content directly tied to the fourth step of the prediction prompt (the High-Risk Combination Prediction stage implemented in llm_interface/predict.py::predictCombinations) using Python-based text parsing. For each of the 52 programs, prompt GPT-4 Turbo to 'summarize the knowledge of the categories of program option combinations that may have potential vulnerabilities'. Compile the per-program summaries, have GPT-4 Turbo synthesize and rank the items from most to least common, and set the framing context to 'For software testers who do not understand security, they hope to have easy-to-understand knowledge to conduct efficient security testing'. Write the 15 ranked rows (knowledge item, supporting option-combination evidence) to knowledge_table.csv; the top-ranked categories in the paper were Resource Management and Limits, Complex Data Processing, and Output and Format Manipulation.

## Related tools

- **GPT-4 Turbo** (LLM used both for the original prediction runs whose CoT is analyzed and for per-program summarization and cross-program synthesis/ranking of the 15 knowledge pieces)
- **Python** (Text parsing of the CoT outputs to extract prediction-related content and writing the ranked knowledge_table.csv) — https://github.com/NASP-THU/ProphetFuzz
- **ProphetFuzz** (Source of the prediction traces: its High-Risk Combination Prediction component (llm_interface/predict.py, predictCombinations) generated the 1748 combinations analyzed) — https://github.com/NASP-THU/ProphetFuzz

## Examples

```
python distill_knowledge.py --cot-dir llm_interface/output --model gpt-4-turbo --out knowledge_table.csv  # 15 ranked rows: (knowledge_item, evidence)
```

## Evaluation signals

- knowledge_table.csv contains exactly 15 ranked rows
- Top three ranked items match the paper's reported categories: Resource Management and Limits; Complex Data Processing; Output and Format Manipulation
- Each knowledge row carries supporting option-combination evidence traceable to the per-program CoT summaries
- All 52 evaluation programs contributed a per-program summary before synthesis
- Rows are ordered most-to-least common as determined by the cross-program ranking prompt

## Limitations

- Knowledge is distilled from GPT-4 Turbo's own CoT for one prototype (about 3k lines of Python code on GPT-4 Turbo); LLM hallucination threatens constraint-extraction and prediction quality (challenge C1)
- Coverage is limited to the 52 programs collected from three related studies (POWER, Carpet Fuzz, Config Fuzz datasets); categories may not transfer to programs outside this corpus
- Ranking 'most to least common' reflects frequency within the analyzed traces, not measured vulnerability risk per combination

## Evidence

- [supplementary] Resource Management and Limits: [evidence span withheld — anchor status: not_found]
- [supplementary] Complex Data Processing: [evidence span withheld — anchor status: not_found]
