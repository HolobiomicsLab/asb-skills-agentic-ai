---
name: multi-hop-qa-results-extraction
description: Use when you need to compare reported multi-hop QA benchmark numbers
  (accuracy, precision, Hit, F1) for ReFactX against baseline systems across Mintaka,
  2WikiMH, and WebQSP, including per-question-type breakdowns.
when_to_use_negative:
- When the goal is to re-run or reproduce the QA experiments themselves rather than
  extract already-reported numbers (use eval.py directly instead)
- When baselines and ReFactX scores need to be merged into a single ranking without
  flagging metric incompatibility — Hit/F1 (GCR) vs accuracy (ReFactX/HQA) must not
  be compared as if equivalent
tools:
- name: eval.py
  role: Repository script to reproduce the reported experiment numbers if extraction
    needs verification against re-runs
  repo: https://github.com/rpo19/ReFactX
- name: Llama3.3-70B
  role: Model whose reported P/A values are extracted from the results tables
- name: Qwen2.5-72B
  role: Model whose reported P/A values are extracted from the results tables
- name: GPT4o-mini
  role: Component of the GCR baseline row on WebQSP (92.2§ Hit / 74.1§ F1)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:1d7b00c0004838a05b156d681baa9ff2917a4fe5acf9285c637c9458b822c7a9
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
  - eval.py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multi-hop-qa-results-extraction

## Summary

Extracts ReFactX's reported LLM-as-a-Judge precision/accuracy values per model and dataset from the paper's results tables and normalizes them alongside external KE-QA baselines (HQA on Mintaka, DoG on 2WikiMH, GCR on WebQSP) into a tidy comparison table.

## When to use

Use when you need to compare reported multi-hop QA benchmark numbers (accuracy, precision, Hit, F1) for ReFactX against baseline systems across Mintaka, 2WikiMH, and WebQSP, including per-question-type breakdowns.

## When NOT to use

- When the goal is to re-run or reproduce the QA experiments themselves rather than extract already-reported numbers (use eval.py directly instead)
- When baselines and ReFactX scores need to be merged into a single ranking without flagging metric incompatibility — Hit/F1 (GCR) vs accuracy (ReFactX/HQA) must not be compared as if equivalent

## Inputs

- ReFactX paper results tables (LLM-as-a-Judge precision and accuracy per model: Qwen2.5-72B, Llama3.3-70B, Qwen2.5-7B, Qwen2.5-70B, Phi-4)
- Baseline rows for HQA (Mintaka), DoG (2WikiMH, 6964 questions), and GCR (WebQSP, 1628 questions) including footnote markers
- Mintaka per-question-type accuracy breakdown (superlative, count, comparative, multi-hop)

## Outputs

- baseline_comparison_table.csv with one row per (method, model, dataset, metric, value) plus caveat columns
- Per-question-type accuracy breakdown for Mintaka (Llama3.3-70B and Qwen2.5-72B)

## How to apply

Read the paper's results tables and pull ReFactX's LLM-as-a-Judge values per model and dataset (e.g. Qwen2.5-72B P=85.2% A=78.9; Llama3.3-70B P=77.8% A=70.6; Bank dataset P up to 51.3% A up to 42.8). Extract the baseline rows verbatim: HQA on Mintaka (GPT3.5 85.9, GPT4 95.9), DoG on 2WikiMH (6964 questions), and GCR on WebQSP (1628 questions; Llama3.1 8B + GPT4o-mini 92.2§ Hit / 74.1§ F1), preserving the ‡, ∗, and § footnote markers that flag differing evaluation settings. Also pull the Mintaka per-question-type accuracies (superlative 14.3%/47.6%, count 55.0%/35.0%, plus comparative and multi-hop) for Llama3.3-70B and Qwen2.5-72B. Normalize everything into a tidy CSV with one row per (method, model, dataset, metric, value) and add caveat columns noting that Hit/F1 is not comparable to accuracy and that dataset sizes differ (6964 vs 1628 vs Mintaka). Reproducibility of the underlying numbers can be checked with the repo's eval.py script.

## Related tools

- **eval.py** (Repository script to reproduce the reported experiment numbers if extraction needs verification against re-runs) — https://github.com/rpo19/ReFactX
- **Llama3.3-70B** (Model whose reported P/A values are extracted from the results tables)
- **Qwen2.5-72B** (Model whose reported P/A values are extracted from the results tables)
- **GPT4o-mini** (Component of the GCR baseline row on WebQSP (92.2§ Hit / 74.1§ F1))

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Every cell in baseline_comparison_table.csv matches the source results-table values, e.g. the Qwen2.5-72B/Mintaka/accuracy row equals 78.9
- Every baseline row carries a metric-setting caveat column (Hit/F1 vs accuracy incompatibility)
- Footnote markers (‡, ∗, §) from the source tables are preserved on HQA and GCR rows
- Tidy schema holds: exactly one row per (method, model, dataset, metric, value) combination
- Dataset-size caveats (DoG 6964 questions, GCR 1628 questions) are recorded for cross-dataset comparisons

## Limitations

- Baseline approaches are not evaluated on the same dataset samples as ReFactX (DoG on 6964 2WikiMH questions, GCR on 1628 WebQSP questions), so comparisons are indicative rather than head-to-head
- GCR reports Hit and F1 while ReFactX reports LLM-as-a-Judge precision and accuracy — the metrics are not directly interchangeable
- On Mintaka and WebQSP ReFactX performs worse than LLM-only models (LLM-only reaches 82.0% and 80.5% accuracy respectively), and per-question-type scores can be very low (superlative 14.3% with Llama3.3-70B)

## Evidence
