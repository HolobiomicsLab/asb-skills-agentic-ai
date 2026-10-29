---
name: reinforcement-learning-policy-shaping
description: Use when training a sparse-reward RL agent whose action distribution
  should be shaped by a language-model action prior, with the blend weight determined
  by the LLM's normalized predictive entropy.
when_to_use_negative:
- The task is dense-reward or the LLM action prior has no calibrated uncertainty estimate
  — unguided PPO, Linear Decay, Q-Learning, or DQN baselines are the intended comparisons
  instead
- Only a single deterministic LLM forward pass is available (no Monte Carlo dropout
  sampling), since H_norm cannot be estimated without stochastic passes
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: ULPS
  role: Main framework providing the calibrated LLM action predictor and the entropy-based
    blending mechanism between the LLM policy and PPO policy
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: Proximal Policy Optimization (PPO)
  role: Learned RL policy P_agent that is convexly blended with the LLM-derived action
    distribution using H_norm
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: BERT (bert-base-uncased)
  role: Fine-tuned language model whose MC-dropout sampled action distribution P_LLM
    and normalized entropy H_norm drive the blend weight
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:ebb7b34d4927c79d8aaf86188adbca3ee30f11f8c5854a6ffc32d5134198b284
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - BERT (bert-base-uncased)
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# reinforcement-learning-policy-shaping

## Summary

Implements ULPS, an entropy-based policy-fusion mechanism that convexly blends an LLM-derived action prior (fine-tuned BERT with Monte Carlo dropout uncertainty) with a PPO policy for sparse-reward MiniGrid-UnlockPickup tasks.

## When to use

Use when training a sparse-reward RL agent whose action distribution should be shaped by a language-model action prior, with the blend weight determined by the LLM's normalized predictive entropy.

## When NOT to use

- The task is dense-reward or the LLM action prior has no calibrated uncertainty estimate — unguided PPO, Linear Decay, Q-Learning, or DQN baselines are the intended comparisons instead
- Only a single deterministic LLM forward pass is available (no Monte Carlo dropout sampling), since H_norm cannot be estimated without stochastic passes

## Inputs

- Fine-tuned BERT-based LLM action predictor with Monte Carlo dropout (from MainCode/)
- PPO agent policy distribution P_agent
- Sampled LLM action distribution P_LLM and its normalized entropy H_norm
- MiniGrid-UnlockPickup environment episodes

## Outputs

- Blended action policy P_final = (1 - H_norm)·P_LLM + H_norm·P_agent
- Per-step entropy logs (entropy_values_*.csv)
- Calibration metrics records (calibration_metrics_*.csv)
- Episode-level CSV logs and summaries from run_4x4/run_8x8 experiment scripts

## How to apply

Run T stochastic Monte Carlo dropout forward passes through the calibrated BERT-based LLM action head to obtain the sampled action distribution P_LLM and its normalized entropy H_norm. Compute the final action policy as the convex combination P_final = (1 - H_norm)·P_LLM + H_norm·P_agent, so H_norm = 0 recovers the pure LLM policy and H_norm = 1 recovers the pure PPO policy — high LLM uncertainty automatically defers to the learned PPO agent. Implement this blend as a standalone module (e.g., entropy_blending.py) with unit fixtures checking both boundary cases and that P_final is a valid probability distribution. Validate the blended distribution against the reference blend computed directly from the equation (agreement within floating-point tolerance) and against Algorithm 1 of Section II-B. Emit per-step entropy values in the entropy_values_*.csv log format consumed by the run_4x4/run_8x8 experiment scripts, and round-trip them with the calibration_metrics_*.csv records.

## Related tools

- **ULPS** (Main framework providing the calibrated LLM action predictor and the entropy-based blending mechanism between the LLM policy and PPO policy) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **Proximal Policy Optimization (PPO)** (Learned RL policy P_agent that is convexly blended with the LLM-derived action distribution using H_norm) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **BERT (bert-base-uncased)** (Fine-tuned language model whose MC-dropout sampled action distribution P_LLM and normalized entropy H_norm drive the blend weight) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl

## Examples

```
python MainCode/run_4x4_experiment.py
```

## Evaluation signals

- Blended distribution P_final matches the reference blend computed directly from P_final = (1 - H_norm)·P_LLM + H_norm·P_agent for all test fixtures within floating-point tolerance
- Boundary cases hold: H_norm = 0 yields exactly P_LLM; H_norm = 1 yields exactly P_agent
- P_final is a valid probability distribution (entries in [0, 1], summing to 1) across fixtures
- Per-step entropy logs round-trip with the calibration_metrics_*.csv records
- The standalone entropy_blending.py script runs without path adjustments beyond those noted in the repository disclaimer

## Limitations

- Not all result files are regenerated automatically by a single script; some CSVs and plots were pre-generated during experimentation and included under results/
- Some plotting and summarization scripts assume the same output folder structure used during the experiments
- The codebase may require minor path adjustments depending on the local environment, per the repository disclaimer

## Evidence
