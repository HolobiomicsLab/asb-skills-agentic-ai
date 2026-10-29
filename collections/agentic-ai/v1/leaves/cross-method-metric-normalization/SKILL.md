---
name: cross-method-metric-normalization
description: Use when a paper reports results under one metric (e.g. LLM-as-a-Judge
  accuracy/precision) while baselines report incompatible metrics (Hit@1, F1) on different
  dataset samples and a comparison table must be built honestly.
when_to_use_negative:
- All compared systems were evaluated on the identical dataset sample under the same
  metric, so no normalization caveats are needed
- The goal is to reproduce experiments rather than tabulate reported numbers — use
  eval.py with INDEX/MODEL/DATASET instead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
tools:
- name: Qwen2.5-72B
  role: Focal-system model whose reported scores (P=85.2%, A=78.9 on Mintaka) populate
    the ReFactX rows of the comparison table
  repo: https://github.com/rpo19/ReFactX
- name: Llama3.3-70B
  role: Focal-system model whose reported scores (P=77.8%, A=70.6) populate ReFactX
    rows and Mintaka per-question-type breakdowns
  repo: https://github.com/rpo19/ReFactX
- name: GPT4o-mini
  role: Component of the GCR baseline on WebQSP (92.2§ Hit / 74.1§ F1) whose metric-setting
    caveat must be carried into the table
- name: GPT4
  role: HQA baseline model on Mintaka (95.9) whose row is extracted with its ‡ evaluation-setting
    footnote
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:70f1a5f08b313ab8b475b1f4ac70bd17b6a3d3b2710a6562dda777a7c7e5b1cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - GPT3.5
  - GPT4
  - GPT4o-mini
  - Llama3.3-70B
  - Qwen2.5-72B
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-method-metric-normalization

## Summary

Extracts reported scores from a paper's results tables and externally published baselines (e.g. ReFactX vs HQA, DoG, GCR) into a tidy CSV with one row per (method, model, dataset, metric, value), preserving metric-setting caveats so heterogeneous metrics are not compared as if commensurable.

## When to use

Use when a paper reports results under one metric (e.g. LLM-as-a-Judge accuracy/precision) while baselines report incompatible metrics (Hit@1, F1) on different dataset samples and a comparison table must be built honestly.

## When NOT to use

- All compared systems were evaluated on the identical dataset sample under the same metric, so no normalization caveats are needed
- The goal is to reproduce experiments rather than tabulate reported numbers — use eval.py with INDEX/MODEL/DATASET instead

## Inputs

- ReFactX results tables (LLM-as-a-Judge precision and accuracy per model: Qwen2.5-3B/7B/72B, Llama3.3-70B, Phi-4, Qwen2.5-70B) across Mintaka, 2WikiMH, WebQSP, Bank
- Published baseline rows: HQA on Mintaka (GPT3.5, GPT4), DoG on 2WikiMH, GCR on WebQSP (Llama3.1 8B + GPT4o-mini)
- Per-question-type accuracy breakdowns for Mintaka (superlative, count, comparative, multi-hop)
- Footnote markers (‡, *, §) indicating differing evaluation settings

## Outputs

- baseline_comparison_table.csv with one row per (method, model, dataset, metric, value)
- Caveat columns per row: metric-setting incompatibility (Hit/F1 vs accuracy) and dataset-size differences (6964 vs 1628 vs Mintaka)
- Validation log confirming each cell matches source-table values

## How to apply

Extract the focal system's reported values per model and dataset from the results tables (e.g. Qwen2.5-72B P=85.2% A=78.9; Llama3.3-70B P=77.8% A=70.6; Bank dataset P up to 51.3%, A up to 42.8%). Separately extract baseline rows — HQA on Mintaka (GPT3.5 85.9, GPT4 95.9), DoG on 2WikiMH (6964 questions), GCR on WebQSP (1628 questions; Llama3.1 8B + GPT4o-mini 92.2§ Hit / 74.1§ F1) — preserving footnote markers (‡, *, §) that flag differing evaluation settings. Normalize everything into a tidy CSV with one row per (method, model, dataset, metric, value) and add explicit caveat columns recording Hit/F1-vs-accuracy incompatibility and dataset-size differences (6964 vs 1628 vs Mintaka). Also capture per-question-type breakdowns where available (e.g. Mintaka superlative 14.3%/47.6%, count 55.0%/35.0%). Validate by spot-checking every cell against the source table (e.g. Qwen2.5-72B/Mintaka/accuracy must equal 78.9) and confirming every baseline row carries a metric caveat.

## Related tools

- **Qwen2.5-72B** (Focal-system model whose reported scores (P=85.2%, A=78.9 on Mintaka) populate the ReFactX rows of the comparison table) — https://github.com/rpo19/ReFactX
- **Llama3.3-70B** (Focal-system model whose reported scores (P=77.8%, A=70.6) populate ReFactX rows and Mintaka per-question-type breakdowns) — https://github.com/rpo19/ReFactX
- **GPT4o-mini** (Component of the GCR baseline on WebQSP (92.2§ Hit / 74.1§ F1) whose metric-setting caveat must be carried into the table)
- **GPT4** (HQA baseline model on Mintaka (95.9) whose row is extracted with its ‡ evaluation-setting footnote)

## Examples

```
python scripts/build_baseline_table.py --source results_tables.md --out baseline_comparison_table.csv --check Qwen2.5-72B,Mintaka,accuracy,78.9
```

## Evaluation signals

- Every cell in baseline_comparison_table.csv matches the source results-table value (e.g. row Qwen2.5-72B/Mintaka/accuracy equals 78.9)
- Every baseline row (HQA, DoG, GCR) carries a metric-setting caveat column noting Hit/F1 vs accuracy incompatibility
- Footnote markers (‡, *, §) from the source tables are preserved or transcribed into caveat fields
- Dataset-size differences (DoG 6964 questions, GCR 1628 questions vs Mintaka) are recorded for every baseline row
- Tidy-schema check: exactly one row per (method, model, dataset, metric, value) tuple with no merged cells

## Limitations

- Baseline and focal-system evaluations are not run on the same dataset sample (e.g. DoG on 6964 2WikiMH questions vs ReFactX's sample), so cross-method comparisons are indicative only
- GCR reports Hit/F1 (92.2§/74.1§) rather than accuracy, making direct numeric comparison with LLM-as-a-Judge accuracy invalid without caveats
- HQA Mintaka scores come from GPT3.5/GPT4 under different (footnote-flagged) settings than ReFactX's open-weight models

## Evidence
