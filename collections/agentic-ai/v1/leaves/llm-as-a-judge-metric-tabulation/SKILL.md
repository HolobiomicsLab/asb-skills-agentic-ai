---
name: llm-as-a-judge-metric-tabulation
description: Use when comparing reported QA scores (precision, accuracy, Hit/F1) across
  a new system and published baselines whose metrics and dataset samples are not directly
  compatible, so values must be normalized with explicit caveats.
when_to_use_negative:
- When the goal is reproducing ReFactX's own experiments rather than tabulating reported
  scores — use eval.py/throughput.py from the repository instead
- When baselines were evaluated on identical questions with identical metrics, making
  direct averaging of rows misleadingly simple or unnecessary normalization
- When the paper under study reports no LLM-as-a-Judge or baseline scores to extract
  (e.g. only discussion content)
tools:
- name: Qwen2.5-72B
  role: Source of LLM-as-a-Judge evaluation results (P=85.2%, A=78.9) tabulated as
    CSV rows
  repo: https://github.com/rpo19/ReFactX
- name: Llama3.3-70B
  role: Source of LLM-as-a-Judge evaluation results (P=77.8%, C=70.6%) tabulated as
    CSV rows
  repo: https://github.com/rpo19/ReFactX
- name: GPT3.5
  role: HQA baseline model on Mintaka (85.9) recorded as a baseline row with footnote
    marker
- name: GPT4
  role: HQA baseline model on Mintaka (95.9) recorded as a baseline row with footnote
    marker
- name: GPT4o-mini
  role: Component of the GCR baseline (92.2 Hit / 74.1 F1 on WebQSP) recorded as a
    baseline row
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:a5b527df5d92083f2c2945aefc598a32a963c4ad1377effd2b266611eb361213
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# LLM-as-a-Judge metric tabulation

## Summary

Extracts ReFactX's LLM-as-a-Judge precision (P) and accuracy (A) values per model and dataset, plus external baseline rows (HQA, DoG, GCR), into a tidy one-row-per-(method, model, dataset, metric, value) CSV preserving footnote markers about differing evaluation settings.

## When to use

Use when comparing reported QA scores (precision, accuracy, Hit/F1) across a new system and published baselines whose metrics and dataset samples are not directly compatible, so values must be normalized with explicit caveats.

## When NOT to use

- When the goal is reproducing ReFactX's own experiments rather than tabulating reported scores — use eval.py/throughput.py from the repository instead
- When baselines were evaluated on identical questions with identical metrics, making direct averaging of rows misleadingly simple or unnecessary normalization
- When the paper under study reports no LLM-as-a-Judge or baseline scores to extract (e.g. only discussion content)

## Inputs

- ReFactX results tables with LLM-as-a-Judge precision and accuracy per model (Llama3.3-70B, Qwen2.5-72B, Qwen2.5-7B, Qwen2.5-70B, Phi-4) and dataset (Mintaka, 2WikiMH, WebQSP, Bank)
- Baseline rows for HQA (Mintaka), DoG (2WikiMH, 6964 questions), and GCR (WebQSP, 1628 questions) including footnote markers
- Mintaka per-question-type accuracy breakdown (superlative, count, comparative, multi-hop)

## Outputs

- baseline_comparison_table.csv with columns (method, model, dataset, metric, value, caveats) and one row per metric observation
- Caveat annotations distinguishing Hit/F1 metrics from accuracy and noting differing dataset sizes/evaluation samples

## How to apply

Read the paper's results tables and extract every LLM-as-a-Judge cell: Qwen2.5-72B gives P=85.2% and A=78.9 on Mintaka, Llama3.3-70B gives P=77.8% and A=70.6%, and 2WikiMH ReFactX rows give 73.0% (Qwen2.5-7B) and 92.5% (Qwen2.5-70B) accuracy. Extract baseline rows separately and carry their footnote markers: HQA on Mintaka with GPT3.5 at 85.9 and GPT4 at 95.9, DoG on 2WikiMH with 84.2% over 6964 questions, and GCR on WebQSP (1628 questions) with 92.2 Hit and 74.1 F1 using Llama3.1 8B + GPT4o-mini. Also pull Mintaka per-question-type accuracies (superlative 14.3%/47.6%, count 55.0%/35.0%) for Llama3.3-70B and Qwen2.5-72B. Normalize everything into one row per (method, model, dataset, metric, value) and add caveat columns flagging Hit/F1-vs-accuracy incompatibility and dataset-size differences (6964 vs 1628 vs Mintaka). Validate by checking each CSV cell matches its source value, e.g. Qwen2.5-72B/Mintaka/accuracy equals 78.9.

## Related tools

- **Qwen2.5-72B** (Source of LLM-as-a-Judge evaluation results (P=85.2%, A=78.9) tabulated as CSV rows) — https://github.com/rpo19/ReFactX
- **Llama3.3-70B** (Source of LLM-as-a-Judge evaluation results (P=77.8%, C=70.6%) tabulated as CSV rows) — https://github.com/rpo19/ReFactX
- **GPT3.5** (HQA baseline model on Mintaka (85.9) recorded as a baseline row with footnote marker)
- **GPT4** (HQA baseline model on Mintaka (95.9) recorded as a baseline row with footnote marker)
- **GPT4o-mini** (Component of the GCR baseline (92.2 Hit / 74.1 F1 on WebQSP) recorded as a baseline row)

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Every numeric cell in baseline_comparison_table.csv equals the corresponding source results-table value (e.g. Qwen2.5-72B/Mintaka/accuracy = 78.9)
- Every baseline row (HQA, DoG, GCR) carries a metric-setting caveat distinguishing Hit/F1 from accuracy
- Dataset-size caveats distinguish 6964 (2WikiMH), 1628 (WebQSP), and Mintaka question counts
- Schema check: exactly one row per (method, model, dataset, metric, value) tuple with no duplicated or dropped metrics
- Mintaka per-question-type rows (superlative 14.3%/47.6%, count 55.0%/35.0%) are present for both Llama3.3-70B and Qwen2.5-72B

## Limitations

- ReFactX and baselines were not evaluated on the same dataset sample, so cross-method comparisons are approximate even after tabulation
- GCR reports Hit/F1 (92.2/74.1) rather than accuracy, making direct comparison with ReFactX accuracy values invalid without caveats
- The Bank dataset proved especially challenging, with precision only up to 51.3% and accuracy up to 42.8%, so Bank rows should not be treated as comparable to Mintaka/2WikiMH scores
- No discussion-section content was available to contextualize the comparison

## Evidence
