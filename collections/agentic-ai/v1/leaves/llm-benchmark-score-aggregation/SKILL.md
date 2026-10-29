---
name: llm-benchmark-score-aggregation
description: Use when you have per-model evaluated outputs from an LLM benchmark (e.g.
  results/experiment2 and experiment1 outputs) and must recompute or verify rank-based
  repeated-measures statistics like Friedman chi-squared across models.
when_to_use_negative:
- The score matrix does not contain matched repeated measures for all k models across
  the same benchmark cases (Friedman requires paired rankings across the same conditions)
- Only a single model or fewer than the full 13-model set is evaluated, since the
  rank-based test is defined across all models jointly
- The goal is pairwise model comparison rather than the omnibus Friedman test across
  all models
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Used with pandas to aggregate per-model mean scores (C_detect, C_extract)
    and with scipy.stats.friedmanchisquare to run the Friedman tests
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: One of the 13 evaluated models contributing scores to the detection case (best
    detection model, mean C_detect = 49.57%)
  repo: https://github.com/sydneyanuyah/CausalDiscovery
- name: Qwen2.5-Coder-32B-Instruct
  role: One of the 13 evaluated models contributing scores to the extraction case
    (best extraction model, mean C_extract = 47.12%)
  repo: https://github.com/sydneyanuyah/CausalDiscovery
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:185e8c8f41e2486ab7bcde7634bdd0065789e9c124a235011d9e136a1f12e238
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sydneyanuyah/CausalDiscovery
  tools:
  - DeepSeek-R1-Distill-Llama-70B
  - Python
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# llm-benchmark-score-aggregation

## Summary

Aggregate per-experiment, per-model benchmark scores (C_detect, C_extract) from evaluated outputs and verify reported Friedman test statistics across the 13 evaluated open-source LLMs using scipy.stats.friedmanchisquare in Python.

## When to use

Use when you have per-model evaluated outputs from an LLM benchmark (e.g. results/experiment2 and experiment1 outputs) and must recompute or verify rank-based repeated-measures statistics like Friedman chi-squared across models.

## When NOT to use

- The score matrix does not contain matched repeated measures for all k models across the same benchmark cases (Friedman requires paired rankings across the same conditions)
- Only a single model or fewer than the full 13-model set is evaluated, since the rank-based test is defined across all models jointly
- The goal is pairwise model comparison rather than the omnibus Friedman test across all models

## Inputs

- Per-model evaluated output files under results/experiment2, grouped by model and prompting strategy
- Experiment 1 causal-detection model outputs from experiments/experiment1
- Per-model mean score tables (C_detect, C_extract) across the 13 evaluated models
- Reported target statistics: chi2 = 157.018 (detection) and chi2 = 229.624 (extraction), p < 0.0001

## Outputs

- friedman_test_verification.tsv with recomputed chi2, p-value, significance flag, and match/mismatch verdict for detection and extraction cases
- Aggregated per-model mean score matrix across 13 models for each of the two cases

## How to apply

Load the per-model evaluated outputs from results/experiment2 and the experiment1 model outputs, aggregating per-model mean scores (C_detect for causal detection, C_extract for causal extraction) with Python/pandas. Assemble the score matrix across the 13 evaluated models named in the paper (DeepSeek-R1-Distill-Llama-70B, Qwen2.5-Coder-32B-Instruct, Llama-3-8B, etc.) for the detection and extraction cases separately. Apply the Friedman non-parametric repeated-measures test (rank-based, k = 13 models) to each case with scipy.stats.friedmanchisquare. Compare each recomputed chi-square statistic and p-value against the reported values — chi2 = 157.018 (detection) and chi2 = 229.624 (extraction), both p < 0.0001 — recording match/mismatch verdicts per case.

## Related tools

- **Python** (Used with pandas to aggregate per-model mean scores (C_detect, C_extract) and with scipy.stats.friedmanchisquare to run the Friedman tests) — https://github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (One of the 13 evaluated models contributing scores to the detection case (best detection model, mean C_detect = 49.57%)) — https://github.com/sydneyanuyah/CausalDiscovery
- **Qwen2.5-Coder-32B-Instruct** (One of the 13 evaluated models contributing scores to the extraction case (best extraction model, mean C_extract = 47.12%)) — https://github.com/sydneyanuyah/CausalDiscovery

## Examples

```
python -c "import pandas as pd; from scipy.stats import friedmanchisquare; chi2, p = friedmanchisquare(*[m['C_detect'] for m in model_scores]); print(chi2, p)"
```

## Evaluation signals

- Recomputed detection chi2 equals 157.018 and recomputed extraction chi2 equals 229.624
- Both recomputed p-values are p < 0.0001, confirming the significance verdicts
- The score matrix covers exactly the 13 evaluated models listed in the paper (e.g. DeepSeek-R1-Distill-Llama-70B, Qwen2.5-Coder-32B-Instruct)
- friedman_test_verification.tsv contains chi2, p-value, significance flag, and match/mismatch verdict rows for both the detection and extraction cases
- Per-model aggregate means match published values such as C_detect = 49.57% for DeepSeek-R1-Distill-Llama-70B and C_extract = 47.12% for Qwen2.5-Coder-32B-Instruct

## Limitations

- Verification depends on all evaluated outputs being present in results/experiment2 and experiments/experiment1; missing per-model output files break the k = 13 matrix
- Model outputs are generated under fixed settings (seed 4000; Experiment 2 defaults of batch size 64, top-k 40, temperature 0.7, max new tokens 256), so recomputed scores only reproduce the paper under the same configuration
- Experiment 1 uses 4-bit model loading, so scores may not match runs done at other precisions

## Evidence

- [other] Friedman test aggregation workflow: [evidence span withheld — anchor status: not_found]
- [other] Reported statistics to verify: [evidence span withheld — anchor status: not_found]
- [readme] Evaluated output locations: "`results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
- [readme] Fixed run configuration: "The scripts use seed `4000`. The Experiment 1 script uses 4-bit model loading."
- [abstract] 13-model benchmark scope: "We investigate this ability by testing 13 open-source LLMs on a fundamental task: pairwise causal discovery (PCD) from text."
