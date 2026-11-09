---
name: gridworld-episode-metric-logging
description: Use when running or reproducing MiniGrid-UnlockPickup RL experiments
  (e.g., ULPS dropout/forward-pass sweeps) and you need per-episode CSV logs plus
  reward-AUC summaries per configuration.
when_to_use_negative:
- The task targets baseline implementations (Qlearning/, DQN/, Unguided PPO) rather
  than the calibrated LLM+PPO ULPS pipeline with its dropout/forward-pass ablation
  axes
- Only a qualitative training-curve comparison is needed and no per-configuration
  average steps-to-goal or reward-AUC consolidation is required
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ULPS
  role: Main implementation (Calibrated LLM + PPO) whose experiment runners generate
    the episode-level CSV logs and summaries
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: Proximal Policy Optimization (PPO)
  role: The learned RL policy executed during the logging runs; episode reward curves
    used for AUC come from PPO training
- name: MiniGrid
  role: UnlockPickup benchmark environment whose episodes produce the steps-to-goal
    and reward metrics being logged
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:f6ff7757efef0c566c150f774fb942b101e41bbaa5a0604ab5e6daeeb01bf595
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gridworld-episode-metric-logging

## Summary

Generates episode-level CSV metric logs (episode_metrics_*.csv, entropy_values_*.csv) from ULPS training runs in the MiniGrid-UnlockPickup gridworld, then consolidates per-configuration average steps to goal and reward AUC into experiment_auc_summary.csv for ablation analysis.

## When to use

Use when running or reproducing MiniGrid-UnlockPickup RL experiments (e.g., ULPS dropout/forward-pass sweeps) and you need per-episode CSV logs plus reward-AUC summaries per configuration.

## When NOT to use

- The task targets baseline implementations (Qlearning/, DQN/, Unguided PPO) rather than the calibrated LLM+PPO ULPS pipeline with its dropout/forward-pass ablation axes
- Only a qualitative training-curve comparison is needed and no per-configuration average steps-to-goal or reward-AUC consolidation is required

## Inputs

- ULPS training run in MiniGrid-UnlockPickup 4x4 (or 8x8) environment via MainCode/run_4x4_experiment.py
- Per-configuration episode logs: episode_metrics_*.csv
- Per-configuration entropy logs: entropy_values_*.csv
- Ablation configurations: dropout rate in {0.05, 0.1, 0.2} x MC forward passes in {4, 8, 12}

## Outputs

- episode_metrics_*.csv (episode-level training logs)
- entropy_values_*.csv (per-episode entropy logs)
- experiment_auc_summary.csv (rows of dropout rate, forward passes, average steps to goal, reward AUC)
- consolidated plots such as final_training_summary_*.png

## How to apply

Launch the ULPS experiment via MainCode/run_4x4_experiment.py (or run_8x8_experiment.py), which trains the calibrated LLM-guided PPO agent with Monte Carlo dropout uncertainty estimation and writes episode-level CSV logs. For ablations, sweep the dropout-rate axis (0.05, 0.1, 0.2) using scripts in the 'Ablation Study/' folder and cross it with the MC forward-pass axis (4, 8, 12) to yield 9 configurations. Parse the episode_metrics_*.csv and entropy_values_*.csv logs produced by each run to compute average steps to goal per configuration, and integrate the per-episode reward curves over training to compute the reward AUC. Consolidate the (dropout, forward-passes) rows of average steps to goal and reward AUC into experiment_auc_summary.csv, mirroring the structure of the pre-generated files under results/.

## Related tools

- **ULPS** (Main implementation (Calibrated LLM + PPO) whose experiment runners generate the episode-level CSV logs and summaries) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **Proximal Policy Optimization (PPO)** (The learned RL policy executed during the logging runs; episode reward curves used for AUC come from PPO training)
- **MiniGrid** (UnlockPickup benchmark environment whose episodes produce the steps-to-goal and reward metrics being logged)

## Examples

```
python MainCode/run_4x4_experiment.py
```

## Evaluation signals

- experiment_auc_summary.csv contains exactly 9 configuration rows when the full 3x3 dropout/forward-pass sweep is run
- All rows in experiment_auc_summary.csv have non-null average-steps-to-goal and reward-AUC values
- Reported metrics are consistent with the pre-generated CSV logs under results/ (e.g., dropout 0.1 / 8 passes ≈ 2055.08 AUC; dropout 0.05 / 4 passes ≈ 599.46 AUC, 50 steps)
- Each configuration produces paired episode_metrics_*.csv and entropy_values_*.csv logs following the expected output folder structure

## Limitations

- Not all result files are regenerated automatically by a single script; some CSVs and plots were pre-generated during experimentation and shipped under results/
- Plotting and summarization scripts assume the same output folder structure used during the original experiments and may require minor path adjustments locally
- Only a subset of development plots was selected for the final paper, so results/ may not cover every generated artifact

## Evidence

- [methods] workflow_parse: [evidence span withheld — anchor status: not_found]
- [methods] workflow_auc: [evidence span withheld — anchor status: not_found]
- [methods] workflow_validation: [evidence span withheld — anchor status: not_found]
