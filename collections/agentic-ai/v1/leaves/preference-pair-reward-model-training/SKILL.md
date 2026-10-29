---
name: preference-pair-reward-model-training
description: Use when you have pairwise preference data (prompt + chosen/rejected
  completions, e.g. UltraFeedback) and need to fit a reward model whose scalar scores
  separate chosen from rejected responses for downstream RL optimization.
when_to_use_negative:
- You only have single completions with verifiable correctness signals (e.g., GSM8K-style
  binary rewards) — use the outcome reward model path code/reward_models/train_orm.py
  instead of pairwise preference training
- Your goal is direct alignment without an explicit reward model (e.g., DPO-style
  direct alignment algorithms) rather than a scorer for PPO/GRPO
- The task is instruction tuning / SFT with completion_mask prompt-token masking rather
  than preference scoring
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TRL
  role: Training stack used for preference reward model training and the surrounding
    RLHF pipeline
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Backend for the Bradley-Terry contrastive loss and per-batch chosen/rejected
    reward computation
- name: Python
  role: Language for the reward_models.train_preference_rm companion script and dataset
    subsampling
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_content_hash: sha256:3f06934fd12f939a81c94c08eeca2a2d0b1d3d396d123ca59c43046b611b6ff4
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# preference-pair-reward-model-training

## Summary

Train a Bradley-Terry preference reward model on pairwise (chosen, rejected) completions — the book's Chapter 5 stage of the canonical RLHF recipe — using the rlhf-book companion script reward_models.train_preference_rm on UltraFeedback, and verify learning via a growing chosen-vs-rejected reward margin.

## When to use

Use when you have pairwise preference data (prompt + chosen/rejected completions, e.g. UltraFeedback) and need to fit a reward model whose scalar scores separate chosen from rejected responses for downstream RL optimization.

## When NOT to use

- You only have single completions with verifiable correctness signals (e.g., GSM8K-style binary rewards) — use the outcome reward model path code/reward_models/train_orm.py instead of pairwise preference training
- Your goal is direct alignment without an explicit reward model (e.g., DPO-style direct alignment algorithms) rather than a scorer for PPO/GRPO
- The task is instruction tuning / SFT with completion_mask prompt-token masking rather than preference scoring

## Inputs

- UltraFeedback preference pairs (prompt, chosen completion, rejected completion) from HuggingFace datasets, subsampled to ~2000 examples
- Training configuration for reward_models.train_preference_rm (e.g., --samples 2000, --epochs 1)
- Optionally, annotator rating deltas for the Llama-2 style margin term m(y_c, y_r)

## Outputs

- Trained Bradley-Terry preference reward model checkpoint
- training_log with per-step records: step index, chosen_rewards, rejected_rewards, loss, and reward margin
- W&B logs showing the chosen-vs-rejected reward margin trend

## How to apply

Load UltraFeedback preference pairs (prompt, chosen completion, rejected completion) from HuggingFace datasets and subsample roughly 2000 examples. Instantiate the reward model via the train_preference_rm script in the rlhf-book code/reward_models module, training with the Bradley-Terry contrastive loss (negative log-sigmoid of the chosen-minus-rejected reward margin), optionally adding the Llama-2 style margin term m(y_c, y_r) derived from annotator rating deltas. Train for 1 epoch with --samples 2000 --epochs 1 on the TRL/PyTorch stack, computing per-batch chosen and rejected reward scores. At each optimization step, log step index, chosen_rewards, rejected_rewards, loss, and margin (chosen minus rejected) to training_log. The rationale is that this reward model then serves as the scorer for the final RL stage (PPO/GRPO in code/policy_gradients), where rewards are penalized as rewards = rewards - self.beta * per_token_kl.

## Related tools

- **TRL** (Training stack used for preference reward model training and the surrounding RLHF pipeline) — https://github.com/huggingface/trl
- **PyTorch** (Backend for the Bradley-Terry contrastive loss and per-batch chosen/rejected reward computation)
- **Python** (Language for the reward_models.train_preference_rm companion script and dataset subsampling)

## Examples

```
python -m reward_models.train_preference_rm --samples 2000 --epochs 1 --dataset ultrafeedback
```

## Evaluation signals

- The logged reward margin (chosen reward minus rejected reward) exhibits a growing trend over the epoch, i.e., chosen reward separates from rejected reward
- The contrastive loss decreases monotonically-ish over training without divergence
- Reward scores remain finite (no blow-up/divergence of chosen or rejected rewards)
- training_log records contain the full schema per step: step index, chosen_rewards, rejected_rewards, loss, margin
- Downstream check: the checkpoint can be loaded as the RL-stage scorer producing finite reward values on sampled completions

## Limitations

- Reward model quality depends on preference data volume and quality; the book's demo uses only ~2000 UltraFeedback samples for 1 epoch, which is a minimal educational setup, not a production recipe
- Over-long generations and scoring location matter downstream: the companion RL stage scores completions only on the eos_token with a length penalty, so reward-model scores must be interpreted with that convention in mind
- Software version numbers for the named tools (TRL, PyTorch) are not stated, so reproducibility requires pinning environments yourself

## Evidence

- [methods] Bradley-Terry contrastive loss on UltraFeedback: [evidence span withheld — anchor status: not_found]
- [results] Growing reward margin observable: [evidence span withheld — anchor status: not_found]
- [methods] RL stage applies KL penalty to rewards: [evidence span withheld — anchor status: not_found]
