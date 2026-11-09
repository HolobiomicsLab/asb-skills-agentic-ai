---
name: paired-statistical-significance-testing
description: Use when two retrieval configurations are scored on the same set of queries
  and you must verify whether a reported difference (e.g. scaling from 1B to 27B parameters,
  or toggling database integration) is statistically supported.
when_to_use_negative:
- Comparisons that are not paired on the same queries (e.g. comparing AgriIR_Gemma-3-27B
  composite 0.820±0.208 against an unpaired standalone baseline like ChatGPT-4o's
  0.840±0.233) require the paper's own unpaired testing setup instead
- Input is aggregate summary statistics only (means and standard deviations like 0.820±0.208)
  with no per-query scores available — the paired t-test and Cohen's d cannot be recomputed
  without per-query values
- Queries outside the 191 annotated AgriIR policy set — joining scores on query ID
  only works for the shared annotated query set
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: AgriIR
  role: Framework whose per-query composite scores for each configuration are the
    inputs to the paired tests
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Llama3.2:3B
  role: Configuration assessed with and without database integration to form the paired
    ablation comparison
- name: Gemma3:1B
  role: Low-parameter configuration (w/o database) paired against Gemma3:27B for the
    scaling comparison
- name: Gemma3:27B
  role: High-parameter configuration (with database) paired against Gemma3:1B for
    the scaling comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:57e24a48ec0d4ba341a8b69cebe8b0369349eddb8a8cd6195c186a068cd5ec9b
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
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# paired-statistical-significance-testing

## Summary

Validates reported system-level comparisons in AgriIR by recomputing paired t-tests and Cohen's d over per-query composite scores for 191 annotated agricultural policy queries, checking both model-scale gains (Gemma3:1B vs Gemma3:27B) and database-integration ablations (Llama3.2:3B with vs without DB).

## When to use

Use when two retrieval configurations are scored on the same set of queries and you must verify whether a reported difference (e.g. scaling from 1B to 27B parameters, or toggling database integration) is statistically supported.

## When NOT to use

- Comparisons that are not paired on the same queries (e.g. comparing AgriIR_Gemma-3-27B composite 0.820±0.208 against an unpaired standalone baseline like ChatGPT-4o's 0.840±0.233) require the paper's own unpaired testing setup instead
- Input is aggregate summary statistics only (means and standard deviations like 0.820±0.208) with no per-query scores available — the paired t-test and Cohen's d cannot be recomputed without per-query values
- Queries outside the 191 annotated AgriIR policy set — joining scores on query ID only works for the shared annotated query set

## Inputs

- Per-query composite score files for four AgriIR configurations (Llama3.2:3B ± database, Gemma3:1B w/o DB, Gemma3:27B with DB), joined on query ID
- AgriIR benchmark of 191 annotated agricultural policy queries (30 human annotators)
- Reported statistics to verify: scaling Δ=+0.172, p<0.001, d=0.82; database ablation Δ=-0.034, p=0.204, d=0.14

## Outputs

- statistical_tests.csv with paired mean deltas, t-test p-values, and Cohen's d for both comparisons, including reported and recomputed columns
- Recomputed per-query score pairs (n=191) for the scaling and database-integration comparisons

## How to apply

Load the per-query composite score files for the four AgriIR configurations (Llama3.2:3B with and without database integration, Gemma3:1B without database, Gemma3:27B with database) and join them on query ID across the 191 annotated queries. Compute the paired mean difference between AgriIR_Gemma3:1B (w/o DB) and AgriIR_Gemma3:27B and run a paired t-test over the 191 per-query score pairs, then compute Cohen's d for this paired scaling comparison. Repeat the paired t-test and Cohen's d computation for the with/without-database comparison of Llama3.2:3B. Compare recomputed deltas, p-values, and effect sizes against the reported values — scaling: Δ=+0.172, p<0.001, d=0.82 (large effect); database ablation: Δ=-0.034, p=0.204, d=0.14 (negligible) — and write all statistics, including both reported and recomputed columns, to statistical_tests.csv. The rationale is that the paper claims architectural design rather than model scale drives reliability, so effect sizes (not just p-values) are needed to judge whether the 27B gain is practically meaningful.

## Related tools

- **AgriIR** (Framework whose per-query composite scores for each configuration are the inputs to the paired tests) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Llama3.2:3B** (Configuration assessed with and without database integration to form the paired ablation comparison)
- **Gemma3:1B** (Low-parameter configuration (w/o database) paired against Gemma3:27B for the scaling comparison)
- **Gemma3:27B** (High-parameter configuration (with database) paired against Gemma3:1B for the scaling comparison)

## Examples

```
from scipy import stats; d = db['composite'] - nodb['composite']; t, p = stats.ttest_rel(db['composite'], nodb['composite']); print(d.mean(), p)
```

## Evaluation signals

- Recomputed scaling delta, p-value, and Cohen's d match reported values: Δ=+0.172, p<0.001, d=0.82 (large effect)
- Recomputed database-ablation statistics match: Δ=-0.034, p=0.204, d=0.14 (negligible impact for Llama3.2:3B)
- Per-query score pairs join cleanly on query ID with exactly 191 matched pairs across all four configurations
- statistical_tests.csv contains both reported and recomputed columns for every statistic
- Effect-size interpretation is consistent: d=0.82 labelled large, d=0.14 labelled negligible, matching the paper's claims

## Limitations

- The p=0.204 database result supports only 'negligible impact' for Llama3.2:3B — absence of significance is not evidence of equivalence for other models
- Cohen's d of 0.82 for the 1B-to-27B scaling gain qualifies the paper's hypothesis that architecture (not scale) compensates for parameter count, since a large effect remains
- No discussion-section limitations were extracted from the source document; reproducibility gaps could not be assessed

## Evidence

- [results] Scaling from 1B to 27B parameters yields substantial gains (Δ=+0.172, p<0.001, d=0.82, large effect), while database integration shows negligible impact: [evidence span withheld — anchor status: not_found]
- [other] Paired t-test workflow over 191 queries: [evidence span withheld — anchor status: not_found]
