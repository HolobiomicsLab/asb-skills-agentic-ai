---
name: pretraining-posttraining-compute-ratio-reconciliation
description: Use when two reported training-compute figures (e.g., GPU hours for post-training
  RL vs base-model pretraining) must be reconciled against a stated percentage claim,
  or when auditing whether a 'compute share' statement in a paper or book is arithmetically
  consistent.
when_to_use_negative:
- The reported compute figures are missing, inconsistent in hardware type (e.g., H800
  vs A100 mixed without normalization), or refer to different pipeline stages than
  post-training RL vs pretraining — the ratio is then not comparable
- The source states a compute share without any underlying raw figures, so no arithmetic
  reconciliation is possible
- The analysis goal is cost or energy estimation (dollars, kWh) rather than a GPU-hour
  ratio reconciliation
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Python
  role: Scripting language used to load figures, compute the ratio, and write the
    CSV results table
- name: NumPy
  role: Numerical library used to hold the GPU-hour figures in an array and compute
    the post-training compute ratio
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:148f98a5437924cc3b4b97577223e222bb15c67e029b81d2d0adedcb80111e75
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pretraining-posttraining-compute-ratio-reconciliation

## Summary

Reconciles reported GPU-hour figures for a post-training RL stage against pretraining compute by computing a simple ratio (e.g., 147K H800 GPU hours for DeepSeek R1 RL training vs 2.8M GPU hours for DeepSeek V3 pretraining ≈ 5.3%) and checking it against a stated share (~5%) within a tolerance band.

## When to use

Use when two reported training-compute figures (e.g., GPU hours for post-training RL vs base-model pretraining) must be reconciled against a stated percentage claim, or when auditing whether a 'compute share' statement in a paper or book is arithmetically consistent.

## When NOT to use

- The reported compute figures are missing, inconsistent in hardware type (e.g., H800 vs A100 mixed without normalization), or refer to different pipeline stages than post-training RL vs pretraining — the ratio is then not comparable
- The source states a compute share without any underlying raw figures, so no arithmetic reconciliation is possible
- The analysis goal is cost or energy estimation (dollars, kWh) rather than a GPU-hour ratio reconciliation

## Inputs

- Reported GPU-hour counts per training stage (e.g., 147,000 H800 GPU hours for R1 RL post-training; 2,800,000 GPU hours for V3 pretraining)
- Stated compute-share claim from the source text (e.g., '~5% of overall compute in post-training')
- Tolerance band for acceptance (e.g., ±1 percentage point)

## Outputs

- Computed post-training/pretraining compute ratio (≈0.0525, i.e., 5.25%)
- Absolute and relative discrepancy versus the stated claim (≈0.25 percentage points)
- CSV results table (e.g., post_training_compute_ratio.csv) with columns: training stage, GPU hours, share of pretraining compute

## How to apply

Load the reported GPU-hour figures into a NumPy array in Python — here 147,000 H800 GPU hours for the DeepSeek R1 RL (post-training) stage and 2,800,000 GPU hours for DeepSeek V3 pretraining. Compute the ratio 147000/2800000 ≈ 0.0525, i.e., the RL stage used ~5.25% of the pretraining compute. Compare the computed ratio to the stated claim ('only about 5% of their overall compute in post-training') and report both absolute (≈0.25 percentage points) and relative discrepancy. Write a results table with columns for training stage, GPU hours, and share of pretraining compute (e.g., post_training_compute_ratio.csv). Apply the acceptance rule: the computed ratio must round to the stated claim within 1 percentage point, and the output CSV must contain rows for both stages.

## Related tools

- **Python** (Scripting language used to load figures, compute the ratio, and write the CSV results table)
- **NumPy** (Numerical library used to hold the GPU-hour figures in an array and compute the post-training compute ratio)

## Examples

```
import numpy as np; hrs = np.array([147000.0, 2800000.0]); ratio = hrs[0]/hrs[1]; print(f"RL/pretraining compute share: {ratio:.4f} ({ratio*100:.2f}%)")
```

## Evaluation signals

- Computed ratio 147000/2800000 ≈ 0.0525 (5.25%) reproduces exactly from the two input GPU-hour figures
- Computed ratio rounds to the stated ~5% claim within the 1-percentage-point tolerance (observed discrepancy ≈0.25 points)
- Output CSV (post_training_compute_ratio.csv) contains rows for both the R1 RL-training stage and the V3 pretraining stage
- CSV column schema matches: training stage, GPU hours, share of pretraining compute
- Reported absolute discrepancy (≈0.25 percentage points) is consistent with the ratio and the stated claim

## Limitations

- The ratio uses reported GPU-hour figures; differences in GPU type (H800 vs other accelerators), utilization, or unreported inference/rollout compute during RL are not captured
- The stated '~5%' claim refers to 'overall compute in post-training' while the ratio denominator is V3 pretraining compute — scope mismatch may explain the residual ≈0.25 percentage-point gap
- Reported figures cannot be independently verified from the provided context; the reconciliation only checks internal arithmetic consistency

## Evidence
