---
name: deepseek-technical-report-figure-extraction
description: Use when a technical report or article reports GPU-hour figures for distinct
  training stages (e.g., pretraining vs. post-training RL) and you must reconcile
  a stated percentage claim against the raw ratio of those reported numbers.
when_to_use_negative:
- When absolute compute cost, energy use, or dollar cost is needed rather than a relative
  GPU-hour ratio (raw GPU-hour ratios ignore GPU type differences and per-stage efficiency)
- When the input figures are already aggregated percentage shares rather than raw
  GPU-hour counts for each stage
- When reconciling compute across different accelerator classes or efficiency regimes,
  since the ratio 147K/2.8M mixes H800 post-training hours with general pretraining
  GPU-hours
edam_operation: http://edamontology.org/operation_2409
tools:
- name: NumPy
  role: Array arithmetic used to compute the 147000/2800000 GPU-hour ratio and validate
    it against the ~5% claim
- name: Python
  role: Scripting environment for the ratio computation and CSV table generation
- name: RLHF Book
  role: Source of the stated ~5% post-training compute claim and the context on post-training
    stages (instruction tuning, reward models, RL)
  repo: https://github.com/natolambert/rlhf-book
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:ca14238a32e772982e9b49a369ab936349fde07809db2ff9575b8196a9f01513
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/natolambert/rlhf-book
  tools:
  - NumPy
  - Python
  - RLHF Book
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# deepseek-technical-report-figure-extraction

## Summary

Extracts self-reported compute figures (e.g., 147K H800 GPU hours for DeepSeek R1 RL training vs. 2.8M GPU hours for DeepSeek V3 pretraining) from technical reports and reconciles them against stated claims (the book's 'only about 5% of their overall compute in post-training') via a raw GPU-hour ratio computation.

## When to use

Use when a technical report or article reports GPU-hour figures for distinct training stages (e.g., pretraining vs. post-training RL) and you must reconcile a stated percentage claim against the raw ratio of those reported numbers.

## When NOT to use

- When absolute compute cost, energy use, or dollar cost is needed rather than a relative GPU-hour ratio (raw GPU-hour ratios ignore GPU type differences and per-stage efficiency)
- When the input figures are already aggregated percentage shares rather than raw GPU-hour counts for each stage
- When reconciling compute across different accelerator classes or efficiency regimes, since the ratio 147K/2.8M mixes H800 post-training hours with general pretraining GPU-hours

## Inputs

- Reported GPU-hour figures from technical reports (147,000 H800 GPU-hours for R1 RL post-training; 2,800,000 GPU-hours for V3 pretraining)
- Stated percentage claim from the source text (~5% of overall compute in post-training)
- CSV output schema (columns: training stage, GPU hours, share of pretraining compute)

## Outputs

- Computed post-training compute ratio (≈0.0525, i.e., 5.25%)
- Absolute discrepancy vs. stated claim (≈0.25 percentage points)
- Results table post_training_compute_ratio.csv with rows for R1 RL-training and V3 pretraining stages

## How to apply

Load the reported GPU-hour figures into a NumPy array: 147,000 H800 GPU hours for DeepSeek R1 RL (post-training) and 2,800,000 GPU hours for DeepSeek V3 pretraining. Compute the ratio 147000/2800000 ≈ 0.0525 (5.25% of pretraining compute). Compare this computed ratio to the stated ~5% claim, reporting the absolute discrepancy (≈0.25 percentage points) and checking it falls within a 1 percentage-point tolerance. Write a results table with columns for training stage, GPU hours, and share of pretraining compute to post_training_compute_ratio.csv.

## Related tools

- **NumPy** (Array arithmetic used to compute the 147000/2800000 GPU-hour ratio and validate it against the ~5% claim)
- **Python** (Scripting environment for the ratio computation and CSV table generation)
- **RLHF Book** (Source of the stated ~5% post-training compute claim and the context on post-training stages (instruction tuning, reward models, RL)) — https://github.com/natolambert/rlhf-book

## Examples

```
import numpy as np; hours = np.array([147000, 2800000]); print('R1_RL_vs_V3_pretrain_ratio =', hours[0]/hours[1])
```

## Evaluation signals

- Computed ratio 147000/2800000 rounds to ≈0.0525 (5.25%)
- Computed ratio falls within 1 percentage point of the stated ~5% claim (absolute discrepancy ≈0.25 pp)
- post_training_compute_ratio.csv contains rows for BOTH the R1 RL-training stage and the V3 pretraining stage
- CSV schema includes columns for training stage, GPU hours, and share of pretraining compute
- Reported relative discrepancy is explicitly stated alongside the raw ratio

## Limitations

- The raw GPU-hour ratio does not normalize for GPU type (H800 vs. pretraining fleet) or utilization efficiency, so it is a bookkeeping reconciliation, not a physical compute-cost estimate
- Reported figures are self-disclosed by the developer in technical reports and cannot be independently verified from the article context
- No released training details exist for some post-training stages (e.g., instruction tuning during rejection sampling), so the ratio only covers the RL stage vs. pretraining, not the full post-training pipeline

## Evidence

- [methods] compute ratio arithmetic grounded in NumPy usage: "import numpy as np
x = np.random.randint(10, size = 10)"
- [results] the reconciliation finding: 147K/2.8M ≈ 5.3% vs. stated ~5%: [evidence span withheld — anchor status: not_found]
- [readme] skill scope: comprehensive post-training guide whose compute claims are being reconciled: "A comprehensive guide to Reinforcement Learning from Human Feedback (and a broad introduction to post-training language models)."
