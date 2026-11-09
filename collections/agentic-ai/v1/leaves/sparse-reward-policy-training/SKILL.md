---
name: sparse-reward-policy-training
description: Use when an agent must learn a policy in a sparse-reward gridworld (e.g.,
  MiniGrid-UnlockPickup) and you want LLM guidance whose influence is conditioned
  on epistemic uncertainty via Monte Carlo dropout.
when_to_use_negative:
- The task provides dense reward signals, so uncertainty-weighted LLM shaping is unnecessary
  overhead
- You need a single script that regenerates all result files automatically — the README
  states not all result files are regenerated automatically by a single script
- The environment is not MiniGrid UnlockPickup and no equivalent oracle trajectories
  can be synthesized for fine-tuning the BERT action predictor
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ULPS
  role: Main framework combining calibrated LLM + PPO with MC dropout uncertainty
    and entropy-based blending
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: BERT (bert-base-uncased)
  role: Fine-tuned LLM action predictor trained on A*-synthesized oracle trajectories
- name: Proximal Policy Optimization (PPO)
  role: Learned RL policy that is blended with LLM guidance via the entropy-based
    mechanism
- name: MiniGrid
  role: UnlockPickup benchmark environment (4x4 and 8x8) used for evaluation
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:8cddd01a0d7e9aa2305b82a2d0357e6569757e73d0302cdd9679cf64b4ef68e6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - BERT (bert-base-uncased)
  - MiniGrid
  - PPO
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# sparse-reward-policy-training

## Summary

Train an uncertainty-aware LLM-guided policy (ULPS) that blends a fine-tuned BERT-based action predictor with PPO via entropy-based blending to solve sparse-reward MiniGrid-UnlockPickup tasks more sample-efficiently than unguided or uncalibrated baselines.

## When to use

Use when an agent must learn a policy in a sparse-reward gridworld (e.g., MiniGrid-UnlockPickup) and you want LLM guidance whose influence is conditioned on epistemic uncertainty via Monte Carlo dropout.

## When NOT to use

- The task provides dense reward signals, so uncertainty-weighted LLM shaping is unnecessary overhead
- You need a single script that regenerates all result files automatically — the README states not all result files are regenerated automatically by a single script
- The environment is not MiniGrid UnlockPickup and no equivalent oracle trajectories can be synthesized for fine-tuning the BERT action predictor

## Inputs

- MiniGrid-UnlockPickup environment configuration (4x4 and 8x8 grid sizes)
- A*-based oracle symbolic trajectories for BERT fine-tuning
- Fine-tuned bert-base-uncased LLM action predictor
- PPO policy with Monte Carlo dropout uncertainty estimation
- Baseline agent implementations from Qlearning/ and DQN/ folders

## Outputs

- episode_metrics_*.csv episode-level logs
- entropy_values_*.csv entropy logs
- calibration_metrics_*.csv calibration metrics
- experiment_auc_summary.csv consolidated cross-method, cross-environment comparison
- final_training_summary_*.png training summary plots

## How to apply

Synthesize optimal symbolic trajectories with an A*-based oracle and use them to fine-tune a bert-base-uncased LLM action predictor. Estimate epistemic uncertainty with Monte Carlo dropout over multiple stochastic forward passes, and use an entropy-based blending mechanism to adaptively balance LLM guidance against the PPO policy. Run the 4x4 experiment via MainCode/run_4x4_experiment.py and the 8x8 experiment via run_8x8_experiment.py for 1000 episodes with seed 42 and a 50-step episode cap, recording episode_metrics_*.csv, entropy_values_*.csv, and calibration_metrics_*.csv per run. Run baselines (Unguided PPO, Uncalibrated LLM, Linear Decay, Q-Learning, DQN) under identical settings, then aggregate reward AUC, success rate, average steps to goal, average reward, total wins, and total steps into experiment_auc_summary.csv. Correct application is confirmed when the ULPS rows show higher reward AUC and success rate than the unguided, uncalibrated, and standard RL baselines.

## Related tools

- **ULPS** (Main framework combining calibrated LLM + PPO with MC dropout uncertainty and entropy-based blending) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **BERT (bert-base-uncased)** (Fine-tuned LLM action predictor trained on A*-synthesized oracle trajectories)
- **Proximal Policy Optimization (PPO)** (Learned RL policy that is blended with LLM guidance via the entropy-based mechanism)
- **MiniGrid** (UnlockPickup benchmark environment (4x4 and 8x8) used for evaluation)

## Examples

```
python MainCode/run_4x4_experiment.py  # trains ULPS (Calibrated LLM + PPO) on MiniGrid UnlockPickup 4x4, 1000 episodes, seed 42, 50-step cap
```

## Evaluation signals

- experiment_auc_summary.csv contains one row per method per grid size with non-null reward AUC and success-rate values
- ULPS rows show higher reward AUC and success rate than unguided PPO, uncalibrated LLM, Linear Decay, Q-Learning, and DQN baselines
- ULPS achieves more than 9% improvement in execution accuracy after fine-tuning and requires fewer environment interactions
- In the 8x8 environment ULPS reaches ~99.7% success versus 72.3% for the uncalibrated LLM and near-total failure for PPO alone
- final_training_summary_*.png plots are generated for each run and are consistent with the CSV logs

## Limitations

- Some results (CSV files and plots) were generated during experimentation and only pre-generated copies are included under results/
- Not all result files are regenerated automatically by a single script; plotting and summarization scripts assume the same output folder structure used during the experiments
- The codebase may require minor path adjustments depending on the local environment
- Only a subset of development plots was selected for the final paper for compactness; no changelog was found

## Evidence

- [results] the proposed ULPS model achieved 99.7% success with much lower cost: [evidence span withheld — anchor status: not_found]
