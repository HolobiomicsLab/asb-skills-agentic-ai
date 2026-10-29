---
name: effect-size-computation-cohens-d
description: Use when per-query paired composite scores from two system configurations
  are available and reported statistics (Δ, p, d) must be verified or effect sizes
  reported alongside significance tests.
when_to_use_negative:
- Inputs are unpaired or independently sampled scores — the reported deltas assume
  within-query pairing across configurations
- The goal is comparing AgriIR against external baselines (ChatGPT-4o, Gemini 2.5
  Flash, GPT-OSS-120B) rather than internal scaling/ablation contrasts
- Fewer than the 191 annotated queries are available after joining on query ID, so
  recomputed statistics would not be comparable to the reported ones
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: AgriIR
  role: Source of the per-query composite scores for the four configurations under
    statistical comparison
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Llama3.2:3B
  role: Configuration whose with/without-database paired scores yield the ablation
    contrast (Δ=-0.034, p=0.204, d=0.14)
- name: Gemma3:1B
  role: Baseline arm of the 1B-to-27B scaling contrast (without database)
- name: Gemma3:27B
  role: Scaled arm of the 1B-to-27B scaling contrast (with database)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:1c7cf90b13b2c933748c15f951bca6372bb30174356cc8a3686eb2da4f6d7f3d
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
yes_rate: 0.875
n_claims: 8
---

# effect-size-computation-cohens-d

## Summary

Recomputes paired mean differences, p-values, and Cohen's d for AgriIR system comparisons over 191 annotated agricultural policy queries, verifying reported statistics for model scaling (1B vs 27B) and database-integration ablation (Llama3.2:3B).

## When to use

Use when per-query paired composite scores from two system configurations are available and reported statistics (Δ, p, d) must be verified or effect sizes reported alongside significance tests.

## When NOT to use

- Inputs are unpaired or independently sampled scores — the reported deltas assume within-query pairing across configurations
- The goal is comparing AgriIR against external baselines (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B) rather than internal scaling/ablation contrasts
- Fewer than the 191 annotated queries are available after joining on query ID, so recomputed statistics would not be comparable to the reported ones

## Inputs

- Per-query composite score files for AgriIR_Gemma3:1B (w/o DB), AgriIR_Gemma3:27B (w/ DB), and Llama3.2:3B (with/without database integration), joined on query ID over 191 annotated agricultural policy queries
- Reported statistics to verify: scaling Δ=+0.172, p<0.001, d=0.82; database ablation Δ=-0.034, p=0.204, d=0.14

## Outputs

- statistical_tests.csv with paired mean differences, t-test p-values, and Cohen's d in both reported and recomputed columns

## How to apply

Load the per-query composite score files for the AgriIR configurations (Llama3.2:3B with/without database, Gemma3:1B without database, Gemma3:27B with database) and join them on query ID across the 191 annotated queries. For each comparison, compute the paired mean difference and run a paired t-test over the 191 per-query score pairs (e.g. scipy.stats.ttest_rel), then compute Cohen's d for the paired differences. Verify recomputed values against the reported statistics — scaling 1B→27B: Δ=+0.172, p<0.001, d=0.82 (large effect); database ablation for Llama3.2:3B: Δ=-0.034, p=0.204, d=0.14 (negligible). Interpret d≈0.82 as a large effect and d≈0.14 as negligible, so significance (p) alone must not be conflated with effect magnitude. Write both reported and recomputed columns to statistical_tests.csv.

## Related tools

- **AgriIR** (Source of the per-query composite scores for the four configurations under statistical comparison) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Llama3.2:3B** (Configuration whose with/without-database paired scores yield the ablation contrast (Δ=-0.034, p=0.204, d=0.14))
- **Gemma3:1B** (Baseline arm of the 1B-to-27B scaling contrast (without database))
- **Gemma3:27B** (Scaled arm of the 1B-to-27B scaling contrast (with database))

## Examples

```
python -c "import pandas as pd; from scipy import stats; df=pd.read_csv('composite_scores.csv'); t,p=stats.ttest_rel(df['Gemma3_27B_w_db'],df['Gemma3_1B_wo_db']); print(t,p)"
```

## Evaluation signals

- Recomputed scaling statistics match reported values: Δ=+0.172, p<0.001, d=0.82 (large effect)
- Recomputed database-ablation statistics match reported values: Δ=-0.034, p=0.204, d=0.14 (negligible)
- All 191 per-query score pairs join successfully on query ID with no dropped rows
- statistical_tests.csv contains both reported and recomputed columns for every comparison
- Effect-size interpretation (large vs negligible) is consistent with Cohen's d conventional thresholds (≈0.8 large, ≈0.2 small)

## Limitations

- The reported comparison of 1B-to-27B scaling spans a database-integration difference as well (1B without DB vs 27B with DB), so the contrast is not a pure parameter-count effect
- No discussion-section limitations were extractable from the source document; reproducibility depends on the public codebase and AgriIR_dataset (15,247 entries on HuggingFace)

## Evidence

- [results] Scaling from 1B to 27B parameters yields substantial gains (Δ=+0.172, p<0.001, d=0.82, large effect): [evidence span withheld — anchor status: not_found]
- [results] database integration shows negligible impact for Llama3.2:3B (p=0.204, d=0.14, Δ=-0.034): [evidence span withheld — anchor status: not_found]
