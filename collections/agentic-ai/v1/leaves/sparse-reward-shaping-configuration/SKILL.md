---
name: sparse-reward-shaping-configuration
description: Use when training RL agents on sparse-reward gridworld tasks (e.g., MiniGrid-UnlockPickup)
  where you need shaped intermediate rewards (key pickup, door opening) and want to
  compare guided vs. unguided baselines by reward AUC and steps-to-goal.
when_to_use_negative:
- Your task is not a sparse-reward gridworld benchmark — dense-reward settings do
  not need LLM-guided shaping or the shaped key/door/goal bonus schedule
- You cannot supply or train the fine-tuned BERT action predictor; Unguided PPO or
  Q-Learning baselines alone are a better fit
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ULPS
  role: Calibrated LLM + PPO framework that shapes the policy under sparse rewards
    via uncertainty-aware LLM guidance
  repo: USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: Proximal Policy Optimization (PPO)
  role: Base RL algorithm whose learned policy is blended with LLM guidance by entropy
    weighting
- name: BERT (bert-base-uncased)
  role: Language model fine-tuned on A*-based oracle trajectories to act as the action
    predictor
- name: MiniGrid
  role: Benchmark environment (MiniGrid-UnlockPickup) on which the reward shaping
    and baselines are evaluated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:661ef1b012a9d8766325c9e137893ea3719bca4d58d28124ac5b7b2092dabd9e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - BERT (bert-base-uncased)
  - MiniGrid
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# sparse-reward-shaping-configuration

## Summary

Configure sparse-reward shaping and LLM-guided policy blending (ULPS) for the MiniGrid-UnlockPickup benchmark, combining a fine-tuned BERT action predictor with Monte Carlo dropout uncertainty and entropy-based blending into PPO, then benchmark against standard RL baselines via reward AUC.

## When to use

Use when training RL agents on sparse-reward gridworld tasks (e.g., MiniGrid-UnlockPickup) where you need shaped intermediate rewards (key pickup, door opening) and want to compare guided vs. unguided baselines by reward AUC and steps-to-goal.

## When NOT to use

- Your task is not a sparse-reward gridworld benchmark — dense-reward settings do not need LLM-guided shaping or the shaped key/door/goal bonus schedule
- You cannot supply or train the fine-tuned BERT action predictor; Unguided PPO or Q-Learning baselines alone are a better fit

## Inputs

- MiniGrid-UnlockPickup environment at 4x4 and 8x8 grid sizes
- Sparse reward function (+0.5 key pickup, +0.5 door opening, +0.2 goal reach, -0.02 invalid action penalty, step-based bonus)
- Fine-tuned BERT (bert-base-uncased) action predictor trained on A*-based oracle symbolic trajectories
- Baseline agent configurations: Unguided PPO, Linear Decay, Q-Learning, DQN

## Outputs

- episode_metrics_*.csv episode-level logs
- entropy_values_*.csv and calibration_metrics_*.csv logs
- experiment_auc_summary.csv with reward AUC per method
- final_training_summary_*.png figures (average-reward trajectory and radar comparison)

## How to apply

Launch the ULPS experiment runners (MainCode/run_4x4_experiment.py and run_8x8_experiment.py) with the Calibrated LLM + PPO agent on MiniGrid-UnlockPickup using the sparse reward function: +0.5 key pickup, +0.5 door opening, +0.2 goal reach, -0.02 invalid action penalty, plus a step-based bonus. Train the Unguided PPO, Linear Decay, Q-Learning, and DQN baselines on the same environments and reward function for comparability. The ULPS pipeline fine-tunes bert-base-uncased on A*-based oracle symbolic trajectories, estimates epistemic uncertainty via Monte Carlo dropout, and blends LLM guidance with the learned PPO policy using an entropy-based mechanism. During training, write episode_metrics_*.csv, entropy_values_*.csv, and calibration_metrics_*.csv logs, then compute reward AUC per method and consolidate into experiment_auc_summary.csv. Plot the average-reward trajectory (Fig. 2) and reward/success/steps/AUC radar comparison (Fig. 3) as final_training_summary_*.png to verify ULPS achieves higher reward AUC (2055.08 in the 4x4 setting) and lower average steps to goal (7.24) than the baselines.

## Related tools

- **ULPS** (Calibrated LLM + PPO framework that shapes the policy under sparse rewards via uncertainty-aware LLM guidance) — USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **Proximal Policy Optimization (PPO)** (Base RL algorithm whose learned policy is blended with LLM guidance by entropy weighting)
- **BERT (bert-base-uncased)** (Language model fine-tuned on A*-based oracle trajectories to act as the action predictor)
- **MiniGrid** (Benchmark environment (MiniGrid-UnlockPickup) on which the reward shaping and baselines are evaluated)

## Examples

```
python MainCode/run_4x4_experiment.py
```

## Evaluation signals

- experiment_auc_summary.csv reproduces the qualitative result that ULPS achieves higher reward AUC than Unguided PPO, Linear Decay, Q-Learning, and DQN
- Calibrated model attains the reported 4x4 values: reward AUC of 2055.08 and average steps to goal of 7.24
- episode_metrics_*.csv show ULPS requires fewer environment interactions and higher success rate than baselines
- final_training_summary_*.png figures reproduce Fig. 2 (average-reward trajectory) and Fig. 3 (radar comparison) qualitatively

## Limitations

- Not all result files are regenerated automatically by a single script; some CSV files and plots were pre-generated during experimentation and shipped under results/
- Some plotting and summarization scripts assume the same output folder structure used during the experiments, and paths may require minor adjustments locally
- Only a subset of development plots was selected for the final paper, so the repository contains additional plots beyond the reported figures
- No changelog found in the repository

## Evidence
