---
name: llm-action-distribution-normalization
description: Use when combining a fine-tuned LLM action predictor's distribution with
  a learned RL policy (e.g. PPO) in sparse-reward environments like MiniGrid-UnlockPickup,
  and you need uncertainty-conditioned weighting.
when_to_use_negative:
- When no uncertainty estimate is needed and a fixed guidance schedule (e.g. Linear
  Decay baseline) suffices
- When the action space is continuous or the LLM output is not a categorical action
  distribution, so entropy normalization is ill-defined
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ULPS
  role: Main framework whose calibrated BERT LLM action predictor and entropy-based
    blending mechanism are reconstructed here
  repo: github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: Proximal Policy Optimization (PPO)
  role: Learned policy P_agent that is convexly blended with the LLM distribution
    using H_norm weights
- name: BERT (bert-base-uncased)
  role: Fine-tuned LLM action predictor whose MC-dropout sampled distribution P_LLM
    supplies the entropy signal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:4f5cf424a618a457282173d03f3753f08bcb2e83a359974f042de2132933f774
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BERT (bert-base-uncased)
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# llm-action-distribution-normalization

## Summary

Computes the normalized entropy H_norm of the LLM-derived action distribution (sampled via Monte Carlo dropout over a fine-tuned BERT-based action predictor) and uses it to blend the LLM policy with the PPO policy as P_final = (1 − H_norm)·P_LLM + H_norm·P_agent.

## When to use

Use when combining a fine-tuned LLM action predictor's distribution with a learned RL policy (e.g. PPO) in sparse-reward environments like MiniGrid-UnlockPickup, and you need uncertainty-conditioned weighting.

## When NOT to use

- When no uncertainty estimate is needed and a fixed guidance schedule (e.g. Linear Decay baseline) suffices
- When the action space is continuous or the LLM output is not a categorical action distribution, so entropy normalization is ill-defined

## Inputs

- Stochastic forward-pass outputs of the calibrated BERT-based LLM action predictor (MC dropout samples)
- PPO agent's action policy distribution P_agent
- MainCode/ ULPS implementation (calibrated LLM + PPO components)

## Outputs

- Blended action distribution P_final = (1 − H_norm)·P_LLM + H_norm·P_agent (valid probability distribution)
- Normalized entropy values H_norm per step
- entropy_values_*.csv per-step entropy logs compatible with run_4x4/run_8x8 experiment runners

## How to apply

Run T stochastic Monte Carlo dropout forward passes over the calibrated BERT-based LLM action head to obtain the sampled action distribution P_LLM and compute its normalized entropy H_norm. Implement the convex blend P_final = (1 − H_norm)·P_LLM + H_norm·P_agent as a standalone module (e.g. entropy_blending.py), where H_norm = 0 recovers the pure LLM policy and H_norm = 1 recovers the pure PPO policy, so the LLM's epistemic uncertainty sets how much the learned policy dominates. Verify boundary cases and that P_final remains a valid probability distribution against Algorithm 1 of Section II-B. Emit per-step entropy values in the entropy_values_*.csv log format consumed by the run_4x4/run_8x8 experiment scripts, round-trippable with calibration_metrics_*.csv records.

## Related tools

- **ULPS** (Main framework whose calibrated BERT LLM action predictor and entropy-based blending mechanism are reconstructed here) — github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **Proximal Policy Optimization (PPO)** (Learned policy P_agent that is convexly blended with the LLM distribution using H_norm weights)
- **BERT (bert-base-uncased)** (Fine-tuned LLM action predictor whose MC-dropout sampled distribution P_LLM supplies the entropy signal)

## Examples

```
P_final = (1 - H_norm) * P_LLM + H_norm * P_agent  # entropy_blending.py, validated against entropy_values_*.csv logs from run_4x4_experiment.py
```

## Evaluation signals

- Blended distribution matches the reference blend computed directly from P_final = (1 − H_norm)·P_LLM + H_norm·P_agent for all test fixtures within floating-point tolerance
- Boundary cases hold: H_norm = 0 recovers pure P_LLM and H_norm = 1 recovers pure P_agent
- P_final is a valid probability distribution (non-negative entries summing to 1)
- Module verified against the equation and Algorithm 1 of Section II-B of the paper
- Entropy logs round-trip with the calibration_metrics_*.csv records

## Limitations

- Some results (CSV files and plots) were pre-generated during experimentation and are not all regenerated automatically by a single script
- The codebase may require minor path adjustments depending on the local environment
- Uncertainty is epistemic only (MC dropout over the BERT action head); aleatoric uncertainty of the environment is not modeled

## Evidence

- [full_text] During training, the final action policy is a convex combination of the LLM-derived action distribution and the PPO agent's policy: [evidence span withheld — anchor status: not_found]
