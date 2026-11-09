---
name: episode-log-aggregation
description: Use when comparing sample efficiency of RL variants from episode-level
  CSV logs, and you need total environment steps, per-episode averages, or a relative
  interaction-reduction percentage.
when_to_use_negative:
- The goal is reward-AUC or success-rate comparison rather than sample complexity
  — use the consolidated summaries (`experiment_auc_summary.csv`) instead
- Logs are calibration or entropy files (`calibration_metrics_*.csv`, `entropy_values_*.csv`),
  which do not contain per-episode step counts
- Only pre-generated plots (`final_training_summary_*.png`) are available and the
  underlying episode CSVs are missing, since totals cannot be recomputed
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ULPS
  role: Guided-RL method whose run logs provide the episode step counts being aggregated
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: Proximal Policy Optimization (PPO)
  role: Unguided baseline learner whose episode logs (49,147 steps) are compared against
    ULPS (7,286 steps)
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: MiniGrid
  role: UnlockPickup benchmark environment defining the 4x4 and 8x8 grid configurations
    over which steps are aggregated and scaled
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:e7155c5c72bba445b6ef96604c35159ef9b603f3bb52c042f513910982f3fa1f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# episode-log-aggregation

## Summary

Aggregates per-episode step counts from `episode_metrics_*.csv` logs to quantify total environment interactions and sample-complexity differences between ULPS and unguided PPO in MiniGrid-UnlockPickup, e.g. the reported 86% step reduction (7,286 vs 49,147).

## When to use

Use when comparing sample efficiency of RL variants from episode-level CSV logs, and you need total environment steps, per-episode averages, or a relative interaction-reduction percentage.

## When NOT to use

- The goal is reward-AUC or success-rate comparison rather than sample complexity — use the consolidated summaries (`experiment_auc_summary.csv`) instead
- Logs are calibration or entropy files (`calibration_metrics_*.csv`, `entropy_values_*.csv`), which do not contain per-episode step counts
- Only pre-generated plots (`final_training_summary_*.png`) are available and the underlying episode CSVs are missing, since totals cannot be recomputed

## Inputs

- Episode-level step-count logs (`episode_metrics_*.csv`) in `results/` for ULPS 4x4 and 8x8 runs and the unguided PPO baseline
- Optionally `experiment_auc_summary.csv` for cross-checking against consolidated summaries

## Outputs

- Sample-efficiency comparison table (`sample_efficiency_summary.csv`) with total environment interactions, relative reduction percentage, and per-grid average steps per episode

## How to apply

Load the per-episode step counts from `episode_metrics_*.csv` files under `results/` for the ULPS runs (4x4 and 8x8 MiniGrid-UnlockPickup) and the unguided PPO baseline using pandas. Aggregate episode steps per configuration to obtain total environment interactions (ULPS 7,286 vs unguided baseline 49,147) and compute the relative reduction as (49,147 - 7,286) / 49,147, yielding the reported ~86% reduction. Also compute average steps per episode for the 4x4 grid (7.24) and the 8x8 grid (15.37); the ratio 15.37/7.24 ≈ 2.12, checked against the quadrupling of grid area, confirms approximately linear scaling. Assemble totals, reduction percentage, and per-grid average steps into a derived sample-efficiency comparison table saved as CSV (e.g. `sample_efficiency_summary.csv`) alongside consolidated summaries like `experiment_auc_summary.csv` in `results/`.

## Related tools

- **ULPS** (Guided-RL method whose run logs provide the episode step counts being aggregated) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **Proximal Policy Optimization (PPO)** (Unguided baseline learner whose episode logs (49,147 steps) are compared against ULPS (7,286 steps)) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **MiniGrid** (UnlockPickup benchmark environment defining the 4x4 and 8x8 grid configurations over which steps are aggregated and scaled) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl

## Examples

```
import pandas as pd; df = pd.read_csv('results/episode_metrics_ulps_4x4.csv'); total = df['steps'].sum(); reduction = 1 - total/49147
```

## Evaluation signals

- Computed reduction reproduces the reported ~86%: (49,147 - 7,286) / 49,147
- Total steps per configuration match 7,286 (ULPS) and 49,147 (unguided PPO) in the 4x4 setting
- Average-steps ratio 15.37 / 7.24 ≈ 2.12 is consistent with linear scaling despite the 4x grid-area increase
- Output CSV schema contains totals, reduction percentage, and per-grid average steps for both grid sizes

## Limitations

- Not all result files are regenerated automatically by a single script; some CSVs and plots were pre-generated during experimentation and are only included under `results/`
- Some plotting and summarization scripts assume the same output folder structure used during the experiments and may require minor path adjustments
- Average-steps-per-episode scaling was only assessed for 4x4 vs 8x8 MiniGrid-UnlockPickup; extrapolation to other grid sizes is unverified

## Evidence

- [methods] load episode logs: [evidence span withheld — anchor status: not_found]
- [methods] compute reduction: [evidence span withheld — anchor status: not_found]
- [methods] linear scaling check: [evidence span withheld — anchor status: not_found]
