---
name: huggingface-model-fine-tuning
description: Use when you need to fine-tune or train a reward model on preference
  data (e.g., UltraFeedback prompt/chosen/rejected pairs) with the book's companion
  scripts and verify a growing reward margin.
when_to_use_negative:
- Your goal is supervised fine-tuning on instruction/output pairs (e.g., Stanford
  Alpaca 52K instruction data) rather than contrastive preference training on chosen/rejected
  pairs.
- You need a reward model for math-question correctness (binary ORM), for which the
  book prescribes code/reward_models/train_orm.py on GSM8K-style data instead.
- The dataset lacks explicit chosen/rejected preference structure, making the Bradley-Terry
  contrastive margin loss inapplicable.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TRL
  role: Transformers Reinforcement Learning library used as the training stack for
    post-training stages, including preference/reward-model training
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Underlying tensor/training framework computing per-batch chosen and rejected
    reward scores in the preference RM loop
- name: natolambert/rlhf-book
  role: Source of the companion codebase, including the reward_models/train_preference_rm
    script and its expected observable (growing reward margin)
  repo: https://github.com/natolambert/rlhf-book
- name: UltraFeedback
  role: Preference dataset supplying ~2000 (prompt, chosen, rejected) training pairs
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:18f3f9d6359808cdf0fdee168626b77cb15e43afa5e4ea20f74579617d4252a3
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/huggingface/trl
  tools:
  - PyTorch
  - Python
  - TRL
  - UltraFeedback
  - natolambert/rlhf-book
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# huggingface-model-fine-tuning

## Summary

Fine-tune and evaluate language models with the Hugging Face training stack (TRL, PyTorch), as exemplified in the RLHF Book by training a Bradley-Terry preference reward model on ~2000 UltraFeedback preference pairs and tracking the chosen-vs-rejected reward margin.

## When to use

Use when you need to fine-tune or train a reward model on preference data (e.g., UltraFeedback prompt/chosen/rejected pairs) with the book's companion scripts and verify a growing reward margin.

## When NOT to use

- Your goal is supervised fine-tuning on instruction/output pairs (e.g., Stanford Alpaca 52K instruction data) rather than contrastive preference training on chosen/rejected pairs.
- You need a reward model for math-question correctness (binary ORM), for which the book prescribes code/reward_models/train_orm.py on GSM8K-style data instead.
- The dataset lacks explicit chosen/rejected preference structure, making the Bradley-Terry contrastive margin loss inapplicable.

## Inputs

- UltraFeedback preference pairs (prompt, chosen completion, rejected completion) from HuggingFace datasets, subsampled to ~2000 examples
- Training hyperparameters: --samples 2000, --epochs 1
- A language model backbone (e.g., Qwen/Qwen3-1.7B in the book's default configs)

## Outputs

- Per-step training log (training_log.l) with records of step index, chosen_rewards, rejected_rewards, loss, and margin
- W&B logs showing chosen-vs-rejected reward margin over training
- Trained Bradley-Terry preference reward model checkpoint

## How to apply

Load UltraFeedback preference pairs (prompt, chosen completion, rejected completion) from HuggingFace datasets and subsample approximately 2000 examples. Instantiate the preference reward model with the train_preference_rm script from the rlhf-book code/reward_models module, using the standard Bradley-Terry contrastive loss (negative log-sigmoid of the chosen-minus-rejected reward margin), and train for 1 epoch with the TRL/PyTorch training stack. At each optimization step compute per-batch chosen_rewards, rejected_rewards, loss, and margin (chosen minus rejected) and log them (e.g., to training_log.l and W&B). The run succeeds when the logged margin exhibits a growing trend over the epoch — chosen rewards separating from rejected rewards — without reward scores diverging, consistent with the contrastive loss decreasing.

## Related tools

- **TRL** (Transformers Reinforcement Learning library used as the training stack for post-training stages, including preference/reward-model training) — https://github.com/huggingface/trl
- **PyTorch** (Underlying tensor/training framework computing per-batch chosen and rejected reward scores in the preference RM loop)
- **natolambert/rlhf-book** (Source of the companion codebase, including the reward_models/train_preference_rm script and its expected observable (growing reward margin)) — https://github.com/natolambert/rlhf-book
- **UltraFeedback** (Preference dataset supplying ~2000 (prompt, chosen, rejected) training pairs)

## Examples

```
python code/reward_models/train_preference_rm.py --samples 2000 --epochs 1
```

## Evaluation signals

- Logged reward margin (chosen reward minus rejected reward) shows a growing trend across the single epoch's optimization steps.
- Chosen rewards separate upward from rejected rewards while neither reward score diverges.
- Contrastive (negative log-sigmoid margin) loss decreases monotonically enough to be consistent with the margin growth.
- training_log.l records are well-formed: each entry has step index, chosen_rewards, rejected_rewards, loss, and margin fields.
- Approximately 2000 UltraFeedback examples were consumed in exactly 1 epoch as configured.

## Limitations

- The prescribed demo is tiny (~2000 samples, 1 epoch) and validates the mechanism (margin growth), not a production-quality reward model.
- Software version numbers for named tools (PyTorch, TRL, etc.) are not stated in the article, so reproducibility of the exact environment is not guaranteed.
- Reward-score divergence is a failure mode to watch for; margin growth must occur without unbounded reward magnitudes.

## Evidence

- [methods] task_002 preference RM experiment: [evidence span withheld — anchor status: not_found]
- [methods] Bradley-Terry loss: [evidence span withheld — anchor status: not_found]
- [readme] TRL role: [evidence span withheld — anchor status: not_found]
