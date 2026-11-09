---
name: entropy-based-policy-blending
description: Use when fusing an LLM-derived action prior with a learned PPO policy
  so that the influence of the LLM guidance is adaptively down-weighted as its epistemic
  uncertainty (distributional entropy) rises.
when_to_use_negative:
- When the LLM guidance is fully calibrated and entropy-free (H_norm = 0 everywhere),
  the mechanism degenerates to pure LLM policy and blending is unnecessary
- When the goal is a time-based (rather than uncertainty-based) guidance schedule,
  e.g. the Linear Decay baseline in the paper
- When the base RL algorithm is not PPO and the action distributions are not comparable
  probability vectors over the same action space
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ULPS
  role: Main framework whose calibrated BERT-based LLM predictor and PPO agent components
    produce the two distributions being blended
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: Proximal Policy Optimization (PPO)
  role: Learned policy whose action distribution P_agent forms the H_norm-weighted
    term of the convex blend
- name: BERT (bert-base-uncased)
  role: Fine-tuned LLM action predictor whose Monte Carlo dropout-sampled distribution
    P_LLM supplies the (1 − H_norm)-weighted term and the entropy signal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:58fc00f4f926255b0a7c79d2b953c5abb5214535c4e802e5d8b6a2bc58541095
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - BERT (bert-base-uncased)
  - Proximal Policy Optimization (PPO)
  - ULPS
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# entropy-based-policy-blending

## Summary

Implements ULPS's entropy-based convex blending in which the final action distribution P_final = (1 − H_norm)·P_LLM + H_norm·P_agent, with the normalized entropy of the Monte Carlo dropout-sampled LLM action distribution setting the relative weight between LLM guidance and the PPO policy during training.

## When to use

Use when fusing an LLM-derived action prior with a learned PPO policy so that the influence of the LLM guidance is adaptively down-weighted as its epistemic uncertainty (distributional entropy) rises.

## When NOT to use

- When the LLM guidance is fully calibrated and entropy-free (H_norm = 0 everywhere), the mechanism degenerates to pure LLM policy and blending is unnecessary
- When the goal is a time-based (rather than uncertainty-based) guidance schedule, e.g. the Linear Decay baseline in the paper
- When the base RL algorithm is not PPO and the action distributions are not comparable probability vectors over the same action space

## Inputs

- LLM action distribution P_LLM sampled via T Monte Carlo dropout forward passes over the fine-tuned BERT-based action predictor
- PPO agent's policy distribution P_agent from the trained/learning PPO agent
- Normalized entropy H_norm of the LLM action distribution

## Outputs

- Blended action distribution P_final (a valid probability distribution over actions)
- Per-step entropy logs in the entropy_values_*.csv format
- Standalone blending module (entropy_blending.py) with unit test fixtures

## How to apply

Run T stochastic Monte Carlo dropout forward passes over the calibrated BERT-based LLM action head to obtain the sampled action distribution P_LLM and compute its normalized entropy H_norm. Implement a standalone blending module (e.g. entropy_blending.py) computing the convex combination P_final = (1 - H_norm) * P_LLM + H_norm * P_agent, where H_norm = 0 recovers the pure LLM policy and H_norm = 1 recovers the pure PPO policy. Verify the module against the equation and Algorithm 1 of Section II-B using unit fixtures that check the boundary cases and confirm P_final remains a valid probability distribution. Emit per-step entropy values in the entropy_values_*.csv log format consumed by the run_4x4/run_8x8 experiment scripts so the blend can be audited against calibration_metrics_*.csv records.

## Related tools

- **ULPS** (Main framework whose calibrated BERT-based LLM predictor and PPO agent components produce the two distributions being blended) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **Proximal Policy Optimization (PPO)** (Learned policy whose action distribution P_agent forms the H_norm-weighted term of the convex blend)
- **BERT (bert-base-uncased)** (Fine-tuned LLM action predictor whose Monte Carlo dropout-sampled distribution P_LLM supplies the (1 − H_norm)-weighted term and the entropy signal)

## Examples

```
python entropy_blending.py --p_llm p_llm.npy --p_agent p_agent.npy --t_dropout 10 --out entropy_values_4x4.csv
```

## Evaluation signals

- Blended distribution matches the reference blend computed directly from P_final = (1 - H_norm) * P_LLM + H_norm * P_agent for all test fixtures within floating-point tolerance
- Boundary cases verified: H_norm = 0 recovers the pure LLM policy and H_norm = 1 recovers the pure PPO policy
- P_final is a valid probability distribution (non-negative entries summing to 1)
- Entropy logs round-trip with the calibration_metrics_*.csv records
- Per-step entropy values are emitted in the entropy_values_*.csv format consumed by run_4x4/run_8x8 experiment scripts

## Limitations

- The codebase reflects the structure used during research; some parts may require minor path adjustments depending on the local environment
- Not all result files are regenerated automatically by a single script; some CSV files and plots were pre-generated during experimentation
- Uncertainty is epistemic only, estimated via Monte Carlo dropout on the BERT action head; it does not capture aleatoric task uncertainty

## Evidence

- [methods] blend formula: [evidence span withheld — anchor status: not_found]
