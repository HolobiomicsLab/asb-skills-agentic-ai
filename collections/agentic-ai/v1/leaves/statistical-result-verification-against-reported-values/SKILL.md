---
name: statistical-result-verification-against-reported-values
description: Use when a paper reports omnibus statistical tests (e.g., Friedman chi-squared
  = 157.018 and 229.624, p < 0.0001) that should be independently recomputed from
  the underlying per-model, per-experiment score data.
when_to_use_negative:
- No per-model, per-experiment raw score outputs are available — Friedman tests cannot
  be recomputed from summary tables alone
- The reported statistic is parametric (e.g., ANOVA F-test) or the design violates
  repeated-measures assumptions (non-paired scores across models)
- Fewer than 3 paired model conditions (k < 3), where the Friedman test is not appropriate
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3360
tools:
- name: Python
  role: Aggregates per-model mean scores with pandas and runs scipy.stats.friedmanchisquare
    to recompute the Friedman tests
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: Best detection model (mean C_detect = 49.57%) whose scores form one column
    of the Friedman test matrix
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Best extraction model (mean C_extract = 47.12%) whose scores form one column
    of the Friedman test matrix
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:b53526710dac5c9a0cea573a0b8900d08427323cfff37da98187806516a8db3d
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
yes_rate: 0.6923076923076923
n_claims: 13
---

# Statistical Result Verification Against Reported Values

## Summary

Recompute reported non-parametric statistics (Friedman chi-squared tests across 13 LLMs for causal detection and causal extraction) from per-model benchmark scores and verify they match the published values and significance thresholds.

## When to use

Use when a paper reports omnibus statistical tests (e.g., Friedman chi-squared = 157.018 and 229.624, p < 0.0001) that should be independently recomputed from the underlying per-model, per-experiment score data.

## When NOT to use

- No per-model, per-experiment raw score outputs are available — Friedman tests cannot be recomputed from summary tables alone
- The reported statistic is parametric (e.g., ANOVA F-test) or the design violates repeated-measures assumptions (non-paired scores across models)
- Fewer than 3 paired model conditions (k < 3), where the Friedman test is not appropriate

## Inputs

- Per-model evaluated outputs from results/experiment2 (causal extraction case)
- Experiment 1 model outputs from experiments/experiment1 (causal detection case, 3,008-row dataset)
- Reported target statistics: chi2 = 157.018 (detection) and chi2 = 229.624 (extraction), both p < 0.0001

## Outputs

- friedman_test_verification.tsv with recomputed chi2, p-value, significance flag, and match/mismatch verdict for detection and extraction cases

## How to apply

Load the per-model evaluated outputs from results/experiment2 and the experiment1 model outputs, aggregating per-model mean scores (C_detect for detection, C_extract for extraction) with Python/pandas. Assemble a score matrix across the 13 evaluated models (DeepSeek-R1-Distill-Llama-70B, Qwen2.5-Coder-32B-Instruct, Llama-3-8B, Mixtral-8x7B-Instruct-v0.1, etc.) separately for the detection and extraction cases. Apply the Friedman rank-based repeated-measures test (k = 13 models) to each case with scipy.stats.friedmanchisquare. Compare each recomputed chi-squared and p-value against the reported detection statistic (chi2 = 157.018, p < 0.0001) and extraction statistic (chi2 = 229.624, p < 0.0001), recording match/mismatch verdicts. Write recomputed chi2, p-value, significance flag, and verification verdict per case to a TSV (e.g., friedman_test_verification.tsv).

## Related tools

- **Python** (Aggregates per-model mean scores with pandas and runs scipy.stats.friedmanchisquare to recompute the Friedman tests) — https://github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (Best detection model (mean C_detect = 49.57%) whose scores form one column of the Friedman test matrix) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Best extraction model (mean C_extract = 47.12%) whose scores form one column of the Friedman test matrix) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
from scipy.stats import friedmanchisquare; chi2, p = friedmanchisquare(*[model_scores[m] for m in models])
```

## Evaluation signals

- Recomputed detection Friedman chi2 equals 157.018 with p < 0.0001
- Recomputed extraction Friedman chi2 equals 229.624 with p < 0.0001
- Both significance verdicts confirmed as p < 0.0001 in the output TSV
- Score matrix contains exactly 13 evaluated models as listed in the paper (e.g., DeepSeek-R1-0528-Qwen3-8B, Mistral-7B-Instruct-v0.3, Qwen2.5-7B-Instruct)
- Detection and extraction cases are tested separately, each producing its own chi2 and p-value row in friedman_test_verification.tsv

## Limitations

- Verification depends on exact model roster and aggregation (mean C_detect / C_extract) matching the paper; different prompt strategies or task slices (2a-2h) change the input matrix and thus the statistic
- Small rounding differences (e.g., to 3 decimal places of chi2) may be the acceptance tolerance; a mismatch does not necessarily imply a reporting error
- The discussion section text is absent from the article source, so any caveats the authors gave about these tests cannot be extracted

## Evidence

- [other] Recomputation of the Friedman tests across models yields chi-squared = 157.018 (p < 0.0001) for the detection case and chi-squared = 229.624 (p < 0.0001) for the extraction case: [evidence span withheld — anchor status: not_found]
- [methods] Apply the Friedman non-parametric repeated-measures test across models (rank-based, k = 13 models) to each case using scipy.stats.friedmanchisquare in Python.: [evidence span withheld — anchor status: not_found]
- [readme] results/experiment2 contains evaluated outputs and timing files grouped by model and prompting strategy.: "`results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
- [abstract] The best model for detection, DeepSeek-R1-Distill-Llama-70B, only achieved a mean score of 49.57%: "The best model for detection, DeepSeek-R1-Distill-Llama-70B, only achieved a mean score of 49.57\% ($C_{detect}$)"
