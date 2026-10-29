---
name: reward-model-checkpoint-evaluation
description: Use when you have trained reward-model checkpoints (outcome-level on
  GSM8K completions, process-level on PRM800K step labels) and need to verify and
  compare what each can score — final answers vs intermediate steps.
when_to_use_negative:
- You need to score intermediate reasoning steps but only hold an ORM checkpoint trained
  on outcome-level final-answer correctness — the ORM cannot produce per-step scores
- You have no held-out completions with verified final-answer correctness, so ORM
  discrimination of correct vs incorrect final answers cannot be established
- The checkpoints are instruction-tuned or policy models rather than reward models
  — the score-discrimination and per-step-vector signals do not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: PyTorch
  role: Framework for loading and running the trained ORM/PRM checkpoints and recording
    score tensors
- name: TRL
  role: Training stack used for the process reward model (PRM800K) whose checkpoint
    is evaluated
  repo: https://github.com/huggingface/trl
- name: Python
  role: Execution environment; evaluation driven by module invocations like `python
    -m reward_models.train_orm` from the rlhf-book codebase
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:4f877bfecc7d229d0095b0a55b7cbce1d6c15c655f181af8985b58ce02e46870
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Reward model checkpoint evaluation (ORM vs PRM)

## Summary

Evaluates trained reward-model checkpoints on held-out GSM8K completions by contrasting evaluation granularity: an ORM trained on outcome-level final-answer correctness is scored on binary discrimination of correct vs incorrect final answers, while a PRM trained on step-level PRM800K labels is scored per reasoning step.

## When to use

Use when you have trained reward-model checkpoints (outcome-level on GSM8K completions, process-level on PRM800K step labels) and need to verify and compare what each can score — final answers vs intermediate steps.

## When NOT to use

- You need to score intermediate reasoning steps but only hold an ORM checkpoint trained on outcome-level final-answer correctness — the ORM cannot produce per-step scores
- You have no held-out completions with verified final-answer correctness, so ORM discrimination of correct vs incorrect final answers cannot be established
- The checkpoints are instruction-tuned or policy models rather than reward models — the score-discrimination and per-step-vector signals do not apply

## Inputs

- Trained ORM checkpoint (PyTorch, trained on GSM8K completions labeled by final-answer correctness)
- Trained PRM checkpoint (TRL/PyTorch, trained on PRM800K step-level labels)
- Held-out GSM8K completions with final-answer correctness labels (via answer checking)
- Multi-step solution traces for per-step PRM scoring

## Outputs

- ORM score records separating correct vs incorrect final answers on held-out GSM8K completions
- PRM per-step score vectors for each multi-step solution trace
- Single comparison table: model type, checkpoint path, evaluation granularity (final-answer vs per-step), score records

## How to apply

Load each saved checkpoint from the rlhf-book codebase (ORM from `code/reward_models/train_orm.py`, PRM via the TRL/PyTorch stack) and verify both files exist and load before scoring. Evaluate the ORM on held-out GSM8K completions, extracting final answers via answer checking and recording scores for correct vs incorrect final answers to demonstrate binary outcome separation. Evaluate the PRM on the same completions, recording its per-step score vector for each multi-step solution trace. Aggregate both into a single comparison table with columns for model type, checkpoint path, evaluation granularity (final-answer vs per-step), and score records, so the supervision signal (outcome-level vs step-level) maps directly to what each model can score.

## Related tools

- **PyTorch** (Framework for loading and running the trained ORM/PRM checkpoints and recording score tensors)
- **TRL** (Training stack used for the process reward model (PRM800K) whose checkpoint is evaluated) — https://github.com/huggingface/trl
- **Python** (Execution environment; evaluation driven by module invocations like `python -m reward_models.train_orm` from the rlhf-book codebase)

## Examples

```
python -m reward_models.train_orm  # then evaluate saved ORM and PRM checkpoints on held-out GSM8K completions
```

## Evaluation signals

- Both checkpoint files exist and load successfully before scoring
- The comparison table contains a non-empty ORM score column that separates correct from incorrect final answers
- The comparison table contains a non-empty PRM per-step score column (a score vector per multi-step trace, not a single scalar)
- The table exposes the granularity column distinguishing final-answer scoring (ORM) from per-step scoring (PRM)
- Each row records a valid checkpoint path matching the saved model files

## Limitations

- The ORM only discriminates final-answer correctness; it provides no signal on intermediate reasoning quality, and vice versa for the PRM on outcome-level judgments
- No software version numbers are stated for PyTorch/TRL in the source, so exact reproducibility of scores is not guaranteed
- Final-answer correctness depends on the answer-checking extraction step; errors there propagate into the ORM's correctness labels

## Evidence
