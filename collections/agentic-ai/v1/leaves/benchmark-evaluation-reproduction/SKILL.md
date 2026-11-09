---
name: benchmark-evaluation-reproduction
description: Use when a paper reports paired statistical tests (Δ, p-value, Cohen's
  d) over per-query benchmark scores and you must independently recompute and validate
  them from raw per-query outputs.
when_to_use_negative:
- The systems being compared were evaluated on different query sets, so per-query
  scores cannot be joined on query ID for a paired test.
- The reported statistic is an unpaired two-sample comparison or a correlation, not
  a paired mean difference over per-query scores.
- Scores are missing for a large fraction of queries, leaving the paired sample far
  below the 191 annotated queries and changing the test's comparability.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: AgriIR
  role: Framework whose multiple configurations (with/without database integration,
    1B vs 27B models) produce the per-query composite scores under evaluation
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Llama3.2:3B
  role: Model variant assessed with and without database integration for the ablation
    comparison (p=0.204, d=0.14)
- name: Gemma3:1B
  role: Small-model configuration evaluated without database; baseline for the scaling
    comparison against Gemma3:27B
- name: Gemma3:27B
  role: Large-model configuration with database; upper end of the 1B-to-27B scaling
    comparison (Δ=+0.172, p<0.001, d=0.82)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:42aa800c779ec358c0945e7a1b4d424509dd078822bf307c0d9472106862809a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Shuvam-Banerji-Seal/AgriIR
  tools:
  - AgriIR
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# benchmark-evaluation-reproduction

## Summary

Reproduce the reported statistical comparisons of an IR/RAG benchmark paper by recomputing paired statistics (mean difference Δ, p-value, Cohen's d) over per-query composite scores, then verifying the recomputed values against the published numbers. For AgriIR, this covers the 1B→27B scaling gain and the Llama3.2:3B database-ablation effect over 191 annotated agricultural policy queries.

## When to use

Use when a paper reports paired statistical tests (Δ, p-value, Cohen's d) over per-query benchmark scores and you must independently recompute and validate them from raw per-query outputs.

## When NOT to use

- The systems being compared were evaluated on different query sets, so per-query scores cannot be joined on query ID for a paired test.
- The reported statistic is an unpaired two-sample comparison or a correlation, not a paired mean difference over per-query scores.
- Scores are missing for a large fraction of queries, leaving the paired sample far below the 191 annotated queries and changing the test's comparability.

## Inputs

- Per-query composite score files for AgriIR_Gemma3:27B (with database)
- Per-query composite score files for AgriIR_Gemma3:1B (without database)
- Per-query composite score files for AgriIR_Llama3.2:3B (with database)
- Per-query composite score files for AgriIR_Llama3.2:3B (without database)
- AgriIR benchmark of 191 annotated agricultural policy queries with query IDs (30 human annotators)

## Outputs

- statistical_tests.csv with paired mean difference (Δ), p-value, and Cohen's d for the scaling comparison (1B vs 27B) and the database ablation (Llama3.2:3B), with both reported and recomputed columns

## How to apply

Load the per-query composite-score files for the four AgriIR configurations (Llama3.2:3B with and without database integration, Gemma3:1B without database, Gemma3:27B with database) and join them on query ID across the 191 annotated queries. Compute the paired mean difference between AgriIR_Gemma3:1B (w/o DB) and AgriIR_Gemma3:27B and run a paired t-test over the 191 per-query score pairs, then compute Cohen's d for the paired scaling comparison. Repeat the paired t-test and Cohen's d computation for the with/without-database comparison of Llama3.2:3B. Write all statistics with both reported and recomputed columns to statistical_tests.csv, comparing against the reported values (scaling: Δ=+0.172, p<0.001, d=0.82; database ablation: Δ=-0.034, p=0.204, d=0.14).

## Related tools

- **AgriIR** (Framework whose multiple configurations (with/without database integration, 1B vs 27B models) produce the per-query composite scores under evaluation) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Llama3.2:3B** (Model variant assessed with and without database integration for the ablation comparison (p=0.204, d=0.14))
- **Gemma3:1B** (Small-model configuration evaluated without database; baseline for the scaling comparison against Gemma3:27B)
- **Gemma3:27B** (Large-model configuration with database; upper end of the 1B-to-27B scaling comparison (Δ=+0.172, p<0.001, d=0.82))

## Examples

```
from scipy.stats import ttest_rel; import numpy as np; d1 = np.array(agg27b) - np.array(agg1b); t, p = ttest_rel(agg27b, agg1b); cohen_d = d1.mean() / d1.std(ddof=1)
```

## Evaluation signals

- Recomputed scaling delta matches the reported Δ=+0.172 with p<0.001 and Cohen's d=0.82 (large effect) over the same 191 paired query scores.
- Recomputed database-ablation delta matches the reported Δ=-0.034, p=0.204, d=0.14 for Llama3.2:3B.
- Each configuration contributes exactly one score per query ID; the join yields 191 complete pairs with no dropped or duplicated queries.
- statistical_tests.csv contains both reported and recomputed columns for every statistic, and the direction of each effect (large positive scaling gain, near-zero ablation effect) is consistent with the paper.
- The annotated-query count (191) used in the tests matches the AgriIR benchmark size of 191 agricultural policy queries annotated by 30 human evaluators.

## Limitations

- Results depend on per-query composite score files for all four configurations being available; the article does not enumerate exact file paths, and the repo README is truncated so score-file locations must be confirmed in the repository.
- No discussion-section text was extractable from the source document, so limitations, missing artifacts, and reproducibility gaps are not documented in the paper itself.
- Statistical equivalence claims elsewhere in the paper (e.g. AgriIR_Gemma-3-27B 0.820±0.208 vs ChatGPT-4o 0.840±0.233, p=0.493) are outside the scope of the 1B-to-27B and database-ablation recomputation.

## Evidence

- [results] Evaluation on 191 agricultural policy queries with 30 human annotators validates our approach.: "Evaluation on 191 agricultural policy queries with 30 human annotators validates our approach."
