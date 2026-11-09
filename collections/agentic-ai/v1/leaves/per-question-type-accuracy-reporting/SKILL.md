---
name: per-question-type-accuracy-reporting
description: Use when comparing reported accuracies across question types or datasets
  (e.g. Mintaka, 2WikiMH, WebQSP) and baseline systems whose metrics are not directly
  comparable (accuracy vs Hit/F1, differing dataset sizes).
when_to_use_negative:
- When only a single method on a single dataset is reported and no cross-system comparison
  or per-type breakdown is needed — a simple accuracy report suffices
- When the source tables do not distinguish metrics (accuracy vs Hit/F1) and no footnote/evaluation-setting
  information is available to populate caveat columns, since merged values would be
  misleading
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: Qwen2.5-72B
  role: Model whose per-dataset and per-question-type accuracies are extracted and
    reported
  repo: https://github.com/rpo19/ReFactX
- name: Llama3.3-70B
  role: Model whose per-dataset and per-question-type accuracies are extracted and
    reported
  repo: https://github.com/rpo19/ReFactX
- name: GPT3.5
  role: HQA baseline model on Mintaka whose reported score is normalized into the
    comparison table
- name: GPT4
  role: HQA baseline model on Mintaka whose reported score is normalized into the
    comparison table
- name: GPT4o-mini
  role: Component of the GCR baseline on WebQSP (with Llama3.1 8B) whose Hit/F1 scores
    carry metric-setting caveats
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:284abba3d80f0864937cc46a1c68e020e8f79d71ee4bc3736ba17c9e73145a62
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
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# per-question-type-accuracy-reporting

## Summary

Extracts and normalizes per-question-type accuracy breakdowns (e.g. Mintaka superlative, count, comparative, multi-hop) from KE-QA results tables for multiple models, producing a tidy comparison table with one row per (method, model, dataset, metric, value) plus metric-compatibility caveats.

## When to use

Use when comparing reported accuracies across question types or datasets (e.g. Mintaka, 2WikiMH, WebQSP) and baseline systems whose metrics are not directly comparable (accuracy vs Hit/F1, differing dataset sizes).

## When NOT to use

- When only a single method on a single dataset is reported and no cross-system comparison or per-type breakdown is needed — a simple accuracy report suffices
- When the source tables do not distinguish metrics (accuracy vs Hit/F1) and no footnote/evaluation-setting information is available to populate caveat columns, since merged values would be misleading

## Inputs

- ReFactX results tables with LLM-as-a-Judge precision (P) and accuracy (A) per model and dataset (Llama3.3-70B, Qwen2.5-72B; Mintaka, 2WikiMH, WebQSP, Bank)
- Baseline rows for HQA (Mintaka), DoG (2WikiMH), and GCR (WebQSP), including footnote markers
- Per-question-type accuracy rows from the Mintaka results (superlative, count, comparative, multi-hop)

## Outputs

- baseline_comparison_table.csv in tidy long format with columns (method, model, dataset, metric, value) plus caveat columns
- Per-question-type accuracy breakdown for Llama3.3-70B and Qwen2.5-72B

## How to apply

First extract the LLM-as-a-Judge precision and accuracy values per model and dataset from the results tables (e.g. Qwen2.5-72B P=85.2% A=78.9 on Mintaka; Llama3.3-70B P=77.8% A=70.6), then pull the baseline rows: HQA on Mintaka (GPT3.5 85.9, GPT4 95.9), DoG on 2WikiMH (6964 questions), and GCR on WebQSP (1628 questions; 92.2 Hit / 74.1 F1), preserving footnote markers that flag differing evaluation settings. Next extract the per-question-type accuracies for each model — e.g. Mintaka superlative 14.3% (Llama3.3-70B) vs 47.6% (Qwen2.5-72B) and count 55.0% vs 35.0%, plus comparative and multi-hop rows. Normalize everything into a tidy CSV with one row per (method, model, dataset, metric, value) and add caveat columns noting the Hit/F1-vs-accuracy incompatibility and dataset-size differences (6964 vs 1628 vs Mintaka). Validation requires every cell to match the source table (e.g. Qwen2.5-72B/Mintaka/accuracy = 78.9) and every baseline row to carry its metric-setting caveat.

## Related tools

- **Qwen2.5-72B** (Model whose per-dataset and per-question-type accuracies are extracted and reported) — https://github.com/rpo19/ReFactX
- **Llama3.3-70B** (Model whose per-dataset and per-question-type accuracies are extracted and reported) — https://github.com/rpo19/ReFactX
- **GPT3.5** (HQA baseline model on Mintaka whose reported score is normalized into the comparison table)
- **GPT4** (HQA baseline model on Mintaka whose reported score is normalized into the comparison table)
- **GPT4o-mini** (Component of the GCR baseline on WebQSP (with Llama3.1 8B) whose Hit/F1 scores carry metric-setting caveats)

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET  # then tabulate per-question-type accuracy rows into baseline_comparison_table.csv
```

## Evaluation signals

- Every cell in baseline_comparison_table.csv matches the source results-table values (e.g. the row Qwen2.5-72B/Mintaka/accuracy equals 78.9)
- Every baseline row carries a metric-setting caveat (Hit/F1 vs accuracy incompatibility flagged, footnote markers preserved)
- Dataset-size differences (DoG 6964 questions, GCR 1628 questions, Mintaka) are recorded as caveats
- Each row conforms to the (method, model, dataset, metric, value) schema with exactly one metric value per row
- Per-question-type rows cover superlative (14.3%/47.6%), count (55.0%/35.0%), comparative, and multi-hop for both Llama3.3-70B and Qwen2.5-72B

## Limitations

- Baseline systems are not evaluated on the same dataset sample (DoG on 6964 2WikiMH questions, GCR on 1628 WebQSP questions), so comparisons are indicative only
- Hit/F1 metrics (GCR: 92.2 Hit / 74.1 F1) are not directly comparable with LLM-as-a-Judge accuracy
- Per-question-type cells can be very low (e.g. superlative 14.3%), so small-sample variance per type should be considered

## Evidence

- [results] Mintaka per-question-type accuracies with metric caveats: [evidence span withheld — anchor status: not_found]
- [results] Tidy CSV normalization with caveats: [evidence span withheld — anchor status: not_found]
- [results] Dataset-size and sampling caveat: [evidence span withheld — anchor status: not_found]
