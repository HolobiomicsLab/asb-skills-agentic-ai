---
name: kl-penalty-reward-shaping
description: Use when optimizing an instruction-tuned policy against a trained reward
  model (e.g., PPO or GRPO from an SFT checkpoint) and you need to keep the policy
  from drifting too far from the reference while maximizing reward-model-ranked completions.
when_to_use_negative:
- Task is direct alignment (e.g., DPO-style preference fine-tuning) rather than an
  RL optimization stage — no reward model or per-token KL shaping is involved
- Long RL runs where the field's practice is to remove the KL penalty, or a job where
  you only need rejection sampling or on-policy distillation rather than policy-gradient
  optimization
- Reward-model training itself (Bradley-Terry loss on preference pairs) — the KL shaping
  belongs to the downstream RL stage only
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TRL
  role: Reinforcement-learning training library for the RLHF optimization stage (PPO/GRPO)
    with reward-model scoring
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Implements the masked per-token loss aggregation, KL penalty arithmetic, and
    clipped PPO objective (e.g., masked_mean over completion_mask)
- name: vLLM
  role: Efficient inference engine for sampling completions from the policy inside
    the RL loop
- name: Ray
  role: Distributed process management to hand information off between the policy-gradient
    learning loop
- name: FSDP
  role: Training framework whose kernels/precision differ from the inference engine,
    relevant to consistency of KL and logprob computations
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:cdc31286c476307cc5f1eb21628b35aa79a798446e4ee0000976d1ef40b7b6a3
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/huggingface/trl
  tools:
  - FSDP
  - PyTorch
  - Python
  - Ray
  - TRL
  - vLLM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# KL-penalty reward shaping

## Summary

Shapes the RL optimization stage of the canonical three-stage RLHF pipeline (instruction tuning → reward model → RL) by subtracting a beta-scaled per-token KL penalty against the SFT reference policy directly from the reward-model scores before computing policy-gradient advantages.

## When to use

Use when optimizing an instruction-tuned policy against a trained reward model (e.g., PPO or GRPO from an SFT checkpoint) and you need to keep the policy from drifting too far from the reference while maximizing reward-model-ranked completions.

## When NOT to use

- Task is direct alignment (e.g., DPO-style preference fine-tuning) rather than an RL optimization stage — no reward model or per-token KL shaping is involved
- Long RL runs where the field's practice is to remove the KL penalty, or a job where you only need rejection sampling or on-policy distillation rather than policy-gradient optimization
- Reward-model training itself (Bradley-Terry loss on preference pairs) — the KL shaping belongs to the downstream RL stage only

## Inputs

- SFT checkpoint of an instruction-tuned language model (book default: Qwen/Qwen3-1.7B)
- Trained reward model (e.g., Bradley-Terry model trained on UltraFeedback pairwise preferences)
- Task-representative set of prompts (e.g., GSM8K-style tasks)
- Policy-gradient config (e.g., policy_gradients/configs/grpo.yaml) with beta, cliprange, gamma=1.0, optional GAE lam=0.95

## Outputs

- RL-optimized policy checkpoint shifted toward higher-reward behavior
- Per-token KL divergence statistics and shaped rewards = rewards - beta * per_token_kl
- Training logs with finite loss and reward values

## How to apply

Initialize the RL stage (e.g., policy_gradients with PPO or a GRPO config like policy_gradients/configs/grpo.yaml) from the SFT checkpoint, using the reward model as the completion scorer. Sample prompts from a task-representative set, generate completions from the policy, and score them with the reward model (scoring only on the eos_token with a length penalty for over-long generations). Shape the reward by applying the KL penalty directly to rewards as rewards = rewards - self.beta * per_token_kl, comparing the policy against the SFT reference per token. Then compute the PPO clipped objective (ratio = exp(new_per_token_logps - per_token_logps), pg_losses2 = -advantages * clamp(ratio, 1-eps, 1+eps), eps = cliprange such as 0.2) or GRPO group-relative advantages (r_i - mean)/std, aggregate loss per sequence as ((per_token_loss * completion_mask).sum(dim=1) / completion_mask.sum(dim=1)).mean(), and monitor finite loss/reward values. The beta coefficient is the key decision point: a recent filtering trend in the field is to remove the KL penalty entirely as RL runs lengthen, so run ablations with and without it.

## Related tools

- **TRL** (Reinforcement-learning training library for the RLHF optimization stage (PPO/GRPO) with reward-model scoring) — https://github.com/huggingface/trl
- **PyTorch** (Implements the masked per-token loss aggregation, KL penalty arithmetic, and clipped PPO objective (e.g., masked_mean over completion_mask))
- **vLLM** (Efficient inference engine for sampling completions from the policy inside the RL loop)
- **Ray** (Distributed process management to hand information off between the policy-gradient learning loop)
- **FSDP** (Training framework whose kernels/precision differ from the inference engine, relevant to consistency of KL and logprob computations)

## Examples

```
rewards = rewards - self.beta * per_token_kl
```

## Evaluation signals

- RL stage logs finite loss and reward values for the whole run
- Shaped rewards equal raw reward-model scores minus self.beta * per_token_kl per token, so rewards are non-increasing in KL divergence from the SFT reference
- All three stage checkpoints (SFT, reward model, RL) exist in the packaged rlhf_pipeline.tar.gz
- Advantages are normalized per batch (per-sequence GRPO, per-token DAPO, or fixed-length Dr. GRPO normalization) and have zero mean
- Clipped objective respects eps = cliprange (e.g., 0.2): ratio clamped to [1-eps, 1+eps] before the torch.max of the two policy losses

## Limitations

- The KL penalty couples the policy to the SFT reference; as RL runs lengthen, removing the KL penalty is a noted filtering/ablation trend, so a fixed beta may not be optimal across run lengths
- Completions are scored only on the eos_token with a length penalty for over-long generations, so reward shaping can be sensitive to generation length behavior
- Learner–sampler logprob mismatches (vLLM inference vs FSDP training kernels, precision, parallelism) can bias importance ratios; TIS for the learner–sampler ratio is adopted across open-source RL frameworks to compensate

## Evidence

- [full_text] KL penalty applied directly to shaped rewards: [evidence span withheld — anchor status: not_found]
- [methods] Remove KL penalty is a documented filter step: [evidence span withheld — anchor status: not_found]
