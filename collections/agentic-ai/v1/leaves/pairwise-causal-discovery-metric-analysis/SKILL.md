---
name: pairwise-causal-discovery-metric-analysis
description: Use when per-experiment, per-model benchmark scores (e.g. C_detect/C_extract)
  are available and you must verify or reproduce reported Friedman chi-squared statistics
  and p-values across models for detection and extraction cases.
when_to_use_negative:
- You lack per-experiment, per-model score outputs and only have aggregate summary
  statistics — the Friedman test requires the full score matrix across models
- The analysis goal is per-dataset performance breakdown rather than across-model
  significance testing of the detection and extraction skills
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Aggregates per-model mean scores with pandas and runs scipy.stats.friedmanchisquare
    on the score matrices
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: Best detection model (49.57% C_detect) whose scores feed the detection-case
    Friedman test
- name: Qwen2.5-Coder-32B-Instruct
  role: Best extraction model (47.12% C_extract) whose scores feed the extraction-case
    Friedman test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:1186de5104155474eb8c63c50177e6ae9e5764e3d3fbc39ed8bd0f65064adcab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - DeepSeek-R1-Distill-Llama-70B
  - Python
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pairwise-causal-discovery-metric-analysis

## Summary

Recomputes Friedman non-parametric repeated-measures tests across the 13 evaluated open-source LLMs to verify reported model-difference statistics for the causal detection (C_detect) and causal extraction (C_extract) skills in the pairwise causal discovery benchmark.

## When to use

Use when per-experiment, per-model benchmark scores (e.g. C_detect/C_extract) are available and you must verify or reproduce reported Friedman chi-squared statistics and p-values across models for detection and extraction cases.

## When NOT to use

- You lack per-experiment, per-model score outputs and only have aggregate summary statistics — the Friedman test requires the full score matrix across models
- The analysis goal is per-dataset performance breakdown rather than across-model significance testing of the detection and extraction skills

## Inputs

- Per-model evaluated outputs from results/experiment2 (grouped by model and prompting strategy)
- Experiment 1 model outputs from experiments/experiment1 (causal-detection data, 3,008-row dataset file)
- Experiment 2 evaluated outputs for the causal-extraction task (task slices 2a-2h, 8,925-row consolidated dataset)

## Outputs

- friedman_test_verification.tsv with recomputed chi2, p-value, significance flag, and verification verdict for the detection and extraction cases

## How to apply

Load the per-model evaluated outputs from results/experiment2 and the experiment1 model outputs, aggregating per-model mean scores (C_detect for causal detection, C_extract for causal extraction) with Python/pandas. Assemble the score matrix across the 13 evaluated models (e.g. DeepSeek-R1-Distill-Llama-70B, Qwen2.5-Coder-32B-Instruct, Llama-3.2-3B, Mixtral-8x7B-Instruct-v0.1, etc.) separately for the detection and extraction cases. Apply the rank-based Friedman test with k = 13 models using scipy.stats.friedmanchisquare on each case. Compare each recomputed chi-square statistic and p-value against the reported values — detection chi2 = 157.018 and extraction chi2 = 229.624, both p < 0.0001 — and record match/mismatch verdicts.

## Related tools

- **Python** (Aggregates per-model mean scores with pandas and runs scipy.stats.friedmanchisquare on the score matrices) — https://github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (Best detection model (49.57% C_detect) whose scores feed the detection-case Friedman test)
- **Qwen2.5-Coder-32B-Instruct** (Best extraction model (47.12% C_extract) whose scores feed the extraction-case Friedman test)

## Examples

```
from scipy.stats import friedmanchisquare; chi2, p = friedmanchisquare(*detect_scores_per_model)
```

## Evaluation signals

- Recomputed detection-case chi2 matches 157.018 with p < 0.0001
- Recomputed extraction-case chi2 matches 229.624 with p < 0.0001
- Significance verdicts for both cases are confirmed in friedman_test_verification.tsv
- Score matrix covers all 13 evaluated models (k = 13) for each case before ranking

## Limitations

- Verification depends on exact reproduction of per-model evaluated outputs; the repository README notes results are grouped by model and prompting strategy, so mixing strategies changes the score matrix
- The benchmark's low absolute performance (best C_detect 49.57%, best C_extract 47.12%) means scores are near chance, which may compress rank differences between models

## Evidence

- [methods] Apply the Friedman non-parametric repeated-measures test across models (rank-based, k = 13 models) to each case using scipy.stats.friedmanchisquare in Python.: [evidence span withheld — anchor status: not_found]
- [results] Recomputation yields chi-squared = 157.018 detection and 229.624 extraction, both p < 0.0001.: [evidence span withheld — anchor status: not_found]
- [readme] README describes results/experiment2 containing evaluated outputs grouped by model and prompting strategy.: "`results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
- [abstract] Best model scores for detection and extraction feed the Friedman tests.: "The best model for detection, DeepSeek-R1-Distill-Llama-70B, only achieved a mean score of 49.57\% ($C_{detect}$)"
