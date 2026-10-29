---
name: gpu-hour-compute-accounting
description: Use when a source reports raw GPU-hour (or compute) totals for distinct
  training stages and you must compute the ratio between stages and reconcile it against
  a stated percentage claim.
when_to_use_negative:
- The source reports FLOPs, dollar costs, or accelerator types that are not comparable
  GPU-hours, so a raw GPU-hour ratio would conflate hardware efficiency with compute
  share
- The claim refers to overall compute including inference or data-generation costs
  not captured by the reported training-stage GPU-hours
tools:
- name: Python
  role: Compute the GPU-hour ratio and write the results CSV in the validation workflow
- name: NumPy
  role: Hold the reported GPU-hour figures in an array and compute the ratio
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:7c8b0ec47655239b0c7762182b7322cfdcda61dbccb2d040742f677ecadc77b8
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gpu-hour-compute-accounting

## Summary

A quantitative skill for reconciling reported GPU-hour figures across training stages (e.g., DeepSeek R1's 147K H800 GPU hours of RL post-training vs. DeepSeek V3's 2.8M GPU hours of pretraining) into a compute-share ratio, and validating it against a stated headline claim (~5% of overall compute in post-training).

## When to use

Use when a source reports raw GPU-hour (or compute) totals for distinct training stages and you must compute the ratio between stages and reconcile it against a stated percentage claim.

## When NOT to use

- The source reports FLOPs, dollar costs, or accelerator types that are not comparable GPU-hours, so a raw GPU-hour ratio would conflate hardware efficiency with compute share
- The claim refers to overall compute including inference or data-generation costs not captured by the reported training-stage GPU-hours

## Inputs

- Reported GPU-hour totals per training stage (e.g., 147,000 H800 GPU hours for R1 RL training; 2,800,000 GPU hours for V3 pretraining)
- A stated headline percentage claim from the source text (e.g., '~5% of overall compute in post-training')

## Outputs

- Computed compute-share ratio (e.g., 147000/2800000 ≈ 0.0525, ~5.25%) with absolute and relative discrepancy vs. the claim
- Results table as CSV (post_training_compute_ratio.csv) with columns: training stage, GPU hours, share of pretraining compute

## How to apply

Load the reported GPU-hour figures for each training stage into a NumPy array in Python (e.g., 147,000 H800 GPU hours for DeepSeek R1 RL training; 2,800,000 GPU hours for DeepSeek V3 pretraining). Compute the post-training compute ratio as numerator/denominator (147000/2800000 ≈ 0.0525, i.e., ~5.25% of pretraining compute). Compare the computed ratio against the stated claim (the book's 'only about 5% of their overall compute in post-training'), reporting the absolute and relative discrepancy (≈0.25 percentage points). Write a results table with columns for training stage, GPU hours, and share of pretraining compute to a CSV (post_training_compute_ratio.csv). Rationale: raw GPU-hour ratios are the only directly reconcilable quantities when sources report compute per stage, and the validation criterion is that the computed ratio rounds to the stated claim within 1 percentage point.

## Related tools

- **Python** (Compute the GPU-hour ratio and write the results CSV in the validation workflow)
- **NumPy** (Hold the reported GPU-hour figures in an array and compute the ratio)

## Examples

```
import numpy as np; hours = np.array([147000, 2800000]); ratio = hours[0]/hours[1]; print(f'{ratio:.4f} ({ratio*100:.2f}%)')
```

## Evaluation signals

- Computed ratio 147000/2800000 rounds to 0.0525 (≈5.25%), within 1 percentage point of the stated ~5% claim
- Output CSV contains rows for both the R1 RL-training (post-training) stage and the V3 pretraining stage
- CSV schema includes columns for training stage, GPU hours, and share of pretraining compute
- Reported absolute discrepancy between computed ratio and stated claim is ≈0.25 percentage points

## Limitations

- The ratio only reconciles if the reported GPU-hour figures cover comparable hardware (H800 here) and the same accounting scope; the book states no released training details for some stages, so denominators may be incomplete
- A raw GPU-hour ratio measures compute share relative to pretraining only, not the source's possibly broader 'overall compute' denominator

## Evidence

- [other] Task finding reconciling R1 RL vs V3 pretraining GPU hours: [evidence span withheld — anchor status: not_found]
- [supplementary] Task workflow validation criterion for the computed ratio: [evidence span withheld — anchor status: not_found]
- [readme] RLHF Book README describing the book's scope: "This book is my attempt to open-source all the knowledge I've gained working at the frontier of open models in the post-ChatGPT takeoff of language models."
