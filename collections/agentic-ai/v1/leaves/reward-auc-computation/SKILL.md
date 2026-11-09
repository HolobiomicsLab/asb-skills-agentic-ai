---
name: reward-auc-computation
description: Use when comparing RL training runs (ULPS, Unguided PPO, Linear Decay,
  Q-Learning, DQN, or dropout/forward-pass ablations) on MiniGrid-UnlockPickup and
  you need a single scalar, the reward AUC, summarizing the average-reward trajectory
  over episodes.
when_to_use_negative:
- You need per-episode reward dynamics or learning-curve shape rather than a single
  integrated scalar — read episode_metrics_*.csv directly instead.
- Evaluating a model on a single evaluation episode or final reward only — AUC summarizes
  the whole training trajectory and will not reflect end-state performance alone.
- Comparing runs with different episode counts or reward functions — the sparse reward
  scheme (+0.5/+0.5/+0.2/-0.02 plus step bonus) must be identical for AUC comparability.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ULPS
  role: Calibrated LLM + PPO agent whose training run generates the reward curves
    that are integrated into the reward AUC
- name: MiniGrid
  role: MiniGrid-UnlockPickup benchmark environment (4x4 and 8x8) in which the reward
    function and AUC comparison are evaluated
- name: Proximal Policy Optimization (PPO)
  role: Learned-policy component trained on the sparse reward whose episode-level
    rewards feed the AUC computation
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:eb12ae5a8059f3ef960bab33fcc0046a966aa4620379385fc1790c3c24eb8fd9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# reward-auc-computation

## Summary

Compute the reward AUC by integrating per-episode reward curves over training (from episode_metrics_*.csv logs) to benchmark RL agents in the MiniGrid-UnlockPickup ULPS experiments, where the calibrated LLM+PPO model reaches AUC 2055.08 versus baselines.

## When to use

Use when comparing RL training runs (ULPS, Unguided PPO, Linear Decay, Q-Learning, DQN, or dropout/forward-pass ablations) on MiniGrid-UnlockPickup and you need a single scalar, the reward AUC, summarizing the average-reward trajectory over episodes.

## When NOT to use

- You need per-episode reward dynamics or learning-curve shape rather than a single integrated scalar — read episode_metrics_*.csv directly instead.
- Evaluating a model on a single evaluation episode or final reward only — AUC summarizes the whole training trajectory and will not reflect end-state performance alone.
- Comparing runs with different episode counts or reward functions — the sparse reward scheme (+0.5/+0.5/+0.2/-0.02 plus step bonus) must be identical for AUC comparability.

## Inputs

- episode_metrics_*.csv (episode-level reward/steps logs produced by run_4x4_experiment.py or run_8x8_experiment.py)
- entropy_values_*.csv (entropy logs used in ablation metric extraction)
- dropout-rate x forward-pass configuration sweep results (0.05/0.1/0.2 x 4/8/12)

## Outputs

- experiment_auc_summary.csv (consolidated reward AUC and average steps to goal per method/configuration)
- Per-method reward AUC scalars feeding the Fig. 2 average-reward trajectory and Fig. 3 radar comparison (final_training_summary_*.png)

## How to apply

Parse the episode-level logs (episode_metrics_*.csv, optionally entropy_values_*.csv) produced by MainCode/run_4x4_experiment.py or run_8x8_experiment.py during training with the sparse reward function (+0.5 key pickup, +0.5 door opening, +0.2 goal reach, -0.02 invalid action penalty, plus a step-based bonus). Integrate each per-episode reward curve over training episodes to obtain the reward AUC for every method or ablation configuration. Consolidate the rows (e.g., the 9 dropout-rate x MC forward-pass configurations: 0.05/0.1/0.2 crossed with 4/8/12 passes) into experiment_auc_summary.csv alongside average steps to goal. The rationale is that a higher AUC plus fewer steps to goal indicates a more sample-efficient, stable training trajectory, which is the paper's headline comparison metric (calibrated model: 2055.08 AUC, 7.24 steps).

## Related tools

- **ULPS** (Calibrated LLM + PPO agent whose training run generates the reward curves that are integrated into the reward AUC)
- **MiniGrid** (MiniGrid-UnlockPickup benchmark environment (4x4 and 8x8) in which the reward function and AUC comparison are evaluated)
- **Proximal Policy Optimization (PPO)** (Learned-policy component trained on the sparse reward whose episode-level rewards feed the AUC computation)

## Examples

```
python MainCode/run_4x4_experiment.py  # trains ULPS and baselines, writes episode_metrics_*.csv for reward-AUC integration
```

## Evaluation signals

- experiment_auc_summary.csv contains exactly 9 non-null (average-steps-to-goal, reward-AUC) rows for the dropout/forward-pass ablation (0.05/0.1/0.2 x 4/8/12)
- Calibrated ULPS run reproduces reward AUC ≈ 2055.08 (4x4, dropout 0.1, 8 passes) and the ablation top AUC ≈ 2055.88 (dropout 0.2, 12 passes)
- Poor configuration (dropout 0.05, 4 passes) should land near 599.46 AUC and ~50 steps to goal — a strong outlier check
- AUC ordering is consistent with the qualitative result: ULPS > Unguided PPO, Linear Decay, Q-Learning, DQN, with ULPS also lowest at 7.24 average steps to goal
- Values are consistent with the pre-generated CSV logs and final_training_summary_*.png under results/

## Limitations

- Not all result files are regenerated automatically by a single script; AUC consolidation must be done from logged CSVs
- Some results (CSV files and plots) were pre-generated during experimentation and shipped under results/ rather than recomputed
- Plotting and summarization scripts assume the same output folder structure used during the experiments, and may require minor path adjustments locally
- No changelog is available, so version drift in the AUC computation cannot be tracked

## Evidence

- [methods] Parse the episode_metrics_*.csv and entropy_values_*.csv logs produced by each run to compute average steps to goal per configuration.: [evidence span withheld — anchor status: not_found]
- [methods] Integrate the per-episode reward curves over training to compute the reward AUC for each configuration.: [evidence span withheld — anchor status: not_found]
- [methods] verify that experiment_auc_summary.csv contains exactly 9 configuration rows with non-null average-steps-to-goal and reward-AUC values: [evidence span withheld — anchor status: not_found]
