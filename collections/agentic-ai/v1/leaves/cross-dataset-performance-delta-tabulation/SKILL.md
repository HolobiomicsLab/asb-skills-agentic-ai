---
name: cross-dataset-performance-delta-tabulation
description: Use when a paper reports a multi-dataset ablation grid (e.g., CoT chain
  counts n and per-perspective chains m) and you need to compute ablated-minus-full
  metric deltas (ACC, AUROC, F1) per dataset and visualize them.
when_to_use_negative:
- The article reports only a single full-model result with no ablation grid — there
  are no (n,m) variants to compute deltas against
- The goal is architectural analysis (e.g., prototype-guided vs context-guided decoder
  internals) rather than quantitative benchmark comparison across settings
- Results come from different MLLM evaluation setups that the table does not flag
  (e.g., mixing LLaVA-based Test Seen numbers with GPT-4-based MultiOFF numbers without
  noting the MLLM column)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: LLaVA-1.5-7B
  role: MLLM used to evaluate EVL-MCoT in the full (n=3, m=3) setting on Hateful Memes
    Test Seen; source of the baseline ACC/AUROC values
  repo: https://github.com/BGWH123/EVL-MCoT
- name: GPT-4
  role: MLLM used to evaluate EVL-MCoT in the full (n=3, m=3) setting on MultiOFF;
    source of the baseline ACC/F1 values
  repo: https://github.com/BGWH123/EVL-MCoT
- name: matplotlib
  role: Renders the grouped bar plot of per-metric deltas across CoT settings and
    datasets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:2d632d11768c0dffcf4fec29f1dade83a7e6b68065bf1785d9c38b0b74f1b785
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - GPT-4
  - LLaVA
  - LLaVA-1.5-7B
  - Long-CLIP
  - matplotlib
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Cross-dataset performance delta tabulation

## Summary

A benchmarking skill that anchors reported model results (e.g., EVL-MCoT's 75.88 ACC / 79.25 AUROC on Hateful Memes Test Seen, 70.0 ACC / 63.8 F1 on MultiOFF), then tabulates per-metric deltas for ablated CoT settings (n,m) = (1,1), (1,3), (3,1), (3,3) to quantify how chain count and per-perspective chain balance drive detection performance.

## When to use

Use when a paper reports a multi-dataset ablation grid (e.g., CoT chain counts n and per-perspective chains m) and you need to compute ablated-minus-full metric deltas (ACC, AUROC, F1) per dataset and visualize them.

## When NOT to use

- The article reports only a single full-model result with no ablation grid — there are no (n,m) variants to compute deltas against
- The goal is architectural analysis (e.g., prototype-guided vs context-guided decoder internals) rather than quantitative benchmark comparison across settings
- Results come from different MLLM evaluation setups that the table does not flag (e.g., mixing LLaVA-based Test Seen numbers with GPT-4-based MultiOFF numbers without noting the MLLM column)

## Inputs

- Article result-table values for Multi-CoT ablation settings (n, m) on Hateful Memes Test Seen / Test Unseen (ACC, AUROC)
- Article result-table values for the same settings on MultiOFF (ACC, F1)
- Full-setting baseline values: 75.88/79.25 (Test Seen, LLaVA MLLM) and 70.0/63.8 (MultiOFF, GPT-4 MLLM)

## Outputs

- Tidy delta table with columns: dataset, CoT setting (n, m), metric, absolute value, delta versus (n=3,m=3)
- Grouped bar plot (matplotlib) of per-metric deltas across CoT settings and datasets

## How to apply

Load the reported ablation and full-setting results from the article's result tables for the Multi-CoT settings (n=1,m=1), (n=1,m=3), (n=3,m=1), and (n=3,m=3) on Hateful Memes Test Seen (ACC, AUROC) and MultiOFF (ACC, F1). Anchor the baseline to the full EVL-MCoT setting (n=3,m=3): 75.88 ACC / 79.25 AUROC on Hateful Memes Test Seen (LLaVA as the MLLM) and 70.0 ACC / 63.8 F1 on MultiOFF (GPT-4 as the MLLM). Compute per-metric deltas (ablated minus full) for each configuration and each dataset, then assemble a tidy table with columns dataset, CoT setting (n, m), metric, absolute value, and delta versus (n=3,m=3). Render a grouped bar plot of the deltas with matplotlib to separate the contribution of chain count (n) from benign-chain balance (m). The key expected signal: the (n=1,m=1) w/o CoT row — 69.20 ACC / 69.70 AUROC on Test Seen, 67.40 ACC / 67.46 AUROC on Test Unseen, 61.08 ACC / 55.70 F1 on MultiOFF — shows the largest degradation (e.g., -6.68 ACC, -9.55 AUROC on Test Seen; -8.87 ACC, -7.98 F1 on MultiOFF).

## Related tools

- **LLaVA-1.5-7B** (MLLM used to evaluate EVL-MCoT in the full (n=3, m=3) setting on Hateful Memes Test Seen; source of the baseline ACC/AUROC values) — https://github.com/BGWH123/EVL-MCoT
- **GPT-4** (MLLM used to evaluate EVL-MCoT in the full (n=3, m=3) setting on MultiOFF; source of the baseline ACC/F1 values) — https://github.com/BGWH123/EVL-MCoT
- **matplotlib** (Renders the grouped bar plot of per-metric deltas across CoT settings and datasets)

## Examples

```
python -c "import matplotlib.pyplot as plt; plt.bar(settings, deltas); plt.ylabel('delta vs (n=3,m=3)'); plt.savefig('cot_ablation_deltas.png')"
```

## Evaluation signals

- The (n=3, m=3) row of the derived table must exactly match the reported values: 75.88 ACC / 79.25 AUROC on Test Seen and 70.0 ACC / 63.8 F1 on MultiOFF
- Every delta must be reproducible as (ablated value minus full value) from the source table, e.g., 69.20 - 75.88 = -6.68 ACC on Test Seen
- The w/o CoT (n=1, m=1) row must reproduce 69.20 ACC / 69.70 AUROC (Test Seen), 67.40 ACC / 67.46 AUROC (Test Unseen), and 61.08 ACC / 55.70 F1 (MultiOFF)
- The delta plot must show the (n=1, m=1) configuration with the largest negative deltas, consistent with 'removing multi-CoT reasoning entirely causes the largest degradation'
- Each dataset-metric pairing must use the correct baseline MLLM column (LLaVA for Test Seen, GPT-4 for MultiOFF) as stated in the methods

## Limitations

- Full-setting baselines come from different MLLMs (LLaVA for Hateful Memes, GPT-4 for MultiOFF), so cross-dataset delta magnitudes are not directly comparable as evidence of dataset difficulty
- The GitHub repository README notes that model definitions, configs, and evaluation scripts are 'coming soon', so the tabulation cannot currently be reproduced end-to-end from the released code
- Delta analysis quantifies the (n, m) CoT ablation only; it does not isolate other components (e.g., Long-CLIP removal) which require their own ablation rows

## Evidence

- [methods] EVL-MCoT achieves 75.88% accuracy and 79.25 AUROC on Hateful Memes Test Seen: "EVL-MCoT 75.88 79.25 75.57 79.40"
- [results] Full-setting MultiOFF baseline for delta anchoring: [evidence span withheld — anchor status: not_found]
- [results] Test Seen baseline used as the delta anchor: "it reaches an accuracy of 75.88 and AUROC of 79.25 on Test Seen"
- [methods] MLLM used for the full-setting evaluation (baseline provenance): "The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM"
- [readme] Reproducibility caveat from the repository: "scripts/ *(upcoming)*: Tools for data preprocessing, visualization, and evaluation."
