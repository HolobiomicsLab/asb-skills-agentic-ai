---
name: qa-benchmark-baseline-comparison
description: Decision support for comparing a constrained-generation QA method's reported
  scores against published baseline systems across benchmarks when result tables use
  different metrics (accuracy vs Hit/F1) or differently sized evaluation samples.
when_to_use_negative:
- The goal is a single-dataset ablation within one paper's own results — no cross-system
  normalization or caveat columns are needed.
- Baseline and method scores share an identical metric and evaluation sample (e.g.
  both report accuracy on the same 6964-question 2WikiMH split), so no metric-setting
  caveats are required.
tools:
- name: Qwen2.5-72B
  role: ReFactX model whose reported scores (P=85.2%, A=78.9) form the method rows
    of the comparison table
- name: Llama3.3-70B
  role: ReFactX model whose reported scores (P=77.8%, A=70.6) form the method rows
    of the comparison table
- name: GPT4o-mini
  role: Baseline scorer for GCR on WebQSP (92.2 Hit / 74.1 F1) used as an external
    baseline row
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:bdff423f29830ded6c3ac0715401ee02b02e9a73498c466356ee01525a456e64
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GPT3.5
  - GPT4
  - GPT4o-mini
  - Llama3.3-70B
  - Qwen2.5-72B
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# qa-benchmark-baseline-comparison

## Summary

Builds a normalized baseline-comparison table for KE-QA by extracting ReFactX's LLM-as-a-Judge precision and accuracy values (e.g. Qwen2.5-72B P=85.2%, A=78.9) alongside external baseline rows (HQA on Mintaka, DoG on 2WikiMH, GCR on WebQSP) into one tidy CSV so cross-system scores can be compared with explicit metric-setting caveats.

## When to use

Decision support for comparing a constrained-generation QA method's reported scores against published baseline systems across benchmarks when result tables use different metrics (accuracy vs Hit/F1) or differently sized evaluation samples.

## When NOT to use

- The goal is a single-dataset ablation within one paper's own results — no cross-system normalization or caveat columns are needed.
- Baseline and method scores share an identical metric and evaluation sample (e.g. both report accuracy on the same 6964-question 2WikiMH split), so no metric-setting caveats are required.

## Inputs

- ReFactX results tables reporting LLM-as-a-Judge precision and accuracy per model and dataset
- Baseline rows for HQA (Mintaka), DoG (2WikiMH), and GCR (WebQSP) including footnote markers on evaluation settings
- Mintaka per-question-type accuracy breakdown for Llama3.3-70B and Qwen2.5-72B

## Outputs

- baseline_comparison_table.csv in tidy format with one row per (method, model, dataset, metric, value) plus caveat columns

## How to apply

Extract the reported ReFactX precision and accuracy values per model and dataset from the results tables (Qwen2.5-72B P=85.2% A=78.9; Llama3.3-70B P=77.8% A=70.6; Qwen2.5-7B 73.0% and Qwen2.5-70B 92.5% accuracy on 2WikiMH; Bank dataset P up to 51.3%, A up to 42.8%). Extract baseline rows preserving their footnote markers: HQA on Mintaka (GPT3.5 85.9, GPT4 95.9), DoG on 2WikiMH (6964 questions), GCR on WebQSP (1628 questions; Llama3.1 8B + GPT4o-mini 92.2 Hit / 74.1 F1). Also extract Mintaka per-question-type accuracies (superlative 14.3%/47.6%, count 55.0%/35.0%, comparative, multi-hop) for Llama3.3-70B and Qwen2.5-72B. Normalize everything into a tidy CSV with one row per (method, model, dataset, metric, value) and attach caveat columns flagging Hit/F1-vs-accuracy incompatibility and dataset-size differences (6964 vs 1628 vs Mintaka). Judge correctness by verifying every cell matches the source results-table values and every baseline row carries its metric-setting caveat.

## Related tools

- **Qwen2.5-72B** (ReFactX model whose reported scores (P=85.2%, A=78.9) form the method rows of the comparison table)
- **Llama3.3-70B** (ReFactX model whose reported scores (P=77.8%, A=70.6) form the method rows of the comparison table)
- **GPT4o-mini** (Baseline scorer for GCR on WebQSP (92.2 Hit / 74.1 F1) used as an external baseline row)

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Every cell in baseline_comparison_table.csv matches the source results-table values (e.g. the row Qwen2.5-72B/Mintaka/accuracy equals 78.9)
- Every baseline row carries a metric-setting caveat column (Hit/F1 vs accuracy incompatibility)
- Dataset-size differences (DoG 6964 questions vs GCR 1628 questions vs Mintaka) are recorded per row
- One row exists per (method, model, dataset, metric, value) with no duplicated keys
- Footnote markers from the source tables (e.g. ‡ for HQA, ∗/§ for GCR) are preserved or explicitly represented as caveat flags

## Limitations

- Baselines are not evaluated on the same dataset sample, so comparisons like DoG (84.2%) vs ReFactX (73.0% Qwen2.5-7B, 92.5% Qwen2.5-70B) on 2WikiMH are indicative rather than head-to-head
- GCR reports Hit and F1 rather than accuracy, so those values cannot be directly compared with ReFactX's LLM-as-a-Judge accuracy
- The Bank dataset proved especially challenging (precision up to 51.3%, accuracy up to 42.8%), so absolute scores on it are low and easily misread

## Evidence
