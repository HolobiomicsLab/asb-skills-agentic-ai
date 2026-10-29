---
name: cohens-kappa-statistics-estimation
description: Use when two annotators (or two label sources) have independently assigned
  binary labels over the same instances and you must quantify chance-corrected agreement
  to validate a gold dataset.
when_to_use_negative:
- You are evaluating model predictions (C_detect / C_extract scores) rather than human
  inter-annotator agreement — exact-match and similarity scoring apply instead
- The labels are multi-class or continuous rather than the binary causal/non-causal
  labels used in the detection task
- You only have the 100%-agreement distilled subset, which by construction cannot
  recover the original kappa between annotators
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Load the double-annotated CSV labels with pandas and compute the 2x2 confusion
    matrix, P_o, P_e, and Cohen's kappa per slice
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:a0a1d651ebcfc38a91bb3843c847ba6f6417bfa81e455b9ffdca63d288a94268
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# Cohen's kappa statistics estimation

## Summary

Compute per-slice and pooled Cohen's kappa (κ = (P_o − P_e)/(1 − P_e)) from double-annotated binary causal/non-causal gold labels to validate the benchmark's inter-annotator agreement claim of κ ≥ 0.758.

## When to use

Use when two annotators (or two label sources) have independently assigned binary labels over the same instances and you must quantify chance-corrected agreement to validate a gold dataset.

## When NOT to use

- You are evaluating model predictions (C_detect / C_extract scores) rather than human inter-annotator agreement — exact-match and similarity scoring apply instead
- The labels are multi-class or continuous rather than the binary causal/non-causal labels used in the detection task
- You only have the 100%-agreement distilled subset, which by construction cannot recover the original kappa between annotators

## Inputs

- Double-annotated causal-detection gold labels for slices 1a-1d (CSV files under experiments/experiment1/ in the CausalDiscovery repository)
- Binary causal/non-causal label column from each annotator

## Outputs

- inter_annotator_agreement.csv with columns: slice id, P_o, P_e, kappa
- Overall pooled Cohen's kappa value (must be >= 0.758)

## How to apply

Load the double-annotated causal-detection gold labels for slices 1a–1d from experiments/experiment1/ in github.com/sydneyanuyah/CausalDiscovery with Python/pandas. For each slice, build the 2×2 annotator-vs-annotator confusion matrix over the binary causal/non-causal labels, then compute observed agreement (P_o), expected agreement (P_e), and kappa = (P_o − P_e)/(1 − P_e) per slice, plus an overall kappa pooled across slices. Write per-slice and overall statistics to inter_annotator_agreement.csv with columns for slice id, P_o, P_e, and kappa. Validate against the reported Table XV values 0.7095, 0.6992, 0.7240, and 0.8983 for slices 1a–1d; the pooled kappa must be ≥ 0.758, matching the paper's reliability claim for the gold detection data before 100%-agreement instances were retained for distillation.

## Related tools

- **Python** (Load the double-annotated CSV labels with pandas and compute the 2x2 confusion matrix, P_o, P_e, and Cohen's kappa per slice) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
import pandas as pd; from sklearn.metrics import cohen_kappa_score; df = pd.read_csv('experiments/experiment1/data/exp1.csv'); k = cohen_kappa_score(df['label_annotator1'], df['label_annotator2'])
```

## Evaluation signals

- Per-slice kappa values match Table XV exactly: 0.7095 (1a), 0.6992 (1b), 0.7240 (1c), 0.8983 (1d)
- Overall pooled kappa across slices is >= 0.758, matching the reported threshold
- inter_annotator_agreement.csv contains all four slice ids with non-null P_o, P_e, and kappa columns
- Each per-slice 2x2 confusion matrix totals the number of double-annotated instances in that slice
- P_e computed from the confusion-matrix marginals is consistent with P_o such that kappa = (P_o - P_e)/(1 - P_e)

## Limitations

- Kappa is only recoverable for slices where both annotators' labels are present; before retaining only 100%-agreement instances for the final distillation, the full double annotations are needed
- Kappa is sensitive to label-prevalence skew in the binary causal/non-causal labels, so per-slice values (0.6992 to 0.8983) vary even when agreement is high
- The discussion/limitations section of the source document contains no changelog, so additional caveats about the annotation protocol could not be extracted

## Evidence

- [abstract] kappa threshold in abstract: [evidence span withheld — anchor status: not_found]
- [readme] experiment1 data location and size: "`experiments/experiment1/` contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows."
- [readme] analysis artifacts: "`analysis/` contains exact-match, similarity-scoring, problem-case, and error-bucket files."
