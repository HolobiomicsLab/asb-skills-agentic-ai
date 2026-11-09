---
name: llm-output-error-bucket-classification
description: Use when you have evaluated extraction outputs grouped by model and prompting
  strategy plus gold cause-effect pairs (e.g., the 8,925-row consolidated Experiment
  2 dataset) and need a quantitative error-source breakdown.
when_to_use_negative:
- The goal is causal detection (Experiment 1, 3,008-row exp1.csv) rather than cause-effect
  phrase extraction — error bucketing here applies to extraction predictions only
- Predicted pairs have not yet been scored for match outcome — the buckets require
  an upstream exact-match/similarity comparison against gold pairs
- A different model's outputs without a comparable gold cause-effect annotation set,
  since bucket percentages are only meaningful relative to Table 7 validation values
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Used with pandas to normalize value_counts and write the bucket-percentage
    table to error_bucket_percentages.csv
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Extraction model whose evaluated outputs are bucketized; best extraction model
    at 47.12% (C_extract) with Incorrect 6.07%, Hallucinated data 4.47%, Switch CE
    4.06%
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:931aabb791b944597678674674991ac66278b612e6938b0e83dfe1afb7fc1c80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - Python
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# llm-output-error-bucket-classification

## Summary

Classifies each LLM causal-extraction prediction against gold cause-effect annotations into the paper's Table 7 error buckets (Correct, Missing, Incorrect, Hallucinated data, Switch CE, False Neg) and computes bucket percentages to identify the main actionable error sources of a model such as Qwen2.5-Coder-32B-Instruct.

## When to use

Use when you have evaluated extraction outputs grouped by model and prompting strategy plus gold cause-effect pairs (e.g., the 8,925-row consolidated Experiment 2 dataset) and need a quantitative error-source breakdown.

## When NOT to use

- The goal is causal detection (Experiment 1, 3,008-row exp1.csv) rather than cause-effect phrase extraction — error bucketing here applies to extraction predictions only
- Predicted pairs have not yet been scored for match outcome — the buckets require an upstream exact-match/similarity comparison against gold pairs
- A different model's outputs without a comparable gold cause-effect annotation set, since bucket percentages are only meaningful relative to Table 7 validation values

## Inputs

- Evaluated causal-extraction outputs from results/experiment2/ grouped by model and prompting strategy (e.g., Qwen2.5-Coder-32B-Instruct outputs on task slices 2a-2h)
- Gold cause-effect annotations from the consolidated Experiment 2 dataset (8,925 rows, families including CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA)

## Outputs

- Per-example error bucket assignments (Correct, Missing, Incorrect, Hallucinated data, Switch CE, False Negative)
- error_bucket_percentages.csv: percentage of examples per bucket (Table 7-style distribution)

## How to apply

Load the evaluated extraction outputs from results/experiment2/ (grouped by model and prompting strategy) and the gold cause-effect annotations from the consolidated Experiment 2 dataset of 8,925 rows. Compare each predicted cause-effect pair against the gold pair using exact-match and similarity-scoring logic consistent with the analysis/ scripts, then assign each example to one of the error buckets — Incorrect, Hallucinated data, Switch CE (cause/effect swapped), False Negative, Missing, or Correct — based on the match outcome. Compute the percentage of examples per bucket with Python (pandas value_counts normalized to the total) and write the result to error_bucket_percentages.csv. Validate that the reproduced percentages match the reported Table 7 distribution (Correct 45.55%, Missing 35.70%, Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, False Neg 3.22%).

## Related tools

- **Python** (Used with pandas to normalize value_counts and write the bucket-percentage table to error_bucket_percentages.csv) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Extraction model whose evaluated outputs are bucketized; best extraction model at 47.12% (C_extract) with Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%)

## Examples

```
df['bucket'].value_counts(normalize=True).to_csv('error_bucket_percentages.csv')
```

## Evaluation signals

- Reproduced bucket percentages match Table 7 exactly: Correct 45.55%, Missing 35.70%, Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, False Neg 3.22%
- Every one of the 8,925 examples receives exactly one bucket assignment (normalized percentages sum to 1.0)
- error_bucket_percentages.csv contains one row per bucket with a percentage column normalized to the total
- Switch CE assignments correspond to cases where predicted cause and effect are each other's gold counterparts (exact-match on the swapped pair)
- Main actionable buckets for Qwen2.5-Coder-32B-Instruct are Incorrect, Hallucinated data, and Switch CE, with false positives rare

## Limitations

- Missing predictions dominate the distribution (35.70%), so bucketing alone does not separate why extraction failed to produce a parseable pair
- Scoring depends on the exact-match and similarity-scoring logic in analysis/; changing the scoring threshold shifts examples between Correct, Incorrect, and Hallucinated buckets
- Percentages are model- and prompting-strategy-specific; distributions for other of the 13 evaluated LLMs will differ from the Table 7 reference values

## Evidence

- [methods] Assign each example to one of the error buckets — Incorrect, Hallucinated data, Switch CE (cause/effect swapped), False Negative, Missing, or Correct — based on the match outcome.: [evidence span withheld — anchor status: not_found]
- [methods] Validation percentages must match the reported Table 7 distribution (Correct 45.55%, Missing 35.70%, Incorrect 6.07%, Hallucinated data 4.47%, Switch CE 4.06%, False Neg 3.22%).: [evidence span withheld — anchor status: not_found]
- [readme] analysis/ contains exact-match, similarity-scoring, problem-case, and error-bucket files.: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
- [readme] results/experiment2/ contains evaluated outputs and timing files grouped by model and prompting strategy.: "`results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
