---
name: reward-margin-convergence-monitoring
description: Use when training a preference reward model on (prompt, chosen, rejected)
  pairs and you need to verify that the reward margin between chosen and rejected
  completions grows during training without divergence.
when_to_use_negative:
- Training an outcome reward model (ORM) on a binary correctness signal such as GSM8K
  — use the train_orm.py path instead
- Training the policy directly with preference data (DPO-style direct alignment) rather
  than a separate reward model
- Full-scale production reward model training where UltraFeedback subsetting to 2000
  samples and 1 epoch is insufficient for convergence assessment
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Implementation language for the train_preference_rm script and logging of
    margin records
- name: PyTorch
  role: Tensor computation and training loop for computing per-batch chosen/rejected
    reward scores
- name: TRL
  role: Training stack used to run the preference reward model epoch
  repo: https://github.com/huggingface/trl
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:759cec2a473a464d8f3baf6366362d1b91b792e941ba76ee3efd3873196b2b3c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reward-margin-convergence-monitoring

## Summary

Trains a Bradley-Terry preference reward model with the rlhf-book companion script on UltraFeedback preference pairs (~2000 samples, 1 epoch) and monitors the chosen-minus-rejected reward margin as the convergence signal for contrastive preference training.

## When to use

Use when training a preference reward model on (prompt, chosen, rejected) pairs and you need to verify that the reward margin between chosen and rejected completions grows during training without divergence.

## When NOT to use

- Training an outcome reward model (ORM) on a binary correctness signal such as GSM8K — use the train_orm.py path instead
- Training the policy directly with preference data (DPO-style direct alignment) rather than a separate reward model
- Full-scale production reward model training where UltraFeedback subsetting to 2000 samples and 1 epoch is insufficient for convergence assessment

## Inputs

- UltraFeedback preference pairs (prompt, chosen completion, rejected completion) from HuggingFace datasets, subsampled to ~2000 examples
- A language model backbone to serve as the reward model (e.g., an AutoModelForSequenceClassification head)
- Training configuration: --samples 2000, --epochs 1

## Outputs

- Trained Bradley-Terry preference reward model checkpoint
- Training log (training_log.l) with per-step records: step index, chosen_rewards, rejected_rewards, loss, margin
- W&B log curves for loss and reward margin

## How to apply

Load UltraFeedback preference pairs (prompt, chosen completion, rejected completion) from HuggingFace datasets and subsample approximately 2000 examples. Instantiate the preference reward model via the train_preference_rm script from the rlhf-book code/reward_models module, using the standard Bradley-Terry contrastive loss (negative log-sigmoid of the chosen-minus-rejected reward margin). Train for 1 epoch with the TRL/PyTorch training stack, computing per-batch chosen and rejected reward scores. At each optimization step, log the step index, chosen_rewards, rejected_rewards, loss, and margin (chosen reward minus rejected reward) to training_log. The rationale is that the contrastive loss drives the chosen reward to separate from the rejected reward, so a growing margin over the epoch, without reward scores diverging, is the expected observable in the demo and W&B logs.

## Related tools

- **Python** (Implementation language for the train_preference_rm script and logging of margin records)
- **PyTorch** (Tensor computation and training loop for computing per-batch chosen/rejected reward scores)
- **TRL** (Training stack used to run the preference reward model epoch) — https://github.com/huggingface/trl

## Examples

```
python -m reward_models.train_preference_rm --samples 2000 --epochs 1
```

## Evaluation signals

- Logged reward margin (chosen reward minus rejected reward) exhibits a growing trend over the 1-epoch run
- Contrastive loss (negative log-sigmoid of the margin) decreases monotonically or near-monotonically over optimization steps
- Chosen reward scores separate upward from rejected reward scores in the demo and W&B logs
- Reward scores do not diverge (no unbounded growth in chosen_rewards or rejected_rewards magnitudes)
- training_log.l contains complete per-step records of step index, chosen_rewards, rejected_rewards, loss, and margin

## Limitations

- The experiment prescribes only ~2000 UltraFeedback samples and 1 epoch, so margin growth is a demo-scale signal, not a converged production reward model
- Margin growth must be checked against divergence: a growing margin is only valid if reward scores themselves remain bounded
- Software version numbers for the training stack are not stated, so exact reproducibility of the margin trajectory is not guaranteed

## Evidence

- [methods] def masked_mean(values: torch.Tensor, mask: torch.Tensor, axis: Optional[int] = None) -> torch.Tensor:: [evidence span withheld — anchor status: not_found]
