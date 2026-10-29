---
name: cross-model-performance-comparison
description: Use when per-model, per-experiment benchmark outputs exist for multiple
  LLMs and you need to statistically verify that reported model differences hold,
  or to rank models on C_detect/C_extract scores.
when_to_use_negative:
- Input is raw inference text outputs rather than scored/evaluated per-model results
  — score aggregation must precede the Friedman test
- Comparing fewer than 3 paired model conditions, where the rank-based Friedman repeated-measures
  design (k = 13 here) does not apply
- The goal is pairwise significance between two specific models rather than omnibus
  cross-model difference testing
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3360
tools:
- name: Python
  role: pandas for aggregating per-model mean scores and scipy.stats.friedmanchisquare
    for the Friedman test across 13 models
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: One of the evaluated models contributing C_detect/C_extract scores to the
    comparison; the best detection model at 49.57%
- name: Qwen2.5-Coder-32B-Instruct
  role: One of the evaluated models contributing C_extract scores; the best extraction
    model at 47.12%
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:e1a0dcf3177ec630f74cb1947038b83fd880e9ee367abfd9e811d44607c555be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - DeepSeek-R1-Distill-Llama-70B
  - Python
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Cross-Model Performance Comparison

## Summary

Aggregates per-model benchmark scores (C_detect, C_extract) across the 13 evaluated open-source LLMs and applies the Friedman non-parametric repeated-measures test to verify whether reported cross-model differences (detection chi-squared = 157.018; extraction chi-squared = 229.624, both p < 0.0001) reproduce from raw per-experiment results.

## When to use

Use when per-model, per-experiment benchmark outputs exist for multiple LLMs and you need to statistically verify that reported model differences hold, or to rank models on C_detect/C_extract scores.

## When NOT to use

- Input is raw inference text outputs rather than scored/evaluated per-model results — score aggregation must precede the Friedman test
- Comparing fewer than 3 paired model conditions, where the rank-based Friedman repeated-measures design (k = 13 here) does not apply
- The goal is pairwise significance between two specific models rather than omnibus cross-model difference testing

## Inputs

- Per-model evaluated outputs under results/experiment2/ grouped by model and prompting strategy
- Experiment 1 model outputs from experiments/experiment1/
- Expected reference statistics: chi2 = 157.018 (detection) and chi2 = 229.624 (extraction)

## Outputs

- friedman_test_verification.tsv with recomputed chi2, p-value, significance flag, and match/mismatch verdict for detection and extraction cases
- Aggregated per-model mean score matrix (C_detect, C_extract) across 13 models

## How to apply

Load evaluated outputs from results/experiment2 and the experiment1 model outputs, aggregating per-model mean scores (e.g. C_detect for causal detection, C_extract for causal extraction) with Python/pandas. Assemble a score matrix across the 13 evaluated models (e.g. DeepSeek-R1-Distill-Llama-70B, Qwen2.5-Coder-32B-Instruct, Mistral-7B-Instruct-v0.3, Mixtral-8x7B-Instruct-v0.1) separately for the detection case (k = 13 models on the 3,008-row detection dataset) and the extraction case (8,925-row consolidated dataset). Apply scipy.stats.friedmanchisquare, the rank-based non-parametric repeated-measures test, to each case. Compare each recomputed chi-square statistic and p-value against the reported detection statistic (chi2 = 157.018) and extraction statistic (chi2 = 229.624), both with p < 0.0001, recording match/mismatch verdicts. Write recomputed chi2, p-value, significance flag, and verification verdict per case to friedman_test_verification.tsv.

## Related tools

- **Python** (pandas for aggregating per-model mean scores and scipy.stats.friedmanchisquare for the Friedman test across 13 models) — https://github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (One of the evaluated models contributing C_detect/C_extract scores to the comparison; the best detection model at 49.57%)
- **Qwen2.5-Coder-32B-Instruct** (One of the evaluated models contributing C_extract scores; the best extraction model at 47.12%)

## Examples

```
from scipy.stats import friedmanchisquare; chi2, p = friedmanchisquare(*model_score_matrix.T)
```

## Evaluation signals

- Recomputed detection chi2 matches 157.018 and extraction chi2 matches 229.624
- Both recomputed p-values are p < 0.0001 and significance verdicts are confirmed in friedman_test_verification.tsv
- Score matrix includes exactly 13 evaluated models (DeepSeek-R1-0528-Qwen3-8B through Qwen2.5-Coder-32B-Instruct)
- Aggregated means reproduce reported headline values (C_detect = 49.57% for DeepSeek-R1-Distill-Llama-70B; C_extract = 47.12% for Qwen2.5-Coder-32B-Instruct)

## Limitations

- Friedman results depend on which per-model mean scores are aggregated; differing prompting-strategy groupings in results/experiment2 (e.g. zero-shot, few-shot, CoT, Least-to-Most, ReAct) can change the matrix
- The test verifies omnibus cross-model differences only; no per-pair post-hoc model comparisons are reported
- Only 13 of the models run in the repository are included in the paper's evaluated set; extra outputs must be excluded

## Evidence

- [methods] Apply the Friedman non-parametric repeated-measures test across models (rank-based, k = 13 models) to each case using scipy.stats.friedmanchisquare: [evidence span withheld — anchor status: not_found]
- [results] Recomputation of the Friedman tests yields chi-squared = 157.018 (detection) and chi-squared = 229.624 (extraction), both p < 0.0001, verifying reported statistics: [evidence span withheld — anchor status: not_found]
- [abstract] The best model for detection, DeepSeek-R1-Distill-Llama-70B, only achieved a mean score of 49.57% (C_detect): "The best model for detection, DeepSeek-R1-Distill-Llama-70B, only achieved a mean score of 49.57\% ($C_{detect}$)"
- [readme] results/experiment2/ contains evaluated outputs and timing files grouped by model and prompting strategy: "`results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
