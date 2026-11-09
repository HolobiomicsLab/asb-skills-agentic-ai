---
name: ultrafeedback-dataset-loading
description: 'Use when a reward-model or preference-fine-tuning experiment needs (prompt,
  chosen, rejected) triplets: UltraFeedback is the prescribed general preference dataset
  in the rlhf-book for training a Bradley-Terry reward model with a growing chosen-vs-rejected
  margin.'
when_to_use_negative:
- The task is an outcome reward model (ORM) on math correctness (e.g. GSM8K-style)
  — the book routes those experiments through code/reward_models/train_orm.py instead
  of the preference RM path
- The data are already a pre-tokenized or pre-paired TRL preference dataset requiring
  no UltraFeedback loading/subsampling
- The goal is direct alignment (DPO-style) rather than explicit reward model training
  — the margin-logging observable does not apply
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: TRL
  role: Provides the training stack used by the companion preference reward model
    script for the preference-fine-tuning loop
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Backend for computing per-batch chosen/rejected reward scores and the Bradley-Terry
    contrastive loss (masked_mean-style tensor ops in the book's code)
- name: Python
  role: Language for the dataset-loading/subsampling step and the train_preference_rm
    entry point
- name: natolambert/rlhf-book
  role: Companion codebase supplying the train_preference_rm script in the code/reward_models
    module that consumes the loaded UltraFeedback pairs
  repo: https://github.com/natolambert/rlhf-book
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:c0d1095481268bf7c739e98f636f66414512d63497b79de2ceb13d1a7db11309
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
  - natolambert/rlhf-book
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ultrafeedback-dataset-loading

## Summary

Loads UltraFeedback preference pairs (prompt, chosen completion, rejected completion) from the HuggingFace datasets hub and subsamples ~2000 examples to feed the Bradley-Terry preference reward model training script from the rlhf-book companion codebase. It is the data-ingestion front end of the reward-model-training stage of the RLHF pipeline.

## When to use

Use when a reward-model or preference-fine-tuning experiment needs (prompt, chosen, rejected) triplets: UltraFeedback is the prescribed general preference dataset in the rlhf-book for training a Bradley-Terry reward model with a growing chosen-vs-rejected margin.

## When NOT to use

- The task is an outcome reward model (ORM) on math correctness (e.g. GSM8K-style) — the book routes those experiments through code/reward_models/train_orm.py instead of the preference RM path
- The data are already a pre-tokenized or pre-paired TRL preference dataset requiring no UltraFeedback loading/subsampling
- The goal is direct alignment (DPO-style) rather than explicit reward model training — the margin-logging observable does not apply

## Inputs

- UltraFeedback dataset from HuggingFace datasets (preference-style records with prompt, chosen completion, rejected completion)
- Subsample size parameter (approximately 2000 samples) and epoch count (1) for the demo experiment

## Outputs

- Subsampled set of ~2000 (prompt, chosen, rejected) preference pairs formatted for train_preference_rm
- Per-step training log records: step index, chosen_rewards, rejected_rewards, loss, and reward margin (demo log and/or W&B)
- Trained Bradley-Terry preference reward model checkpoint

## How to apply

Load UltraFeedback from the HuggingFace datasets hub, extract each preference pair as a prompt with a chosen completion and a rejected completion, and subsample approximately 2000 examples as specified by the book's demo experiment. Instantiate the preference reward model using the train_preference_rm script from the rlhf-book code/reward_models module, which applies the standard Bradley-Terry contrastive loss (negative log-sigmoid of the chosen-minus-rejected reward margin) on top of the TRL/PyTorch training stack. Train for 1 epoch (--samples 2000 --epochs 1), computing per-batch chosen and rejected reward scores, and log step index, chosen_rewards, rejected_rewards, loss, and margin records to a training log (training_log.l / W&B). The rationale is that a correct run should show the reward margin growing over the epoch — chosen rewards separating from rejected rewards — while the contrastive loss decreases, mirroring the book's reward-model-training stage of the canonical RLHF recipe.

## Related tools

- **TRL** (Provides the training stack used by the companion preference reward model script for the preference-fine-tuning loop) — https://github.com/huggingface/trl
- **PyTorch** (Backend for computing per-batch chosen/rejected reward scores and the Bradley-Terry contrastive loss (masked_mean-style tensor ops in the book's code))
- **Python** (Language for the dataset-loading/subsampling step and the train_preference_rm entry point)
- **natolambert/rlhf-book** (Companion codebase supplying the train_preference_rm script in the code/reward_models module that consumes the loaded UltraFeedback pairs) — https://github.com/natolambert/rlhf-book

## Examples

```
python -m reward_models.train_preference_rm --samples 2000 --epochs 1
```

## Evaluation signals

- The logged reward margin (chosen reward minus rejected reward) exhibits a growing trend over the 1-epoch run, with chosen rewards separating from rejected rewards
- Reward scores do not diverge during training, consistent with the contrastive loss decreasing
- The training log records contain the expected schema: step index, chosen_rewards, rejected_rewards, loss, and margin per optimization step
- The subsample contains approximately 2000 UltraFeedback preference pairs, each with non-empty prompt, chosen, and rejected completions
- Loss values in the demo/W&B logs monotonically decrease across the epoch

## Limitations

- Software version numbers for the named tools (PyTorch, TRL, etc.) are not stated, so exact reproducibility of the run may vary
- UltraFeedback is a general-preference dataset; it is not suited to correctness-style reward signals — the book's ORM experiments instead use GSM8K
- Margin growth is verified qualitatively (a growing trend without divergence), not against a published numeric threshold

## Evidence

- [intro] TRL is the named training stack used in the book's ecosystem: [evidence span withheld — anchor status: not_found]
