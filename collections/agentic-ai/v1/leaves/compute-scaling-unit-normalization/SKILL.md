---
name: compute-scaling-unit-normalization
description: Use when raw compute counts (GPU-hours, wall-clock, or cost figures)
  are reported for different training stages in incompatible magnitudes and a stated
  percentage or scaling claim must be checked, e.g. 147K/2.8M ≈ 5.3% vs a stated ~5%.
when_to_use_negative:
- The figures are already normalized (percentages or shares) — re-normalizing would
  double-transform and cannot validate the original claim
- The numerator and denominator come from different accounting boundaries (e.g. RL
  training GPU-hours vs total pipeline cost including inference), so the ratio is
  not meaningful
- The analysis goal is qualitative (what RLHF offers models) rather than quantitative
  reconciliation of a numeric claim
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3417
tools:
- name: NumPy
  role: Array container for the stage-level GPU-hour figures and the arithmetic needed
    to compute the ratio (147000/2800000 ≈ 0.0525)
  repo: https://github.com/natolambert/rlhf-book
- name: Python
  role: 'Driver for the reconciliation workflow: computing the ratio, checking the
    1-percentage-point acceptance tolerance, and writing the CSV results table'
  repo: https://github.com/natolambert/rlhf-book
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:6fc82150d568f051f516b7d0e327167b395137b2efc264ee38a5ed6b0b52d54c
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/natolambert/rlhf-book
  tools:
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3417
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# compute-scaling-unit-normalization

## Summary

Normalize reported training-compute figures (GPU-hours per training stage) into unit-free shares or ratios so that claims like 'post-training used only ~5% of overall compute' can be verified arithmetically. Motivated by reconciling DeepSeek R1's 147K H800 GPU-hour RL stage against DeepSeek V3's 2.8M GPU-hour pretraining run.

## When to use

Use when raw compute counts (GPU-hours, wall-clock, or cost figures) are reported for different training stages in incompatible magnitudes and a stated percentage or scaling claim must be checked, e.g. 147K/2.8M ≈ 5.3% vs a stated ~5%.

## When NOT to use

- The figures are already normalized (percentages or shares) — re-normalizing would double-transform and cannot validate the original claim
- The numerator and denominator come from different accounting boundaries (e.g. RL training GPU-hours vs total pipeline cost including inference), so the ratio is not meaningful
- The analysis goal is qualitative (what RLHF offers models) rather than quantitative reconciliation of a numeric claim

## Inputs

- Reported GPU-hour (or compute-cost) figures per training stage, e.g. 147000 H800 GPU-hours for R1 RL and 2800000 GPU-hours for V3 pretraining
- The stated share claim to reconcile (e.g. 'only about 5% of their overall compute in post-training')

## Outputs

- Computed stage-compute ratio (float, e.g. 0.0525) plus absolute and relative discrepancy versus the stated claim
- CSV results table with columns: training stage, GPU hours, share of pretraining compute (one row per stage, e.g. R1 RL-training and V3 pretraining)

## How to apply

Load the reported figures for each training stage (here: 147,000 H800 GPU-hours for R1 RL post-training and 2,800,000 GPU-hours for V3 pretraining) into a NumPy array. Compute the ratio of the post-training figure to the pretraining figure (147000/2800000 ≈ 0.0525, i.e. 5.25%). Compare the computed ratio to the published claim using an acceptance tolerance of 1 percentage point, and report both the absolute discrepancy (≈0.25 percentage points here) and the relative discrepancy. Emit a results table with columns for training stage, GPU hours, and share of pretraining compute, written to CSV (post_training_compute_ratio.csv), with one row per stage so the normalization is auditable. The rationale is that a stage-level share only reconciles when both numerator and denominator refer to the same hardware class (H800) and accounting boundary (training compute only, excluding inference or data generation).

## Related tools

- **NumPy** (Array container for the stage-level GPU-hour figures and the arithmetic needed to compute the ratio (147000/2800000 ≈ 0.0525)) — https://github.com/natolambert/rlhf-book
- **Python** (Driver for the reconciliation workflow: computing the ratio, checking the 1-percentage-point acceptance tolerance, and writing the CSV results table) — https://github.com/natolambert/rlhf-book

## Examples

```
import numpy as np; hours = np.array([147000., 2800000.]); ratio = hours[0]/hours[1]; print(f'{ratio:.4f}')
```

## Evaluation signals

- Computed ratio rounds to the stated ~5% claim within 1 percentage point (147000/2800000 ≈ 0.0525 → accept)
- The output CSV (post_training_compute_ratio.csv) contains rows for both the R1 RL-training stage and the V3 pretraining stage
- The absolute discrepancy versus the stated claim (≈0.25 percentage points) and the relative discrepancy are both explicitly reported
- Ratio is dimensionless and bounded in (0,1) for a part-of-whole share; any value outside this range signals a unit or accounting error
- Both figures in the numerator/denominator refer to the same hardware class (H800) before the ratio is formed

## Limitations

- The GPU-hour figures are reported (self-published) numbers, not independently measured; the ratio only validates internal consistency of the claim
- GPU-hours are not a hardware-normalized unit: the ratio assumes both stages use the same accelerator class (H800); cross-hardware comparisons would need FLOP accounting
- Software version numbers for the analysis tooling (e.g. NumPy/Python) are not stated, affecting reproducibility of the validation script

## Evidence

- [methods] NumPy used for array-based arithmetic in the book's methods examples: "import numpy as np
x = np.random.randint(10, size = 10)"
- [intro] Multi-stage post-training framing justifies per-stage compute accounting: "Post-training can be summarized as a many-stage training process using three optimization methods:"
- [readme] Book's hands-on code examples rationale for a reusable quantitative workflow: "This book is my attempt to open-source all the knowledge I've gained working at the frontier of open models in the post-ChatGPT takeoff of language models."
