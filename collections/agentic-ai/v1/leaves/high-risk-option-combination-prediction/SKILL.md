---
name: high-risk-option-combination-prediction
description: Use when you have parsed Groff man-page content (program name, description,
  synopsis, option descriptions) plus an extracted option-constraint list, and need
  to prioritize which option combinations of a CLI program are most likely to trigger
  vulnerabilities.
when_to_use_negative:
- The program's documentation is not available as Groff man pages (or equivalent parseable
  option descriptions), since prediction relies on documentation alone
- 'No constraint list exists yet: predicting before the Section 3.2 constraint extraction
  and bidirectional-reasoning self-check risks flagging combinations that violate
  conflicts/dependencies (Challenge C1)'
- You need command execution or fuzzing artifacts (files, assembled commands) — this
  skill only predicts combinations; command assembly and file generation are downstream
  steps (Sections 3.4–3.5)
tools:
- name: ProphetFuzz
  role: Implements the prediction workflow; predictCombinations in llm_interface/predict.py
    performs high-risk combination prediction (Section 3.3)
  repo: https://deepwiki.com/NASP-THU/ProphetFuzz
- name: GPT-4 Turbo
  role: LLM backend (gpt-4-1106-preview) queried with temperature 0.7 and n=10 inferences
    per program for diverse prediction outcomes
- name: Python
  role: Implementation language of the ~3k-line ProphetFuzz prototype, including the
    prediction scripts and JSON record writing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:76adb343ffc3548da81612d061bf2cebf0c4b566c23604008de8610cf9038ccd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GPT-4 Turbo
  - OpenAI API
  - ProphetFuzz
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# high-risk-option-combination-prediction

## Summary

LLM-driven prediction of high-risk command-line option combinations using only parsed program documentation (Groff man pages), as implemented in ProphetFuzz. Predicted combinations are deduplicated per program and later validated by 72-hour fuzzing against ground-truth vulnerabilities.

## When to use

Use when you have parsed Groff man-page content (program name, description, synopsis, option descriptions) plus an extracted option-constraint list, and need to prioritize which option combinations of a CLI program are most likely to trigger vulnerabilities.

## When NOT to use

- The program's documentation is not available as Groff man pages (or equivalent parseable option descriptions), since prediction relies on documentation alone
- No constraint list exists yet: predicting before the Section 3.2 constraint extraction and bidirectional-reasoning self-check risks flagging combinations that violate conflicts/dependencies (Challenge C1)
- You need command execution or fuzzing artifacts (files, assembled commands) — this skill only predicts combinations; command assembly and file generation are downstream steps (Sections 3.4–3.5)

## Inputs

- Parsed Groff man-page content per program (name, description, synopsis, option descriptions)
- Self-checked option constraint list (conflicts and dependencies) from the Section 3.2 extraction step
- Auto CoT few-shot examples generated once with n=1 (e.g., manpage_htmldoc., manpage_jhead., manpage_yasm.)
- OpenAI API key configured in llm_interface/config/.env

## Outputs

- Per-combination JSON records with fields: program, option combination, rationale
- Consolidated, deduplicated set of predicted high-risk option combinations per program (1748 across the 52-program dataset)

## How to apply

Load each program's parsed Groff man-page content and its self-checked constraint list (extracted via the LLM and filtered by bidirectional-reasoning self-check, Section 3.2). Build a multi-step chain-of-thought prediction prompt embedding those constraints so the LLM avoids flagging combinations that violate conflicts/dependencies, and attach Auto CoT few-shot examples generated once with n=1 (few-shot_generate.py, generatePrompt). Query GPT-4 Turbo (gpt-4-1106-preview) via the OpenAI API with temperature 0.7 and parameter n=10, so the model makes ten diverse inferences per program (llm_interface/predict.py, predictCombinations). Consolidate the ten inferences into a deduplicated set of predicted high-risk option combinations and write one JSON record per combination containing program, option combination, and rationale. Across the paper's 52-program dataset this procedure yielded 1748 predictions at an average cost of $8.69 per program.

## Related tools

- **ProphetFuzz** (Implements the prediction workflow; predictCombinations in llm_interface/predict.py performs high-risk combination prediction (Section 3.3)) — https://deepwiki.com/NASP-THU/ProphetFuzz
- **GPT-4 Turbo** (LLM backend (gpt-4-1106-preview) queried with temperature 0.7 and n=10 inferences per program for diverse prediction outcomes)
- **Python** (Implementation language of the ~3k-line ProphetFuzz prototype, including the prediction scripts and JSON record writing)

## Examples

```
bash run_all_in_one.sh bison
```

## Evaluation signals

- Every per-combination record parses as well-formed JSON with the expected fields (program, option combination, rationale)
- Consolidated prediction count across the 52 programs reaches approximately the reported 1748 combinations
- Predicted combinations do not violate extracted constraints (conflicts/dependencies), confirming constraints were correctly embedded in the prompt
- Downstream validation: 12.30% of predicted combinations were associated with 364 unique vulnerabilities found during 72 hours of fuzzing, 32.85% more effective than state-of-the-art
- Average LLM cost stays near the reported $8.69 per program, indicating comparable prompt/API usage

## Limitations

- LLM hallucination threatens constraint-extraction precision, so combinations violating real constraints may still be flagged if the constraint list is incomplete (Challenge C1)
- Predictions are stochastic: temperature 0.7 with n=10 inferences requires deduplication; counts may vary across runs and model versions (gpt-4-1106-preview may be deprecated)
- Cost scales with the number of programs (~$8.69 per program); the paper's effectiveness figure (32.85% over state-of-the-art) is specific to its 52-program dataset and 72-hour fuzzing budget

## Evidence
