---
name: math-answer-verification
description: Use when you have GSM8K-style math completions and need to label each
  completion's final answer as correct or incorrect (answer checking) so the labels
  can train an ORM or demonstrate outcome-level score discrimination.
when_to_use_negative:
- When supervision is step-level (PRM800K) and you need per-step scores across intermediate
  reasoning — train a PRM instead of doing final-answer verification
- When the task text has no extractable final answer (e.g. open-ended instruction
  following), since outcome-level answer checking requires a checkable final answer
- When input is already a labeled correctness dataset — skip re-extraction and go
  straight to ORM training/evaluation
edam_topics:
- http://edamontology.org/topic_3376
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Training framework for the ORM script (e.g. masked_mean-style loss aggregation
    over completions) in the rlhf-book codebase
  repo: https://github.com/natolambert/rlhf-book
- name: TRL
  role: Companion post-training stack used for the process reward model path in the
    ORM-vs-PRM comparison
  repo: https://github.com/huggingface/trl
- name: rlhf-book codebase
  role: Provides code/reward_models/train_orm.py as the starting point for GSM8K-style
    ORM experiments
  repo: https://github.com/natolambert/rlhf-book
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:247b2ef1a823e38b9e41e76f4de55b4fb1d92cc9e7d2a445e22686cd50efc46a
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
  - rlhf-book codebase
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3376
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# math-answer-verification

## Summary

Extracts final-answer correctness labels from GSM8K math word-problem completions by checking extracted answers against ground truth, producing the binary outcome-level supervision used to train and evaluate an outcome reward model (ORM) in the rlhf-book companion codebase.

## When to use

Use when you have GSM8K-style math completions and need to label each completion's final answer as correct or incorrect (answer checking) so the labels can train an ORM or demonstrate outcome-level score discrimination.

## When NOT to use

- When supervision is step-level (PRM800K) and you need per-step scores across intermediate reasoning — train a PRM instead of doing final-answer verification
- When the task text has no extractable final answer (e.g. open-ended instruction following), since outcome-level answer checking requires a checkable final answer
- When input is already a labeled correctness dataset — skip re-extraction and go straight to ORM training/evaluation

## Inputs

- GSM8K math word-problem completions (multi-step solution traces ending in a final answer)
- GSM8K ground-truth final answers for answer checking
- Trained ORM checkpoint (e.g. produced by python -m reward_models.train_orm)

## Outputs

- Binary final-answer correctness labels (correct vs incorrect) per GSM8K completion
- ORM outcome-discrimination score records on held-out completions
- Comparison-table rows with model type, checkpoint path, evaluation granularity (final-answer vs per-step), and score columns

## How to apply

Load GSM8K math word-problem training data and extract the final answer from each model completion via answer checking, following the verification examples in the rlhf-book (ground truth is annotated in the GSM8K `####` field). Assign a binary correctness label per completion by comparing the extracted answer to the gold answer. Start from the companion script `code/reward_models/train_orm.py` in the natolambert/rlhf-book codebase and train the ORM with PyTorch, saving a checkpoint. Then evaluate the ORM checkpoint on held-out GSM8K completions, recording scores for correct vs incorrect final answers to show binary outcome separation — in contrast to a PRM trained on PRM800K step-level labels, which produces per-step score vectors rather than final-answer discrimination.

## Related tools

- **PyTorch** (Training framework for the ORM script (e.g. masked_mean-style loss aggregation over completions) in the rlhf-book codebase) — https://github.com/natolambert/rlhf-book
- **TRL** (Companion post-training stack used for the process reward model path in the ORM-vs-PRM comparison) — https://github.com/huggingface/trl
- **rlhf-book codebase** (Provides code/reward_models/train_orm.py as the starting point for GSM8K-style ORM experiments) — https://github.com/natolambert/rlhf-book

## Examples

```
python -m reward_models.train_orm
```

## Evaluation signals

- Every GSM8K completion receives a non-empty binary label (correct/incorrect) after answer extraction and comparison to the gold answer
- Validation: the ORM checkpoint file exists and loads
- Comparison table contains a non-empty ORM score column separating correct from incorrect final answers
- Evaluation-granularity column correctly distinguishes final-answer scoring (ORM) from per-step scoring (PRM)

## Limitations

- Software version numbers are not stated for named tools (PyTorch, TRL), so exact reproduction of the training stack is under-specified
- The ORM only discriminates final answers; it cannot score intermediate reasoning steps — that requires PRM800K step-level supervision
- Answer checking depends on completions ending in an extractable, comparable final answer format

## Evidence

- [methods] Load the GSM8K math word-problem training data with completions labeled by final-answer correctness (extracted via answer checking as in the book's verification examples).: [evidence span withheld — anchor status: not_found]
- [methods] verify that both checkpoint files exist and load, and that the comparison table contains a non-empty ORM score column separating correct from incorrect final answers: [evidence span withheld — anchor status: not_found]
- [readme] A comprehensive library to post-train foundation models: [evidence span withheld — anchor status: not_found]
