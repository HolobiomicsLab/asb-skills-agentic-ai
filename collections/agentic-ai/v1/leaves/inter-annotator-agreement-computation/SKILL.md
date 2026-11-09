---
name: inter-annotator-agreement-computation
description: Use when you must validate the reliability of double-annotated binary
  causal/non-causal labels (e.g., replicating the paper's Table XV kappa values or
  auditing the gold detection data before model comparison).
when_to_use_negative:
- The data are causal-extraction outputs (experiment 2, task slices 2a–2h) — this
  skill only covers the binary detection labels of experiment 1
- The goal is to score LLM agreement with the gold labels rather than annotator-vs-annotator
  reliability — that requires the paper's exact-match/similarity scoring workflow
  instead
- Only a single annotation pass exists (no second annotator), so a 2x2 annotator-vs-annotator
  confusion matrix cannot be built
edam_topics: []
tools:
- name: Python
  role: Used with pandas to load the double-annotated detection CSV, build 2x2 confusion
    matrices per slice, and compute P_o, P_e, and Cohen's kappa
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: github.com/sydneyanuyah/CausalDiscovery
  role: Released repository supplying the experiments/experiment1/ causal-detection
    data (3,008 rows) that the agreement analysis runs on
  repo: github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:7c1fac584cb8ef45328f4758f800d6de67b86cafc55a648ddc102267a76f07b9
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# inter-annotator-agreement-computation

## Summary

Computes Cohen's kappa inter-annotator agreement over the double-annotated binary causal-detection gold labels (slices 1a–1d) of the CausalDiscovery benchmark, verifying the paper's claim that the gold dataset was validated with kappa >= 0.758.

## When to use

Use when you must validate the reliability of double-annotated binary causal/non-causal labels (e.g., replicating the paper's Table XV kappa values or auditing the gold detection data before model comparison).

## When NOT to use

- The data are causal-extraction outputs (experiment 2, task slices 2a–2h) — this skill only covers the binary detection labels of experiment 1
- The goal is to score LLM agreement with the gold labels rather than annotator-vs-annotator reliability — that requires the paper's exact-match/similarity scoring workflow instead
- Only a single annotation pass exists (no second annotator), so a 2x2 annotator-vs-annotator confusion matrix cannot be built

## Inputs

- Double-annotated causal-detection gold labels for slices 1a–1d (binary causal/non-causal per instance) from experiments/experiment1/ in github.com/sydneyanuyah/CausalDiscovery
- Slice identifier column grouping instances into 1a, 1b, 1c, 1d

## Outputs

- inter_annotator_agreement.csv with columns: slice id, P_o (observed agreement), P_e (expected agreement), kappa
- Overall pooled Cohen's kappa across slices 1a–1d (expected >= 0.758)

## How to apply

Load the double-annotated causal-detection gold labels for slices 1a–1d from the released experiments/experiment1/ data in github.com/sydneyanuyah/CausalDiscovery using Python/pandas. For each slice, build a 2x2 annotator-vs-annotator confusion matrix over the binary causal/non-causal labels, then compute observed agreement (P_o), expected agreement (P_e), and Cohen's kappa = (P_o - P_e) / (1 - P_e) per slice, plus an overall kappa pooled across slices. Write per-slice and overall statistics to inter_annotator_agreement.csv with columns slice id, P_o, P_e, kappa. Validation: per-slice kappa must match the reported Table XV values 0.7095 (1a), 0.6992 (1b), 0.7240 (1c), 0.8983 (1d), and the pooled overall kappa must be >= 0.758, matching the abstract's reliability claim.

## Related tools

- **Python** (Used with pandas to load the double-annotated detection CSV, build 2x2 confusion matrices per slice, and compute P_o, P_e, and Cohen's kappa) — github.com/sydneyanuyah/CausalDiscovery
- **github.com/sydneyanuyah/CausalDiscovery** (Released repository supplying the experiments/experiment1/ causal-detection data (3,008 rows) that the agreement analysis runs on) — github.com/sydneyanuyah/CausalDiscovery

## Examples

```
from sklearn.metrics import cohen_kappa_score; kappa = cohen_kappa_score(annotator1_labels, annotator2_labels)
```

## Evaluation signals

- Per-slice kappa matches Table XV exactly: 0.7095 (slice 1a), 0.6992 (1b), 0.7240 (1c), 0.8983 (1d)
- Overall pooled kappa across slices 1a–1d is >= 0.758
- Each slice yields a valid 2x2 annotator-vs-annotator confusion matrix over binary causal/non-causal labels, and P_e is computed from row/column marginals of that matrix
- inter_annotator_agreement.csv has the required columns (slice id, P_o, P_e, kappa) and exactly the four slice rows plus the pooled overall row
- Identity check: kappa = (P_o - P_e) / (1 - P_e) reproduces from the stored P_o and P_e values in the CSV

## Limitations

- Per-slice kappa for slices 1a (0.7095) and 1b (0.6992) falls below the abstract's headline threshold; only the pooled overall kappa (>= 0.758) meets it
- The workflow retains only instances with 100% annotator agreement for the final distilled gold set, so the released gold data is a filtered subset of the double-annotated pool
- Agreement is computed only for the causal-detection task (experiment 1); no equivalent annotator-agreement statistics are reported for the extraction task

## Evidence

- [abstract] Benchmark dataset validated with high inter-annotator agreement (kappa >= 0.758): [evidence span withheld — anchor status: not_found]
- [readme] Repository location of the double-annotated causal-detection data used for the agreement analysis: "`experiments/experiment1/` contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows."
- [readme] Public availability of the data and code needed to reproduce the agreement analysis: "make all our data, code, and prompts publicly available to spur further research."
