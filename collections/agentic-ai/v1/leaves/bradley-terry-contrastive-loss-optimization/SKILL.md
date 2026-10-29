---
name: bradley-terry-contrastive-loss-optimization
description: Use when you have (prompt, chosen completion, rejected completion) preference
  pairs and need to train or reproduce a Bradley-Terry reward model, verifying the
  reward margin between chosen and rejected responses separates over training.
when_to_use_negative:
- The task is outcome reward modeling on verifiable scalar correctness signals (e.g.
  GSM8K-style math correctness) — the book prescribes code/reward_models/train_orm.py
  for that instead of train_preference_rm.
- You are applying direct alignment algorithms (e.g. DPO) that do not train an explicit
  reward model.
- Your data contains only single completions without chosen/rejected contrast, so
  the contrastive margin loss cannot be formed.
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_0526
- http://edamontology.org/topic_3473
tools:
- name: rlhf-book codebase
  role: Provides the companion train_preference_rm script in code/reward_models that
    implements the Bradley-Terry preference reward model training run
  repo: https://github.com/natolambert/rlhf-book
- name: TRL
  role: Training stack used to run the 1-epoch preference reward model optimization
  repo: https://github.com/huggingface/trl
- name: PyTorch
  role: Backend for computing per-batch chosen/rejected reward scores and the negative
    log-sigmoid margin loss
- name: UltraFeedback
  role: Source dataset of preference pairs (first of the preference-data category)
    subsampled to ~2000 examples
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:168ff04c3a83af1fa1e2713fadb5d8a4f9a6ec9c181803c503cdaadda1acf095
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/natolambert/rlhf-book
  tools:
  - PyTorch
  - Python
  - TRL
  - UltraFeedback
  - rlhf-book codebase
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_0526
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bradley-terry-contrastive-loss-optimization

## Summary

Train a Bradley-Terry preference reward model on UltraFeedback preference pairs (~2000 samples, 1 epoch) using the rlhf-book companion script reward_models.train_preference_rm, optimizing the negative log-sigmoid of the chosen-minus-rejected reward margin and monitoring that the margin grows over training.

## When to use

Use when you have (prompt, chosen completion, rejected completion) preference pairs and need to train or reproduce a Bradley-Terry reward model, verifying the reward margin between chosen and rejected responses separates over training.

## When NOT to use

- The task is outcome reward modeling on verifiable scalar correctness signals (e.g. GSM8K-style math correctness) — the book prescribes code/reward_models/train_orm.py for that instead of train_preference_rm.
- You are applying direct alignment algorithms (e.g. DPO) that do not train an explicit reward model.
- Your data contains only single completions without chosen/rejected contrast, so the contrastive margin loss cannot be formed.

## Inputs

- UltraFeedback preference pairs (prompt, chosen completion, rejected completion) from HuggingFace datasets
- Subsample size parameter (--samples 2000)
- Training duration parameter (--epochs 1)

## Outputs

- Trained Bradley-Terry preference reward model checkpoint
- Per-step training log records: step index, chosen_rewards, rejected_rewards, loss, margin
- W&B log curves for chosen/rejected rewards and margin

## How to apply

Load UltraFeedback preference pairs from HuggingFace datasets and subsample approximately 2000 examples. Instantiate the preference reward model via the train_preference_rm script from the rlhf-book code/reward_models module, using the standard Bradley-Terry contrastive loss — the negative log-sigmoid of the chosen-minus-rejected reward margin. Train for 1 epoch on the TRL/PyTorch stack, computing per-batch chosen and rejected reward scores. At each optimization step, log the step index, chosen_rewards, rejected_rewards, loss, and margin (chosen reward minus rejected reward) to a training log (e.g. training_log.l) and W&B. The rationale is that minimizing the contrastive loss pushes the chosen reward above the rejected reward, so a growing, non-diverging margin is the expected observable.

## Related tools

- **rlhf-book codebase** (Provides the companion train_preference_rm script in code/reward_models that implements the Bradley-Terry preference reward model training run) — https://github.com/natolambert/rlhf-book
- **TRL** (Training stack used to run the 1-epoch preference reward model optimization) — https://github.com/huggingface/trl
- **PyTorch** (Backend for computing per-batch chosen/rejected reward scores and the negative log-sigmoid margin loss)
- **UltraFeedback** (Source dataset of preference pairs (first of the preference-data category) subsampled to ~2000 examples)

## Examples

```
python -m reward_models.train_preference_rm --dataset UltraFeedback --samples 2000 --epochs 1
```

## Evaluation signals

- The logged reward margin (chosen reward minus rejected reward) exhibits a growing trend over the 1-epoch run, i.e. chosen reward separates from rejected reward.
- The Bradley-Terry contrastive loss (negative log-sigmoid of the margin) decreases monotonically or near-monotonically across optimization steps.
- Reward scores do not diverge: neither chosen_rewards nor rejected_rewards should blow up while the margin separates.
- The training log contains the required fields per step: step index, chosen_rewards, rejected_rewards, loss, and margin.
- The run reproduces the book's prescribed configuration: --samples 2000 and --epochs 1 on UltraFeedback.

## Limitations

- The demonstration is a small-scale teaching experiment (~2000 samples, 1 epoch); it validates the margin-growth observable, not production reward model quality.
- Behavior on other preference datasets (e.g. HelpSteer2-Preference) is not prescribed by this recipe; only UltraFeedback is specified.
- Software version numbers for the training stack (TRL, PyTorch) are not stated in the source, so exact numerical reproduction may vary across environments.

## Evidence

- [supplementary] task card: prescribed experiment: [evidence span withheld — anchor status: not_found]
- [supplementary] Bradley-Terry loss definition: [evidence span withheld — anchor status: not_found]
- [abstract] reward model stage in RLHF pipeline: "from starting with instruction tuning to training a reward model and finally all of rejection sampling, reinforcement learning,"
- [methods] UltraFeedback as preference dataset: "UltraFeedback (the first of this category)"
- [readme] rlhf-book README: companion codebase: "The book is released with a variety of companion resources, including a codebase, a library to compare model completions from within post-training stages"
