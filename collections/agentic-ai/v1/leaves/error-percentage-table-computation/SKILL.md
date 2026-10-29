---
name: error-percentage-table-computation
description: Use when you have model-vs-gold comparisons for the causal extraction
  tasks (2a-2h) and need to quantify the share of each error bucket, e.g. to identify
  actionable error sources like Incorrect or Hallucinated data.
when_to_use_negative:
- The goal is causal detection (Experiment 1, 3,008-row dataset) rather than causal
  extraction (Experiment 2) — the error buckets are defined only for extraction outputs.
- You only need aggregate extraction accuracy (e.g. the 47.12% C_extract score) without
  a per-error-type breakdown.
- Predictions have not yet been evaluated against gold pairs with exact-match and
  similarity-scoring logic.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Bucket assignment and percentage computation via pandas value_counts normalized
    to the total; writes error_bucket_percentages.csv
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Extraction model whose evaluated outputs are bucketized; its main actionable
    error sources were Incorrect (6.07%), Hallucinated data (4.47%), and Switch CE
    (4.06%)
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:21c1bd2878d0618101f915213793b6a1ffcb37f6fc8c4e35442d8b8aed7ccdd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - Python
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# error-percentage-table-computation

## Summary

Computation of a per-bucket percentage distribution (Table 7-style error buckets: Correct, Missing, Incorrect, Hallucinated data, Switch CE, False Neg) over evaluated causal-extraction outputs for a given model and prompting strategy, normalized with pandas value_counts and written to error_bucket_percentages.csv.

## When to use

Use when you have model-vs-gold comparisons for the causal extraction tasks (2a-2h) and need to quantify the share of each error bucket, e.g. to identify actionable error sources like Incorrect or Hallucinated data.

## When NOT to use

- The goal is causal detection (Experiment 1, 3,008-row dataset) rather than causal extraction (Experiment 2) — the error buckets are defined only for extraction outputs.
- You only need aggregate extraction accuracy (e.g. the 47.12% C_extract score) without a per-error-type breakdown.
- Predictions have not yet been evaluated against gold pairs with exact-match and similarity-scoring logic.

## Inputs

- Evaluated extraction outputs from results/experiment2/ grouped by model and prompting strategy
- Consolidated Experiment 2 gold cause-effect dataset (8,925 rows, task slices 2a-2h)

## Outputs

- error_bucket_percentages.csv: bucket-to-percentage table (Incorrect, Hallucinated data, Switch CE, False Neg, Missing, Correct)

## How to apply

Load the evaluated extraction outputs for the target model (e.g. Qwen2.5-Coder-32B-Instruct) from results/experiment2/, grouped by model and prompting strategy, along with the consolidated Experiment 2 gold annotations (8,925 rows). Compare each predicted cause-effect pair against the gold pair using exact-match and similarity-scoring logic consistent with the analysis/ scripts. Assign each example to one of the error buckets — Incorrect, Hallucinated data, Switch CE (cause/effect swapped), False Negative, Missing, or Correct — based on the match outcome. Compute the percentage of examples per bucket with Python (pandas value_counts normalized to the total) and write the bucket-percentage table to error_bucket_percentages.csv. Validation requires the reproduced percentages to match the reported Table 7 distribution (Correct 45.55%, Missing 35.70%, Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, False Neg 3.22%).

## Related tools

- **Python** (Bucket assignment and percentage computation via pandas value_counts normalized to the total; writes error_bucket_percentages.csv) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Extraction model whose evaluated outputs are bucketized; its main actionable error sources were Incorrect (6.07%), Hallucinated data (4.47%), and Switch CE (4.06%)) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
df['bucket'].value_counts(normalize=True).mul(100).to_csv('error_bucket_percentages.csv')
```

## Evaluation signals

- Reproduced percentages match the reported Table 7 distribution: Correct 45.55%, Missing 35.70%, Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, False Neg 3.22%
- Bucket percentages sum to 100% across the six buckets (value_counts normalized to total)
- Every example is assigned to exactly one of the six buckets with no unlabeled rows
- Output file error_bucket_percentages.csv exists and covers all buckets including the rare ones (false positives and False Neg ~3.22%)

## Limitations

- Validation percentages are specific to Qwen2.5-Coder-32B-Instruct; other models will have different bucket distributions so Table 7 match is not a universal check.
- Switch CE (cause/effect swapped) detection depends on the exact-match/similarity-scoring logic in analysis/; inconsistent scoring will misassign buckets.
- The document contains no discussion/limitations section text, so additional caveats are not extractable from context.

## Evidence

- [methods] Compute the percentage of examples per bucket with Python (pandas value_counts normalized to the total).: [evidence span withheld — anchor status: not_found]
- [methods] the reproduced percentages must match the reported Table 7 distribution: [evidence span withheld — anchor status: not_found]
- [readme] analysis/ contains exact-match, similarity-scoring, problem-case, and error-bucket files.: "- `analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [readme] results/experiment2/ contains evaluated outputs and timing files grouped by model and prompting strategy.: "- `results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
