---
name: llm-guided-policy-shaping
description: Use when training RL agents on sparse-reward gridworld tasks (e.g., MiniGrid-UnlockPickup)
  where unguided PPO or standard RL baselines stall, and pretrained LLM guidance must
  be calibrated and uncertainty-gated before shaping the policy.
when_to_use_negative:
- Tasks with dense reward signals, where unguided PPO or standard RL baselines do
  not stall and uncertainty-gated LLM shaping adds overhead without benefit.
- 'Settings requiring fully automated one-script reproduction: not all result files
  are regenerated automatically by a single script, so some outputs must come from
  the pre-generated results/ folder.'
- Environments where an A* oracle cannot synthesize near-optimal symbolic trajectories
  for fine-tuning the LLM action predictor.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3474
tools:
- name: ULPS
  role: Main framework combining calibrated BERT LLM, MC dropout uncertainty, and
    entropy-based blending over PPO
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: BERT (bert-base-uncased)
  role: Fine-tuned LLM action predictor trained on A*-oracle trajectories
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: Proximal Policy Optimization (PPO)
  role: Learned RL policy that is entropy-blended with uncertainty-gated LLM guidance
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: MiniGrid
  role: Benchmark environment (MiniGrid-UnlockPickup, 4x4 and 8x8) on which ULPS and
    baselines are evaluated
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:641c363916bc411e2387363d1079f44a28d018bb600748535dd1a88cd7429aeb
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# llm-guided-policy-shaping

## Summary

Implements ULPS, an uncertainty-aware LLM-guided policy shaping framework in which a fine-tuned BERT action predictor, whose influence is conditioned on Monte Carlo dropout epistemic uncertainty, is entropy-blended with a PPO policy to solve sparse-reward MiniGrid-UnlockPickup tasks.

## When to use

Use when training RL agents on sparse-reward gridworld tasks (e.g., MiniGrid-UnlockPickup) where unguided PPO or standard RL baselines stall, and pretrained LLM guidance must be calibrated and uncertainty-gated before shaping the policy.

## When NOT to use

- Tasks with dense reward signals, where unguided PPO or standard RL baselines do not stall and uncertainty-gated LLM shaping adds overhead without benefit.
- Settings requiring fully automated one-script reproduction: not all result files are regenerated automatically by a single script, so some outputs must come from the pre-generated results/ folder.
- Environments where an A* oracle cannot synthesize near-optimal symbolic trajectories for fine-tuning the LLM action predictor.

## Inputs

- MiniGrid-UnlockPickup environment (4x4 and 8x8 grid configurations)
- A*-oracle-synthesized symbolic trajectories for fine-tuning the BERT-based action predictor
- bert-base-uncased pretrained language model checkpoint
- Experiment configuration: 1000 episodes, seed 42, 50-step episode cap

## Outputs

- episode_metrics_*.csv (episode-level logs)
- entropy_values_*.csv (entropy logs)
- calibration_metrics_*.csv (calibration metrics)
- experiment_auc_summary.csv (cross-method, cross-environment comparison)
- final_training_summary_*.png plots

## How to apply

First run an A*-based oracle on MiniGrid-UnlockPickup to synthesize optimal symbolic trajectories, then fine-tune a bert-base-uncased model as an action predictor on those trajectories. During training, estimate epistemic uncertainty via Monte Carlo dropout over the LLM and use an entropy-based blending mechanism to adaptively balance LLM guidance against the learned PPO policy. Run experiments with MainCode/run_4x4_experiment.py and MainCode/run_8x8_experiment.py for 1000 episodes, fixed seed 42, and a 50-step episode cap, comparing against Unguided PPO, Uncalibrated LLM, Linear Decay PPO, Q-Learning (Qlearning/), and DQN (DQN/) baselines under identical settings. Collect episode_metrics_*.csv, entropy_values_*.csv, and calibration_metrics_*.csv, then aggregate into per-method summaries (reward AUC, success rate, average steps to goal, average reward, total wins, total steps) written to experiment_auc_summary.csv. The rationale is that calibrated, uncertainty-gated LLM shaping reduces sample complexity in sparse-reward settings where either component alone fails (e.g., PPO alone on 8x8).

## Related tools

- **ULPS** (Main framework combining calibrated BERT LLM, MC dropout uncertainty, and entropy-based blending over PPO) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **BERT (bert-base-uncased)** (Fine-tuned LLM action predictor trained on A*-oracle trajectories) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **Proximal Policy Optimization (PPO)** (Learned RL policy that is entropy-blended with uncertainty-gated LLM guidance) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **MiniGrid** (Benchmark environment (MiniGrid-UnlockPickup, 4x4 and 8x8) on which ULPS and baselines are evaluated) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl

## Examples

```
python MainCode/run_8x8_experiment.py  # ULPS on MiniGrid-UnlockPickup 8x8: 1000 episodes, seed 42, 50-step cap; writes episode_metrics_*.csv and experiment_auc_summary.csv
```

## Evaluation signals

- experiment_auc_summary.csv exists with one row per method per grid size (4x4 and 8x8).
- ULPS rows show higher reward AUC and success rate than unguided PPO, uncalibrated LLM, Linear Decay PPO, Q-Learning, and DQN baselines.
- On 8x8 MiniGrid-UnlockPickup, ULPS reaches ~99.7% success while PPO alone fails almost completely and the uncalibrated LLM reaches only ~72.3%.
- ULPS shows fewer environment interactions (lower sample complexity) and higher reward AUC than baselines.
- Episode-level logs (episode_metrics_*.csv), entropy logs (entropy_values_*.csv), and calibration metrics (calibration_metrics_*.csv) are produced with consistent schemas across runs.

## Limitations

- Not all result files are regenerated automatically by a single script; some CSVs and plots were pre-generated during experimentation and are only included under results/.
- Some plotting and summarization scripts assume the same output folder structure used during the experiments, and code may require minor path adjustments locally.
- Several additional plots were generated during development, but only a subset was selected for the final paper for compactness.
- No changelog found in the repository.

## Evidence
