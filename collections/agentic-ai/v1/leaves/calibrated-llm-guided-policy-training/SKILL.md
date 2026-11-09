---
name: calibrated-llm-guided-policy-training
description: Use when training RL policies in sparse-reward environments such as MiniGrid-UnlockPickup
  and you want an uncertainty-calibrated LLM action predictor (fine-tuned BERT) to
  shape a PPO policy via entropy-based blending.
when_to_use_negative:
- The reward signal is dense or Unguided PPO/Q-Learning/DQN already solve the task
  — the method targets sparse-reward settings like MiniGrid-UnlockPickup.
- No A*-based oracle symbolic trajectories are available to fine-tune the BERT action
  predictor, removing the calibrated guidance component.
- You need a single script to regenerate every artifact under results/ — the README
  warns some CSVs and plots are pre-generated.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ULPS
  role: 'Framework under study: Calibrated LLM + PPO implementation and experiment
    runners (MainCode/run_4x4_experiment.py, run_8x8_experiment.py)'
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: BERT (bert-base-uncased)
  role: LLM action predictor fine-tuned on A*-based oracle symbolic trajectories;
    its guidance is entropy-blended with the PPO policy
- name: Proximal Policy Optimization (PPO)
  role: Learned policy shaped by the uncertainty-conditioned LLM guidance during training
- name: MiniGrid
  role: Benchmark environment (MiniGrid-UnlockPickup) supplying the sparse-reward
    task for training and evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:0fafc3b1a855c1676bddb7fea37d65257bf1362e5e975cbebe712b6ec7f76ee3
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
yes_rate: 0.8333333333333334
n_claims: 12
---

# calibrated-llm-guided-policy-training

## Summary

Trains RL policies with ULPS-style guidance: a fine-tuned BERT action predictor, Monte Carlo dropout uncertainty, and entropy-based blending shape a PPO policy in sparse-reward MiniGrid-UnlockPickup.

## When to use

Use when training RL policies in sparse-reward environments such as MiniGrid-UnlockPickup and you want an uncertainty-calibrated LLM action predictor (fine-tuned BERT) to shape a PPO policy via entropy-based blending.

## When NOT to use

- The reward signal is dense or Unguided PPO/Q-Learning/DQN already solve the task — the method targets sparse-reward settings like MiniGrid-UnlockPickup.
- No A*-based oracle symbolic trajectories are available to fine-tune the BERT action predictor, removing the calibrated guidance component.
- You need a single script to regenerate every artifact under results/ — the README warns some CSVs and plots are pre-generated.

## Inputs

- MiniGrid-UnlockPickup environment instance (4x4 or 8x8 grid)
- Sparse reward specification: +0.5 key pickup, +0.5 door opening, +0.2 goal reach, -0.02 invalid action penalty, plus a step-based bonus
- A*-based oracle symbolic trajectories for fine-tuning bert-base-uncased
- Baseline training configurations: Unguided PPO, Linear Decay, Q-Learning, DQN

## Outputs

- Trained Calibrated LLM + PPO (ULPS) policy
- Episode-level logs: episode_metrics_*.csv, entropy_values_*.csv, calibration_metrics_*.csv
- experiment_auc_summary.csv consolidating reward AUC per method
- final_training_summary_*.png figures (average-reward trajectory and radar comparison)

## How to apply

First fine-tune a BERT-based language model (bert-base-uncased) as an action predictor on optimal symbolic trajectories synthesized by an A*-based oracle. During training, estimate the predictor's epistemic uncertainty with Monte Carlo (MC) dropout and let an entropy-based blending mechanism adaptively balance LLM guidance against the PPO learned policy. Run the ULPS experiment runners (MainCode/run_4x4_experiment.py or run_8x8_experiment.py) on MiniGrid-UnlockPickup with the sparse reward function (+0.5 key pickup, +0.5 door opening, +0.2 goal reach, -0.02 invalid action penalty, plus a step-based bonus). Write episode-level logs to episode_metrics_*.csv, entropy_values_*.csv, and calibration_metrics_*.csv, then compute reward AUC per method into experiment_auc_summary.csv. Train the Unguided PPO, Linear Decay, Q-Learning, and DQN baselines on the same environments and reward function, and plot final_training_summary_*.png (Fig. 2 average-reward trajectory; Fig. 3 radar comparison).

## Related tools

- **ULPS** (Framework under study: Calibrated LLM + PPO implementation and experiment runners (MainCode/run_4x4_experiment.py, run_8x8_experiment.py)) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **BERT (bert-base-uncased)** (LLM action predictor fine-tuned on A*-based oracle symbolic trajectories; its guidance is entropy-blended with the PPO policy)
- **Proximal Policy Optimization (PPO)** (Learned policy shaped by the uncertainty-conditioned LLM guidance during training)
- **MiniGrid** (Benchmark environment (MiniGrid-UnlockPickup) supplying the sparse-reward task for training and evaluation)

## Examples

```
python MainCode/run_4x4_experiment.py
```

## Evaluation signals

- experiment_auc_summary.csv shows the Calibrated LLM + PPO method with the highest reward AUC (paper 4x4 value: 2055.08) and lowest average steps to goal (7.24)
- Fine-tuned BERT execution accuracy improves by more than 9% over the pre-fine-tuned model
- ULPS reaches higher success rate and fewer environment interactions than Unguided PPO, Linear Decay, Q-Learning, and DQN trained on the same reward function
- All three log families are produced during training: episode_metrics_*.csv, entropy_values_*.csv, calibration_metrics_*.csv
- final_training_summary_*.png reproduces the paper's qualitative result (Fig. 2 trajectories, Fig. 3 radar) that ULPS leads on reward AUC and success rate

## Limitations

- Not all result files are regenerated automatically by a single script; some CSVs and plots under results/ are pre-generated artifacts
- Plotting and summarization scripts assume the same output folder structure used during the experiments; minor path adjustments may be required
- Only a subset of development plots was selected for the final paper, and the repository has no changelog

## Evidence
