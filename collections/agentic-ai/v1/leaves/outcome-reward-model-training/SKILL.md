---
name: outcome-reward-model-training
description: Use when you have completions with binary final-answer correctness labels
  (e.g., from answer checking on GSM8K) and need a reward model that discriminates
  correct vs incorrect outcomes rather than scoring intermediate reasoning steps.
when_to_use_negative:
- You need per-step scores over intermediate reasoning traces — a process reward model
  trained on step-level labels (e.g., PRM800K) is the right choice instead.
- Your supervision is human preference pairs over completions rather than objective
  final-answer correctness labels.
- The task is not verifiable-outcome math/code (e.g., open-ended chat quality), where
  answer checking cannot produce outcome labels.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3372
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Training framework used to train the ORM and load/save its checkpoint (e.g.,
    masked_mean loss helpers in the book's methods code).
  repo: https://github.com/natolambert/rlhf-book
- name: TRL
  role: Hugging Face post-training library used in the companion TRL/PyTorch training
    stack for reward model training in this workflow.
  repo: https://github.com/huggingface/trl
- name: GSM8K
  role: Math word-problem dataset whose completions, labeled by final-answer correctness,
    form the ORM training signal.
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:1108506d5c09cff62b106d6ac333b78e50103f946f4e7308e24b2b2d3d46fefb
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/natolambert/rlhf-book
  tools:
  - GSM8K
  - PyTorch
  - Python
  - TRL
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3372
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# outcome-reward-model-training

## Summary

Train an outcome reward model (ORM) on GSM8K-style math word-problem completions labeled by final-answer correctness, so the model can later score whether a completion's final answer is right. In the RLHF-book codebase this is done via the companion script reward_models/train_orm.py with PyTorch, and can be contrasted with a PRM trained on PRM800K step-level labels.

## When to use

Use when you have completions with binary final-answer correctness labels (e.g., from answer checking on GSM8K) and need a reward model that discriminates correct vs incorrect outcomes rather than scoring intermediate reasoning steps.

## When NOT to use

- You need per-step scores over intermediate reasoning traces — a process reward model trained on step-level labels (e.g., PRM800K) is the right choice instead.
- Your supervision is human preference pairs over completions rather than objective final-answer correctness labels.
- The task is not verifiable-outcome math/code (e.g., open-ended chat quality), where answer checking cannot produce outcome labels.

## Inputs

- GSM8K math word-problem training data (prompts plus completions)
- Final-answer correctness labels obtained via answer checking on each completion

## Outputs

- Trained ORM checkpoint file (loadable with PyTorch)
- Score records for held-out GSM8K completions separating correct from incorrect final answers
- Comparison table rows with model type, checkpoint path, evaluation granularity, and score records

## How to apply

Load the GSM8K math word-problem training data with completions labeled by final-answer correctness, where labels are extracted via answer checking as in the book's verification examples. Train the outcome reward model with the companion script `python -m reward_models.train_orm` from the natolambert/rlhf-book codebase using PyTorch, and save the trained checkpoint. After training, evaluate the checkpoint on held-out GSM8K completions, recording scores for correct vs incorrect final answers to demonstrate binary outcome separation. Optionally aggregate alongside a PRM's per-step score vectors into a comparison table with columns for model type, checkpoint path, evaluation granularity (final-answer vs per-step), and score records.

## Related tools

- **PyTorch** (Training framework used to train the ORM and load/save its checkpoint (e.g., masked_mean loss helpers in the book's methods code).) — https://github.com/natolambert/rlhf-book
- **TRL** (Hugging Face post-training library used in the companion TRL/PyTorch training stack for reward model training in this workflow.) — https://github.com/huggingface/trl
- **GSM8K** (Math word-problem dataset whose completions, labeled by final-answer correctness, form the ORM training signal.)

## Examples

```
python -m reward_models.train_orm
```

## Evaluation signals

- The trained checkpoint file exists and loads successfully with PyTorch.
- The ORM score column is non-empty and separates correct from incorrect final answers on held-out GSM8K completions (binary outcome discrimination).
- Answer checking of final answers reproduced the correctness labels used for training, as in the book's verification examples.
- In a side-by-side comparison, the ORM yields a single outcome-level score per completion while the PRM yields a non-empty per-step score vector for multi-step traces.

## Limitations

- Outcome-level supervision means the ORM cannot assign scores to intermediate reasoning steps — only the final answer is scored.
- Binary correctness labels require reliable answer checking; ambiguous or unparseable final answers yield noisy outcome labels.
- The article/README do not state software version numbers for the training stack, so exact reproducibility of versions is not pinned down.

## Evidence

- [methods] masked_mean helper: [evidence span withheld — anchor status: not_found]
