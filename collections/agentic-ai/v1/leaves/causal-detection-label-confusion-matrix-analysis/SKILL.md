---
name: causal-detection-label-confusion-matrix-analysis
description: Use when double-annotated binary causal/non-causal gold labels (slices
  1a-1d) from the CausalDetection experiment need their inter-annotator agreement
  quantified or validated against a reported kappa threshold.
when_to_use_negative:
- The goal is evaluating LLM model outputs against the gold labels (C_detect scoring),
  not annotator-vs-annotator reliability on the gold data itself
- The task involves causal extraction (experiment 2, task slices 2a-2h) with cause-effect
  phrase spans, since binary kappa/confusion-matrix scoring does not apply to phrase
  extraction
- Labels are continuous or multi-class agreement ratings rather than the binary causal/non-causal
  detection labels
tools:
- name: Python
  role: Used with pandas to load the double-annotated slices, build the 2x2 confusion
    matrices, and compute P_o, P_e, and Cohen's kappa
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:c66d0740621b83245a7d02335e9b252b81e037bd7a6a69811020c4520ae03c64
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - github.com/sydneyanuyah/CausalDiscovery
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# causal-detection-label-confusion-matrix-analysis

## Summary

Reproduce the inter-annotator reliability check for the paper's causal-detection gold data by building 2x2 annotator-vs-annotator confusion matrices per task slice (1a-1d), computing P_o, P_e, and Cohen's kappa, and verifying the pooled kappa meets the reported threshold of κ >= 0.758.

## When to use

Use when double-annotated binary causal/non-causal gold labels (slices 1a-1d) from the CausalDetection experiment need their inter-annotator agreement quantified or validated against a reported kappa threshold.

## When NOT to use

- The goal is evaluating LLM model outputs against the gold labels (C_detect scoring), not annotator-vs-annotator reliability on the gold data itself
- The task involves causal extraction (experiment 2, task slices 2a-2h) with cause-effect phrase spans, since binary kappa/confusion-matrix scoring does not apply to phrase extraction
- Labels are continuous or multi-class agreement ratings rather than the binary causal/non-causal detection labels

## Inputs

- Double-annotated causal-detection gold-label CSVs for slices 1a-1d from experiments/experiment1/ in github.com/sydneyanuyah/CausalDiscovery
- Binary causal/non-causal label columns from two independent annotators per slice

## Outputs

- inter_annotator_agreement.csv with columns slice id, P_o, P_e, kappa
- Per-slice 2x2 annotator-vs-annotator confusion matrices
- Pooled overall kappa value across slices 1a-1d

## How to apply

Load the double-annotated causal-detection gold labels for slices 1a-1d from the released experiments/experiment1/ data in the sydneyanuyah/CausalDiscovery repository (the detection dataset file contains 3,008 rows) using Python/pandas. For each slice, cross-tabulate the two annotators' binary causal/non-causal labels into a 2x2 confusion matrix and derive observed agreement (P_o) and expected agreement (P_e) from its cells. Compute per-slice kappa = (P_o - P_e) / (1 - P_e), then pool counts across slices for an overall kappa. Write per-slice and pooled statistics to inter_annotator_agreement.csv with columns for slice id, P_o, P_e, and kappa. The rationale is that the paper claims its gold detection dataset was 'validated with high inter-annotator agreement (κ >= 0.758)', so this skill independently confirms that reliability claim before any label-filtering step (e.g., retaining only instances with 100% agreement for distillation).

## Related tools

- **Python** (Used with pandas to load the double-annotated slices, build the 2x2 confusion matrices, and compute P_o, P_e, and Cohen's kappa)

## Examples

```
from sklearn.metrics import cohen_kappa_score; kappa = cohen_kappa_score(labels_annotator1, labels_annotator2)
```

## Evaluation signals

- Computed kappa for slices 1a-1d must match the reported Table XV values 0.7095, 0.6992, 0.7240, and 0.8983
- Overall kappa pooled across slices must be >= 0.758, matching the paper's reliability claim
- inter_annotator_agreement.csv must contain one row per slice plus a pooled row, each with valid P_o, P_e, and kappa columns
- Each per-slice 2x2 confusion matrix must have cell counts summing to the slice's total number of double-annotated instances
- Each kappa value must satisfy -1 <= kappa <= 1 and be consistent with its P_o and P_e (kappa = (P_o - P_e) / (1 - P_e))

## Limitations

- The paper retains only instances with 100% annotator agreement for the final distilled gold set, so agreement statistics describe the pre-distillation data, not the filtered labels actually used downstream
- Analysis depends on the double-annotated slices being present in the released experiments/experiment1/ data; if only consolidated label files are available, per-slice confusion matrices cannot be reconstructed

## Evidence

- [abstract] Benchmark dataset validated with high inter-annotator agreement (kappa >= 0.758): [evidence span withheld — anchor status: not_found]
- [readme] experiments/experiment1/ contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows.: "`experiments/experiment1/` contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows."
