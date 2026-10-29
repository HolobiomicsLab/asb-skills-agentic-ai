---
name: friedman-omnibus-significance-testing
description: Use when you have per-model scores (e.g. C_detect or C_extract means)
  across k>=3 models measured on shared benchmark cases and need to test whether model
  differences are statistically significant, or to verify published Friedman statistics.
when_to_use_negative:
- Scores are independent samples rather than repeated measures over shared benchmark
  cases (use a Kruskal-Wallis or ANOVA-style test instead)
- You need pairwise post-hoc model comparisons rather than the omnibus across-models
  test (Friedman only establishes that some models differ)
- Fewer than 3 models are being compared (k = 13 in this benchmark; Friedman requires
  k >= 3)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implementation language for aggregating per-model mean scores with pandas
    and running scipy.stats.friedmanchisquare
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: Best-performing detection model (mean C_detect 49.57%) whose scores feed the
    detection-case Friedman test
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: Best-performing extraction model (C_extract 47.12%) whose scores feed the
    extraction-case Friedman test
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:2d298183981b1f09071bd7e92ae31f68520c6f5859643692f0d7418c907951cb
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# friedman-omnibus-significance-testing

## Summary

Recompute the Friedman non-parametric repeated-measures omnibus test across the 13 evaluated open-source LLMs on the pairwise causal discovery benchmark, verifying reported chi-squared statistics (157.018 for detection, 229.624 for extraction) from per-model benchmark scores.

## When to use

Use when you have per-model scores (e.g. C_detect or C_extract means) across k>=3 models measured on shared benchmark cases and need to test whether model differences are statistically significant, or to verify published Friedman statistics.

## When NOT to use

- Scores are independent samples rather than repeated measures over shared benchmark cases (use a Kruskal-Wallis or ANOVA-style test instead)
- You need pairwise post-hoc model comparisons rather than the omnibus across-models test (Friedman only establishes that some models differ)
- Fewer than 3 models are being compared (k = 13 in this benchmark; Friedman requires k >= 3)

## Inputs

- Per-model evaluated outputs from results/experiment2 (causal extraction, grouped by model and prompting strategy)
- Experiment 1 model outputs (causal detection, exp1.csv-based, 3008-row dataset)
- Aggregated per-model mean score matrix across the 13 evaluated open-source LLMs (e.g. C_detect, C_extract)

## Outputs

- friedman_test_verification.tsv with recomputed chi2, p-value, significance flag, and verification verdict for both the detection and extraction cases

## How to apply

Load the per-model evaluated outputs from results/experiment2 (causal extraction) and the experiment1 model outputs (causal detection), aggregating per-model mean scores such as C_detect and C_extract with Python/pandas. Assemble the score matrix across the 13 evaluated models (DeepSeek-R1-Distill-Llama-70B, Qwen2.5-Coder-32B-Instruct, etc.) separately for the detection case and the extraction case. Apply the rank-based Friedman repeated-measures test across models (k = 13) using scipy.stats.friedmanchisquare for each case. Compare each recomputed chi-squared statistic and p-value against the reported detection statistic (chi2 = 157.018, p < 0.0001) and extraction statistic (chi2 = 229.624, p < 0.0001), recording match/mismatch verdicts. The rationale is that Friedman's rank-based design treats benchmark cases as repeated measures and is appropriate for non-normal score distributions of LLM benchmark outputs.

## Related tools

- **Python** (Implementation language for aggregating per-model mean scores with pandas and running scipy.stats.friedmanchisquare) — https://github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (Best-performing detection model (mean C_detect 49.57%) whose scores feed the detection-case Friedman test) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (Best-performing extraction model (C_extract 47.12%) whose scores feed the extraction-case Friedman test) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
from scipy.stats import friedmanchisquare; chi2, p = friedmanchisquare(*[detection_scores[m] for m in models_13])
```

## Evaluation signals

- Recomputed detection-case chi2 matches 157.018 with p < 0.0001
- Recomputed extraction-case chi2 matches 229.624 with p < 0.0001
- Significance verdicts are confirmed in friedman_test_verification.tsv
- Score matrix covers all 13 evaluated models for each case before running the test
- Both cases yield significance flags (p < 0.0001) consistent with the paper's reported omnibus rejection

## Limitations

- Verification depends on reproducing the exact per-model aggregation from the repository's evaluated outputs; deviations in score assembly change the chi2 statistic
- Inference reproducibility depends on fixed run settings — scripts use seed 4000, and Experiment 2 defaults to batch size 64, top-k 40, temperature 0.7, and max new tokens 256
- The Friedman test is omnibus only; significant chi2 does not identify which model pairs differ
- The paper's discussion section is absent, so no documented limitations or caveats from the authors are available

## Evidence

- [methods] Apply the Friedman non-parametric repeated-measures test across models (rank-based, k = 13 models) to each case using scipy.stats.friedmanchisquare in Python.: [evidence span withheld — anchor status: not_found]
- [results] Recomputation of the Friedman tests across models yields chi-squared = 157.018 (p < 0.0001) for the detection case and chi-squared = 229.624 (p < 0.0001) for the extraction case.: [evidence span withheld — anchor status: not_found]
- [intro] The study evaluates 13 open-source large language models on pairwise causal discovery from text.: "The study evaluates 13 open-source large language models on pairwise causal discovery from text."
- [readme] results/experiment2/ contains evaluated outputs and timing files grouped by model and prompting strategy.: "results/experiment2/ contains evaluated outputs and timing files grouped by model and prompting strategy."
