---
name: grid-size-scaling-analysis
description: Decision support for whether LLM-guided policy shaping reduces environment
  interactions, and whether step counts scale linearly when the MiniGrid grid size
  doubles (4x4 vs 8x8).
when_to_use_negative:
- Inputs are already aggregated summary CSVs (e.g. `experiment_auc_summary.csv`) rather
  than per-episode logs, so step totals and episode averages cannot be recomputed
  independently
- The analysis goal is reward AUC or success-rate comparison rather than environment-step/sample-efficiency
  and grid-size scaling
- The environment is not MiniGrid-UnlockPickup (or grid sizes differ), so the 4x4/8x8
  linear-scaling comparison does not apply
tools:
- name: ULPS
  role: Guided (LLM+PPO) training run whose episode step counts are compared against
    the unguided baseline
  repo: USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: Proximal Policy Optimization (PPO)
  role: Unguided PPO baseline training run providing the reference total environment-step
    count (49,147)
  repo: USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: MiniGrid
  role: MiniGrid-UnlockPickup benchmark in 4x4 and 8x8 configurations whose grid-size
    doubling grounds the linear-scaling check
- name: pandas
  role: Used to load and aggregate episode steps from `episode_metrics_*.csv`
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:baf2b2bead8877a409368eba7a32393fabf5a57831bdb1b2bb10e2ef5c3d2603
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# grid-size-scaling-analysis

## Summary

Quantifies sample-efficiency and scaling behavior of ULPS versus unguided PPO on MiniGrid-UnlockPickup by aggregating per-episode environment-step counts from episode-level CSV logs and checking how average steps per episode grow with grid size (4x4 to 8x8).

## When to use

Decision support for whether LLM-guided policy shaping reduces environment interactions, and whether step counts scale linearly when the MiniGrid grid size doubles (4x4 vs 8x8).

## When NOT to use

- Inputs are already aggregated summary CSVs (e.g. `experiment_auc_summary.csv`) rather than per-episode logs, so step totals and episode averages cannot be recomputed independently
- The analysis goal is reward AUC or success-rate comparison rather than environment-step/sample-efficiency and grid-size scaling
- The environment is not MiniGrid-UnlockPickup (or grid sizes differ), so the 4x4/8x8 linear-scaling comparison does not apply

## Inputs

- Episode-level CSV logs (`episode_metrics_*.csv`) from `results/` for ULPS 4x4 and 8x8 MiniGrid-UnlockPickup runs
- Episode-level CSV logs for the unguided PPO baseline
- Grid-size configuration identifiers (4x4 vs 8x8) and per-episode step counts

## Outputs

- Sample-efficiency comparison table (e.g. `sample_efficiency_summary.csv`) with total environment steps, reduction percentage (~86%), and average steps per episode per grid size
- Computed scaling ratio of average steps per episode between grid sizes (15.37/7.24 ≈ 2.12)

## How to apply

Load the per-episode step counts from the episode-level logs (`episode_metrics_*.csv`) under `results/` for the ULPS runs (4x4 and 8x8 MiniGrid-UnlockPickup) and the unguided PPO baseline using pandas. Aggregate episode steps per configuration to obtain total environment interactions (expected: ULPS 7,286 vs unguided baseline 49,147 in the 4x4 setting) and compute the relative reduction (49,147 - 7,286) / 49,147 ≈ 86%. Separately compute average steps per episode per grid size (7.24 for 4x4, 15.37 for 8x8) and evaluate the ratio (15.37/7.24 ≈ 2.12) against the doubling of grid size to confirm approximately linear scaling while the grid area quadruples. Assemble totals, reduction percentage, and per-grid average steps into a derived sample-efficiency summary table saved as CSV (e.g. `sample_efficiency_summary.csv`) alongside the consolidated summaries in `results/`.

## Related tools

- **ULPS** (Guided (LLM+PPO) training run whose episode step counts are compared against the unguided baseline) — USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **Proximal Policy Optimization (PPO)** (Unguided PPO baseline training run providing the reference total environment-step count (49,147)) — USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **MiniGrid** (MiniGrid-UnlockPickup benchmark in 4x4 and 8x8 configurations whose grid-size doubling grounds the linear-scaling check)
- **pandas** (Used to load and aggregate episode steps from `episode_metrics_*.csv`)

## Examples

```
import pandas as pd, glob; df = pd.concat([pd.read_csv(f) for f in glob.glob('results/episode_metrics_*.csv')]); totals = df.groupby('config')['steps'].sum(); reduction = 1 - totals['ulps']/totals['unguided']
```

## Evaluation signals

- Total steps for ULPS in the 4x4 setting reproduce the reported 7,286, versus 49,147 for unguided RL
- Computed relative reduction in environment interactions reproduces the reported ~86% ((49,147 - 7,286) / 49,147)
- Average steps per episode match 7.24 (4x4) and 15.37 (8x8), with ratio 15.37/7.24 ≈ 2.12, consistent with linear scaling under a 2x grid-size increase
- Output CSV (e.g. `sample_efficiency_summary.csv`) contains totals, reduction percentage, and per-grid average steps for every configuration present in the input logs

## Limitations

- Not all result files are regenerated automatically by a single script; some CSVs and plots were pre-generated during experimentation and are only included under `results/`
- Plotting and summarization scripts assume the same output folder structure used during the experiments and may require minor path adjustments locally
- Scaling conclusion rests on only two grid sizes (4x4 and 8x8) from a single benchmark (MiniGrid-UnlockPickup)

## Evidence

- [results] ULPS requires only 7,286 total environment steps versus 49,147 for unguided RL, an 86% reduction in environment interactions.: [evidence span withheld — anchor status: not_found]
