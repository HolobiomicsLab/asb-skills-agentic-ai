---
name: cause-effect-pair-string-matching
description: Use when evaluating causal-extraction outputs (task slices 2a-2h) from
  results/experiment2/ against consolidated gold cause-effect annotations and you
  need error-bucket diagnostics per model and prompting strategy.
when_to_use_negative:
- Evaluating the causal-detection task (Experiment 1, 3,008-row exp1.csv), which uses
  binary causal-link labels rather than cause-effect pair extraction
- The gold annotations themselves need validation (use inter-annotator agreement,
  kappa >= 0.758, not pair matching)
- Measuring detection skill C_detect scores — this skill targets extraction quality
  (C_extract) only
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3068
tools:
- name: Python
  role: Implements exact-match and similarity-scoring comparison of predicted vs.
    gold pairs and computes bucket percentages via pandas value_counts normalization
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Best extraction model whose outputs (C_extract = 47.12%) are scored and bucketized
    by this matching logic
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:ec5adc01f1f5f23dbe7faf43ec8c0eb18373d326076aab2af93807c5deadd63d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - Python
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3068
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cause-effect-pair-string-matching

## Summary

Scores predicted cause-effect pairs from LLM extraction outputs against gold annotations using exact-match and similarity-scoring logic, then assigns each example to Table 7 error buckets (Correct, Missing, Incorrect, Hallucinated data, Switch CE, False Negative) with bucket-level percentages.

## When to use

Use when evaluating causal-extraction outputs (task slices 2a-2h) from results/experiment2/ against consolidated gold cause-effect annotations and you need error-bucket diagnostics per model and prompting strategy.

## When NOT to use

- Evaluating the causal-detection task (Experiment 1, 3,008-row exp1.csv), which uses binary causal-link labels rather than cause-effect pair extraction
- The gold annotations themselves need validation (use inter-annotator agreement, kappa >= 0.758, not pair matching)
- Measuring detection skill C_detect scores — this skill targets extraction quality (C_extract) only

## Inputs

- Evaluated extraction outputs from results/experiment2/ grouped by model and prompting strategy (e.g., Qwen2.5-Coder-32B-Instruct)
- Consolidated Experiment 2 gold cause-effect dataset (8,925 rows, task slices 2a-2h)
- analysis/ scripts for exact-match, similarity-scoring, and error-bucket assignment

## Outputs

- error_bucket_percentages.csv with per-bucket percentages (Correct, Missing, Incorrect, Hallucinated data, Switch CE, False Neg)
- Per-example bucket assignments for each model/prompting-strategy combination

## How to apply

Load evaluated extraction outputs for the model/prompting strategy from results/experiment2/ and the gold cause-effect pairs from the consolidated Experiment 2 dataset (8,925 rows). Compare each predicted cause-effect pair against the gold pair using exact-match first, then similarity scoring as a fallback, matching the logic in the analysis/ scripts (exact-match, similarity-scoring, problem-case, and error-bucket files). Assign each example to one of the error buckets — Correct, Missing, Incorrect, Hallucinated data, Switch CE (cause/effect swapped), or False Negative — based on the match outcome. Compute per-bucket percentages with pandas value_counts normalized to the total and write error_bucket_percentages.csv. Validate against the reported Table 7 distribution: Correct 45.55%, Missing 35.70%, Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, False Neg 3.22%.

## Related tools

- **Python** (Implements exact-match and similarity-scoring comparison of predicted vs. gold pairs and computes bucket percentages via pandas value_counts normalization) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Best extraction model whose outputs (C_extract = 47.12%) are scored and bucketized by this matching logic) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
python analysis/error_buckets.py --results results/experiment2/ --model Qwen2.5-Coder-32B-Instruct --gold experiments/experiment2/data/Experiment2.csv --out error_bucket_percentages.csv
```

## Evaluation signals

- Reproduced bucket percentages match Table 7: Correct 45.55%, Missing 35.70%, Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, False Neg 3.22%
- Percentages from value_counts(normalize=True) sum to 1.0 across the six buckets
- Every example in the 8,925-row Experiment 2 dataset receives exactly one bucket label (no unassigned rows)
- Switch CE assignments only occur when extracted cause and effect phrases match the gold pair after swapping

## Limitations

- Exact-match scoring is brittle to paraphrase; similarity scoring is needed as fallback for near-matches, and boundary choices between Incorrect and Switch CE buckets depend on the analysis/ scripts' thresholds
- Extraction performance remains low overall (best model reached just 47.12% C_extract), so bucket counts for the best model are dominated by Missing (35.70%) and Incorrect (6.07%)
- Models performed best on simple, explicit, single-sentence relations but performance dropped for implicit relationships, links spanning multiple sentences, and texts with multiple causal pairs, biasing bucket distributions by difficulty

## Evidence

- [methods] Compare each predicted cause-effect pair against the gold pair using exact-match and similarity-scoring logic consistent with the analysis/ scripts.: [evidence span withheld — anchor status: not_found]
- [methods] Assign each example to one of the error buckets — Incorrect, Hallucinated data, Switch CE (cause/effect swapped), False Negative, Missing, or Correct — based on the match outcome.: [evidence span withheld — anchor status: not_found]
- [methods] the reproduced percentages must match the reported Table 7 distribution (Correct 45.55%, Missing 35.70%, Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, False Neg 3.22%).: [evidence span withheld — anchor status: not_found]
- [readme] `analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files.: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [abstract] the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12\% ($C_{extract}$): "the best for extraction, Qwen2.5-Coder-32B-Instruct, reached just 47.12\% ($C_{extract}$)"
