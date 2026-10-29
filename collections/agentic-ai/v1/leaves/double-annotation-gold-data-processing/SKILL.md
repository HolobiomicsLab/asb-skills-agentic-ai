---
name: double-annotation-gold-data-processing
description: Use when you need to validate that a double-annotated binary gold dataset
  (e.g., causal detection labels in slices 1a-1d) meets a reported inter-annotator
  agreement threshold such as κ ≥ 0.758.
when_to_use_negative:
- The annotations are graded/continuous rather than binary causal/non-causal labels
  — Cohen's kappa over a 2x2 confusion matrix does not apply; use a weighted or multiclass
  kappa instead.
- Each instance was labeled by only one annotator — without double annotation there
  is no annotator-vs-annotator confusion matrix to build.
- The goal is model performance evaluation (causal detection C_detect or extraction
  C_extract scoring), not gold-label reliability verification.
edam_operation: http://edamontology.org/operation_2238
tools:
- name: Python
  role: Used with pandas to load the experiment1 CSV and compute per-slice and pooled
    Cohen's kappa from the 2x2 annotator confusion matrices
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:ca36c7930b100e8e5dc7ae435a975e28d182cff0221e4fed42b932ba68d897a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - Python
  - github.com/sydneyanuyah/CausalDiscovery
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# double-annotation-gold-data-processing

## Summary

Recomputes inter-annotator Cohen's kappa on the double-annotated binary causal/non-causal gold labels from the CausalDiscovery benchmark (slices 1a-1d in experiments/experiment1/) to verify the paper's reliability claim of κ ≥ 0.758 before the 100%-agreement distillation step.

## When to use

Use when you need to validate that a double-annotated binary gold dataset (e.g., causal detection labels in slices 1a-1d) meets a reported inter-annotator agreement threshold such as κ ≥ 0.758.

## When NOT to use

- The annotations are graded/continuous rather than binary causal/non-causal labels — Cohen's kappa over a 2x2 confusion matrix does not apply; use a weighted or multiclass kappa instead.
- Each instance was labeled by only one annotator — without double annotation there is no annotator-vs-annotator confusion matrix to build.
- The goal is model performance evaluation (causal detection C_detect or extraction C_extract scoring), not gold-label reliability verification.

## Inputs

- Double-annotated causal-detection gold label CSV for slices 1a-1d from experiments/experiment1/ (github.com/sydneyanuyah/CausalDiscovery; the dataset file contains 3,008 rows)
- Binary causal/non-causal label columns from each of the two annotators, grouped by slice id

## Outputs

- inter_annotator_agreement.csv with columns: slice id, P_o (observed agreement), P_e (expected agreement), kappa
- Overall kappa value pooled across slices 1a-1d for comparison against the κ ≥ 0.758 threshold

## How to apply

Load the double-annotated causal-detection gold labels for slices 1a-1d from experiments/experiment1/ of github.com/sydneyanuyah/CausalDiscovery using Python/pandas. For each slice, build a 2x2 annotator-vs-annotator confusion matrix over the binary causal/non-causal labels, then compute observed agreement (P_o), expected agreement (P_e), and Cohen's kappa = (P_o - P_e)/(1 - P_e), plus an overall kappa pooled across slices. Validate the results against the reported Table XV per-slice values (0.7095, 0.6992, 0.7240, 0.8983 for 1a-1d) and the pooled threshold κ ≥ 0.758. Rationale: the paper retained only instances with 100% annotator agreement for the final distilled gold data, so kappa is the upstream reliability check that justifies that filtering.

## Related tools

- **Python** (Used with pandas to load the experiment1 CSV and compute per-slice and pooled Cohen's kappa from the 2x2 annotator confusion matrices) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
po=(a==b).mean(); pe=(a==1).mean()*(b==1).mean()+(a==0).mean()*(b==0).mean(); kappa=(po-pe)/(1-pe)
```

## Evaluation signals

- Per-slice kappa values must reproduce the reported Table XV values: 0.7095 (1a), 0.6992 (1b), 0.7240 (1c), and 0.8983 (1d)
- Overall pooled kappa across slices 1a-1d must satisfy κ ≥ 0.758 as reported in the paper's abstract
- Each slice's 2x2 annotator-vs-annotator confusion matrix must have cells summing to the slice's instance count (total across slices equals 3,008 rows in the exp1 dataset file)
- inter_annotator_agreement.csv must contain the columns slice id, P_o, P_e, kappa, with every P_o and P_e in [0, 1] and kappa consistent with (P_o - P_e)/(1 - P_e)

## Limitations

- The reported Table XV kappa values are the validation target only if the released experiments/experiment1/ data matches the paper's annotation snapshot; the paper's discussion section is unavailable, so no limitations or missing-artifact notes could be extracted from the article itself.
- Only instances with 100% annotator agreement were retained for the final gold distillation, so the kappa analysis characterizes the full double-annotated set, not necessarily the filtered subset used downstream.
- Kappa is computed on binary causal/non-causal labels only; the skill does not address cause-effect span agreement in the extraction task (slices 2a-2h).

## Evidence

- [abstract] Benchmark dataset validated with high inter-annotator agreement (kappa >= 0.758): [evidence span withheld — anchor status: not_found]
- [readme] Readme describing experiment1 data location and size: "`experiments/experiment1/` contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows."
- [other] Source card workflow describing kappa computation: [evidence span withheld — anchor status: not_found]
- [other] Source card validation criterion against Table XV: [evidence span withheld — anchor status: not_found]
