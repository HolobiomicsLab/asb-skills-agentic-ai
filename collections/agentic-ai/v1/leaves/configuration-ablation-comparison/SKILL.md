---
name: configuration-ablation-comparison
description: Use when comparing two paired system configurations (e.g., a scale ablation
  or a component-removal ablation) on identical per-query evaluation scores to decide
  whether the reported delta is statistically significant.
when_to_use_negative:
- When comparing AgriIR configurations against standalone baselines (ChatGPT-4o, Gemini
  2.5 Flash, GPT-OSS-120B) rather than internal scale or component ablations — use
  the baseline-equivalence tests (e.g., p=0.493 vs ChatGPT-4o) instead.
- When per-query scores cannot be joined on query ID (unpaired or different query
  sets), since the paired t-test over 191 matched pairs is the core of this skill.
- When the outcome is a single aggregate score without per-query variance, which cannot
  support a paired test or Cohen's d.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3360
tools:
- name: AgriIR
  role: Configurable RAG framework whose per-configuration runs (model scale and database
    integration variants) generate the per-query composite scores being compared
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Llama3.2:3B
  role: Model whose with/without-database variants form the component-ablation comparison
    (p=0.204, d=0.14, Δ=-0.034)
- name: Gemma3:1B
  role: Small-scale configuration (without database) serving as the low end of the
    scaling comparison
- name: Gemma3:27B
  role: Large-scale configuration (with database) serving as the high end of the scaling
    comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:5cf20c2880885c8e993cfd5781545a52ca261f4e56d6208436b3f1c730c9889a
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
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# configuration-ablation-comparison

## Summary

Statistically validates system-configuration effects in the AgriIR RAG framework by running paired t-tests and Cohen's d over per-query composite scores for model-scale (Gemma3:1B vs Gemma3:27B) and database-integration ablations (Llama3.2:3B with/without DB) across 191 annotated queries.

## When to use

Use when comparing two paired system configurations (e.g., a scale ablation or a component-removal ablation) on identical per-query evaluation scores to decide whether the reported delta is statistically significant.

## When NOT to use

- When comparing AgriIR configurations against standalone baselines (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B) rather than internal scale or component ablations — use the baseline-equivalence tests (e.g., p=0.493 vs ChatGPT-4o) instead.
- When per-query scores cannot be joined on query ID (unpaired or different query sets), since the paired t-test over 191 matched pairs is the core of this skill.
- When the outcome is a single aggregate score without per-query variance, which cannot support a paired test or Cohen's d.

## Inputs

- Per-query composite score files for AgriIR_Gemma3:27B (with database)
- Per-query composite score files for AgriIR_Gemma3:1B (without database)
- Per-query composite score files for AgriIR_Llama3.2:3B with database integration
- Per-query composite score files for AgriIR_Llama3.2:3B without database integration
- Query-ID join key across the 191 annotated agricultural policy queries

## Outputs

- statistical_tests.csv with paired mean deltas, p-values, and Cohen's d, including both reported and recomputed columns for the scaling comparison (Δ=+0.172, p<0.001, d=0.82) and the database ablation (Δ=-0.034, p=0.204, d=0.14)

## How to apply

Load per-query composite score files for the four AgriIR configurations (Llama3.2:3B with and without database integration, Gemma3:1B without database, Gemma3:27B with database) and join them on query ID across the 191 annotated agricultural policy queries. Compute the paired mean difference between AgriIR_Gemma3:1B (w/o DB) and AgriIR_Gemma3:27B, run a paired t-test over the 191 per-query score pairs, and compute Cohen's d for the scaling comparison; expected reproduction targets are Δ=+0.172, p<0.001, d=0.82 (large effect). Repeat the paired t-test and Cohen's d for the with/without-database comparison of Llama3.2:3B, expecting negligible impact (Δ=-0.034, p=0.204, d=0.14). Interpret d≈0.8 as a large effect and p≥0.05 with small d as negligible impact, then write both reported and recomputed statistics side by side to statistical_tests.csv so recomputation directly audits the paper's claims.

## Related tools

- **AgriIR** (Configurable RAG framework whose per-configuration runs (model scale and database integration variants) generate the per-query composite scores being compared) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Llama3.2:3B** (Model whose with/without-database variants form the component-ablation comparison (p=0.204, d=0.14, Δ=-0.034))
- **Gemma3:1B** (Small-scale configuration (without database) serving as the low end of the scaling comparison)
- **Gemma3:27B** (Large-scale configuration (with database) serving as the high end of the scaling comparison)

## Examples

```
from scipy import stats; t, p = stats.ttest_rel(scores_1b, scores_27b); d = (scores_27b - scores_1b).mean() / (scores_27b - scores_1b).std(ddof=1)
```

## Evaluation signals

- Recomputed scaling delta matches Δ=+0.172 with p<0.001 and Cohen's d≈0.82 (large effect) over 191 paired scores.
- Recomputed database-ablation delta matches Δ=-0.034 with p=0.204 and d≈0.14, confirming negligible impact for Llama3.2:3B.
- Per-query score join on query ID yields exactly 191 matched pairs with no dropped queries.
- statistical_tests.csv contains both reported and recomputed columns for every statistic, allowing direct diff against the paper's values.
- Paired scores are on the same composite-scale range (e.g., AgriIR_Gemma-3-27B composite 0.820±0.208), so deltas are interpretable on the 0–1 composite metric.

## Limitations

- The source card workflow references per-query composite score files that are not listed as released artifacts in the visible README; availability must be confirmed in the AgriIR repo or the AgriIR_dataset HuggingFace release.
- No discussion-section text was present in the source document, so limitations, reproducibility gaps, and caveats could not be extracted from the paper itself.
- Cohen's d on paired composite scores depends on the paired-difference variance; sensitivity to score non-normality across only 191 queries is unquantified in the paper.

## Evidence

- [results] Scaling from 1B to 27B parameters yields substantial gains (Δ=+0.172, p<0.001, d=0.82, large effect), while database integration shows negligible impact for Llama3.2:3B (p=0.204, d=0.14, Δ=-0.034).: [evidence span withheld — anchor status: not_found]
