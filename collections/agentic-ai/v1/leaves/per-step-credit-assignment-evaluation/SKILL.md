---
name: per-step-credit-assignment-evaluation
description: 'Use when you need to compare reward model supervision granularities:
  you have math-problem completions with final-answer labels and/or step-level labels,
  and must show whether a reward model can score intermediate reasoning steps versus
  only final outcomes.'
when_to_use_negative:
- Your supervision only labels final answers and your analysis goal is fine-grained
  intermediate-step scoring — an ORM trained on outcome-level GSM8K labels cannot
  produce per-step score vectors
- You need a single aggregate reward signal for RL optimization rather than an evaluation
  comparing supervision granularities — train one reward model instead of running
  the ORM-vs-PRM comparison
- The task is not math reasoning with verifiable step structure (e.g. open-ended preference
  data) — PRM800K-style step labels do not exist
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: PyTorch
  role: Deep learning framework used to train the outcome reward model (train_orm)
    and process reward model (train_prm) checkpoints
  repo: https://github.com/natolambert/rlhf-book
- name: TRL
  role: Training stack used alongside PyTorch to train the process reward model on
    PRM800K step-level labels
  repo: https://github.com/huggingface/trl
- name: GSM8K
  role: Math word-problem dataset providing outcome-level (final-answer correctness)
    training and evaluation completions for the ORM
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:62e115e5717d376e2856373c117502287363c992a1121f6e3f9a9b122304efa7
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Per-step credit assignment evaluation

## Summary

Trains and evaluates process reward models (PRMs) on step-level supervision (PRM800K) alongside outcome reward models (ORMs) on final-answer correctness (GSM8K), producing a comparison table that shows PRMs score each intermediate reasoning step while ORMs only separate correct from incorrect final answers.

## When to use

Use when you need to compare reward model supervision granularities: you have math-problem completions with final-answer labels and/or step-level labels, and must show whether a reward model can score intermediate reasoning steps versus only final outcomes.

## When NOT to use

- Your supervision only labels final answers and your analysis goal is fine-grained intermediate-step scoring — an ORM trained on outcome-level GSM8K labels cannot produce per-step score vectors
- You need a single aggregate reward signal for RL optimization rather than an evaluation comparing supervision granularities — train one reward model instead of running the ORM-vs-PRM comparison
- The task is not math reasoning with verifiable step structure (e.g. open-ended preference data) — PRM800K-style step labels do not exist

## Inputs

- GSM8K math word-problem completions labeled by final-answer correctness (extracted via answer checking)
- PRM800K step-level labels over multi-step solution traces
- Held-out GSM8K completions for evaluation
- Trained ORM and PRM checkpoint files

## Outputs

- Trained ORM checkpoint (final-answer discrimination)
- Trained PRM checkpoint (per-step score vectors)
- Comparison table with columns: model type, checkpoint path, evaluation granularity (final-answer vs per-step), score records

## How to apply

Load GSM8K math word-problem training data with completions labeled by final-answer correctness (answer checking as in the book's verification examples), then train the outcome reward model with the rlhf-book companion script `python -m reward_models.train_orm` using PyTorch. Separately load PRM800K step-level labels and train the process reward model with `python -m reward_models.train_prm` on the TRL/PyTorch stack, saving both checkpoints. Evaluate the ORM on held-out GSM8K completions by recording scores for correct vs incorrect final answers (binary outcome separation), and evaluate the PRM on the same completions by recording its per-step score vector for each multi-step solution trace. Aggregate both into a single comparison table with columns for model type, checkpoint path, evaluation granularity (final-answer vs per-step), and score records; validate that both checkpoint files load and that the ORM score column separates correct from incorrect finals while the PRM per-step column is non-empty.

## Related tools

- **PyTorch** (Deep learning framework used to train the outcome reward model (train_orm) and process reward model (train_prm) checkpoints) — https://github.com/natolambert/rlhf-book
- **TRL** (Training stack used alongside PyTorch to train the process reward model on PRM800K step-level labels) — https://github.com/huggingface/trl
- **GSM8K** (Math word-problem dataset providing outcome-level (final-answer correctness) training and evaluation completions for the ORM)

## Examples

```
python -m reward_models.train_orm && python -m reward_models.train_prm
```

## Evaluation signals

- Both checkpoint files (ORM and PRM) exist and load successfully
- The ORM score column is non-empty and separates correct from incorrect final answers on held-out GSM8K completions (binary outcome discrimination)
- The PRM score column is non-empty and contains a per-step score vector for each multi-step solution trace
- The comparison table contains rows for both model types with explicit evaluation granularity (final-answer vs per-step) and populated score records

## Limitations

- Outcome-level supervision (GSM8K final-answer correctness) only supports binary outcome separation — the ORM cannot score intermediate reasoning steps
- The workflow does not state software version numbers for PyTorch/TRL, so environment reproducibility must be pinned separately by the practitioner
- Per-step scoring depends on the availability of step-level labels (PRM800K); it cannot be applied to completions lacking step segmentation

## Evidence

- [other] After training, the ORM distinguishes correct and incorrect final answers, while the PRM assigns scores across intermediate reasoning steps.: [evidence span withheld — anchor status: not_found]
