---
name: per-query-composite-score-analysis
description: Use when you have paired per-query quality scores (e.g. composite scores)
  from two RAG/IR system configurations over the same query set and must test whether
  a configuration change (model scale or retrieval component) yields a statistically
  significant gain.
when_to_use_negative:
- Comparing a single aggregate mean score between systems without per-query pairing
  — a paired t-test over matched query scores is not applicable
- Analyzing configurations not evaluated on the same query set (scores cannot be joined
  on query ID), e.g. comparing AgriIR variants against ChatGPT-4o / Gemini 2.5 Flash
  baselines reported only as aggregate means
- Questions about pipeline stage behavior (query refinement, retrieval, synthesis)
  rather than end-to-end score differences between configurations
edam_operation: http://edamontology.org/operation_2239
tools:
- name: AgriIR
  role: Framework whose configurable RAG pipeline produced the per-query composite
    scores being compared across configurations
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Llama3.2:3B
  role: Model assessed with and without database integration to isolate the database-integration
    effect (Δ=-0.034, p=0.204, d=0.14)
- name: Gemma3:1B
  role: Baseline configuration (without database) in the 1B-to-27B paired scaling
    comparison
- name: Gemma3:27B
  role: Scaled configuration (with database) paired against Gemma3:1B to test the
    scaling effect (Δ=+0.172, p<0.001, d=0.82)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:1c1a841d51f3c1b1746c358fcff7c38468ba94c35ead40a4859f54aa1413f54f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Shuvam-Banerji-Seal/AgriIR
  tools:
  - AgriIR
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Per-query composite score analysis

## Summary

Joins per-query composite evaluation scores across AgriIR configurations (Llama3.2:3B with/without database, Gemma3:1B without database, Gemma3:27B with database) on query ID over 191 annotated agricultural policy queries, then runs paired t-tests and Cohen's d to separate the effect of model scaling (1B→27B) from the effect of database integration.

## When to use

Use when you have paired per-query quality scores (e.g. composite scores) from two RAG/IR system configurations over the same query set and must test whether a configuration change (model scale or retrieval component) yields a statistically significant gain.

## When NOT to use

- Comparing a single aggregate mean score between systems without per-query pairing — a paired t-test over matched query scores is not applicable
- Analyzing configurations not evaluated on the same query set (scores cannot be joined on query ID), e.g. comparing AgriIR variants against ChatGPT-4o / Gemini 2.5 Flash baselines reported only as aggregate means
- Questions about pipeline stage behavior (query refinement, retrieval, synthesis) rather than end-to-end score differences between configurations

## Inputs

- Per-query composite score files for AgriIR_Gemma3:1B (without database)
- Per-query composite score files for AgriIR_Gemma3:27B (with database)
- Per-query composite score files for Llama3.2:3B with database integration
- Per-query composite score files for Llama3.2:3B without database integration
- AgriIR benchmark of 191 annotated agricultural policy queries (query IDs for joining)

## Outputs

- statistical_tests.csv containing paired mean difference (Δ), t-test p-value, and Cohen's d for the 1B-to-27B scaling comparison and the Llama3.2:3B database-impact ablation, with both reported and recomputed columns

## How to apply

Load the per-query composite score files for the four AgriIR configurations and join them on query ID across the 191 annotated queries. For the scaling comparison, compute the paired mean difference between AgriIR_Gemma3:1B (without database) and AgriIR_Gemma3:27B and run a paired t-test over the 191 per-query score pairs, then compute Cohen's d for this paired comparison. Repeat the paired t-test and Cohen's d computation for the with/without-database ablation of Llama3.2:3B. Compare recomputed deltas, p-values, and effect sizes against the reported values — scaling: Δ=+0.172, p<0.001, d=0.82 (large effect); database ablation: Δ=-0.034, p=0.204, d=0.14 (negligible) — and write all statistics, including both reported and recomputed columns, to statistical_tests.csv.

## Related tools

- **AgriIR** (Framework whose configurable RAG pipeline produced the per-query composite scores being compared across configurations) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Llama3.2:3B** (Model assessed with and without database integration to isolate the database-integration effect (Δ=-0.034, p=0.204, d=0.14))
- **Gemma3:1B** (Baseline configuration (without database) in the 1B-to-27B paired scaling comparison)
- **Gemma3:27B** (Scaled configuration (with database) paired against Gemma3:1B to test the scaling effect (Δ=+0.172, p<0.001, d=0.82))

## Examples

```
from scipy import stats; t, p = stats.ttest_rel(df_27b.sort_values('qid')['composite'], df_1b.sort_values('qid')['composite'])
```

## Evaluation signals

- The joined score table contains exactly 191 per-query pairs for each comparison (matching the annotated query count)
- Recomputed scaling statistics match the reported Δ=+0.172, p<0.001, d=0.82 (large effect)
- Recomputed database-ablation statistics match the reported Δ=-0.034, p=0.204, d=0.14 (negligible impact)
- statistical_tests.csv contains both reported and recomputed columns for every statistic
- p-value directions are consistent with conclusions: scaling significant at p<0.001, database effect non-significant at p=0.204

## Limitations

- The database-impact conclusion (p=0.204, d=0.14) holds specifically for Llama3.2:3B and is not tested for other models
- Composite scores depend on human annotation quality (30 annotators over 191 queries), so score noise may propagate into the effect-size estimates
- AgriIR_Gemma-3-27B's composite score (0.820±0.208) is statistically equivalent only to ChatGPT-4o (0.840±0.233, p=0.493), not to all baselines — Gemini Flash and GPT-OSS-120B were significantly outperformed

## Evidence
