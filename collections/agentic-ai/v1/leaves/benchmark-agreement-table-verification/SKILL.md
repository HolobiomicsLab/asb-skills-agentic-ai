---
name: benchmark-agreement-table-verification
description: Use when auditing the reliability of a double-annotated gold dataset
  for pairwise causal discovery — specifically verifying that Cohen's kappa for binary
  causal/non-causal labels meets a reported agreement threshold.
when_to_use_negative:
- The dataset has more than two annotators or ordinal/multi-class labels — Cohen's
  kappa as specified here covers only pairwise binary causal/non-causal labels
- The goal is scoring model outputs against gold labels (causal detection C_detect
  or extraction C_extract), not validating annotator agreement — use the benchmark
  scoring workflow instead
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Scripted computation of per-slice confusion matrices, P_o, P_e, and Cohen's
    kappa with pandas
- name: github.com/sydneyanuyah/CausalDiscovery
  role: Source of the released double-annotated causal-detection gold labels in experiments/experiment1/
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:b4b9c0161b8c12c8b2976fdf696949f01ff03237aeb395373a5935f425f4bbc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - Python
  - github.com/sydneyanuyah/CausalDiscovery
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# benchmark-agreement-table-verification

## Summary

Reproduces the inter-annotator Cohen's kappa validation of the causal-detection gold labels in the CausalDiscovery benchmark by building 2x2 annotator-vs-annotator confusion matrices for slices 1a-1d and checking per-slice kappa against the reported Table XV values and the overall threshold of kappa >= 0.758.

## When to use

Use when auditing the reliability of a double-annotated gold dataset for pairwise causal discovery — specifically verifying that Cohen's kappa for binary causal/non-causal labels meets a reported agreement threshold.

## When NOT to use

- The dataset has more than two annotators or ordinal/multi-class labels — Cohen's kappa as specified here covers only pairwise binary causal/non-causal labels
- The goal is scoring model outputs against gold labels (causal detection C_detect or extraction C_extract), not validating annotator agreement — use the benchmark scoring workflow instead

## Inputs

- Double-annotated causal-detection gold label CSV for slices 1a-1d from experiments/experiment1/ (binary causal/non-causal labels, 3,008 rows total)

## Outputs

- inter_annotator_agreement.csv with columns slice_id, P_o, P_e, kappa (per-slice rows plus pooled overall kappa)

## How to apply

Load the double-annotated causal-detection gold labels for slices 1a-1d from the released experiments/experiment1/ directory of github.com/sydneyanuyah/CausalDiscovery using Python/pandas (the dataset file has 3,008 rows). For each slice, build the 2x2 annotator-vs-annotator confusion matrix over the binary causal/non-causal labels, then compute observed agreement (P_o), expected agreement (P_e), and Cohen's kappa = (P_o - P_e) / (1 - P_e) per slice, plus the overall kappa pooled across slices. Validate against the reported Table XV values 0.7095, 0.6992, 0.7240, and 0.8983 for slices 1a-1d, and confirm the pooled overall kappa is >= 0.758, matching the paper's abstract claim of high inter-annotator agreement. Note that the final distilled gold set retains only instances with 100% annotator agreement, so kappa is computed before that filtering step.

## Related tools

- **Python** (Scripted computation of per-slice confusion matrices, P_o, P_e, and Cohen's kappa with pandas)
- **github.com/sydneyanuyah/CausalDiscovery** (Source of the released double-annotated causal-detection gold labels in experiments/experiment1/) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
from sklearn.metrics import cohen_kappa_score; df = pd.read_csv('experiments/experiment1/data/exp1.csv'); kappa = cohen_kappa_score(df['annotator1'], df['annotator2'])
```

## Evaluation signals

- Per-slice kappa values match Table XV: 0.7095 (slice 1a), 0.6992 (slice 1b), 0.7240 (slice 1c), 0.8983 (slice 1d)
- Pooled overall kappa across slices 1a-1d is >= 0.758, reproducing the abstract's reliability claim
- inter_annotator_agreement.csv contains one row per slice plus the overall row with columns slice_id, P_o, P_e, kappa
- Each per-slice confusion matrix is 2x2 over the binary causal/non-causal label space and its cells sum to the slice's row count
- P_e is recomputable from the marginal label distributions and satisfies 0 <= P_e <= 1 per slice

## Limitations

- Kappa must be computed on the full double-annotated labels before the distillation step that keeps only 100%-agreement instances; applying it post-filter would inflate agreement
- Agreement verification only covers the causal-detection (Experiment 1) labels; the causal-extraction data (Experiment 2, task slices 2a-2h) is not covered by the reported kappa values

## Evidence

- [abstract] Benchmark dataset validated with high inter-annotator agreement (kappa >= 0.758): [evidence span withheld — anchor status: not_found]
- [readme] Experiment 1 data location and size: "experiments/experiment1/ contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows."
- [methods] Final gold set retains only full-agreement instances: [evidence span withheld — anchor status: not_found]
