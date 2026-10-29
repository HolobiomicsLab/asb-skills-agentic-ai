---
name: majority-vote-task-labeling
description: Use when a new task must be assigned one pre-training objective and you
  have a trained Mask-Filling/Map-Phrasal classifier producing per-sample category
  predictions that need aggregation to a task-level label.
when_to_use_negative:
- The task samples are unlabeled and no trained category classifier exists yet — classifier
  training on the 5,000 labeled ATOMIC2020 samples must precede this skill
- 'You need a many-way objective assignment: this skill only distinguishes the two
  categories (Mask-Filling vs Map-Phrasal) defined by the paper'
edam_operation: http://edamontology.org/operation_0321
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_3474
tools:
- name: Match Task to Objective (MTO) framework
  role: The overall framework in which the classifier-based task categorization, majority
    vote, and objective mapping are performed
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder pre-trained model to which the assigned objectives (denoising
    vs LM) are applied during adaptation and fine-tuning
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:3efbac5ed883f7a3fcdf4bd4cb6a77895c92dc90dae6884a0ae415922fd64bd1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/puraminy/MTO
  repo_url: https://github.com/puraminy/MTO
  tools:
  - Match Task to Objective (MTO) framework
  - T5
  edam_operation: http://edamontology.org/operation_0321
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# majority-vote-task-labeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns each new task a single training-objective category (Mask-Filling vs Map-Phrasal) by majority vote over per-sample predictions from a task-category classifier trained on 5,000 labeled ATOMIC2020 head-tail pairs, then maps the winning category to its pre-training objective (denoising vs language modeling) within the MTO framework.

## When to use

Use when a new task must be assigned one pre-training objective and you have a trained Mask-Filling/Map-Phrasal classifier producing per-sample category predictions that need aggregation to a task-level label.

## When NOT to use

- The task samples are unlabeled and no trained category classifier exists yet — classifier training on the 5,000 labeled ATOMIC2020 samples must precede this skill
- You need a many-way objective assignment: this skill only distinguishes the two categories (Mask-Filling vs Map-Phrasal) defined by the paper

## Inputs

- 5,000 labeled head-tail pairs from selected ATOMIC2020 tasks with Mask-Filling/Map-Phrasal category labels (classifier training set)
- Samples of each new task (head-tail pairs, with relation names converted to natural phrases appended to the head)
- Trained task-category classifier outputting per-sample Mask-Filling vs Map-Phrasal predictions

## Outputs

- One category label (Mask-Filling or Map-Phrasal) per task, chosen by majority vote
- task_objective_assignments mapping each task's category to its training objective (denoising vs LM)

## How to apply

First train a classifier on 5,000 labeled samples drawn from selected ATOMIC2020 tasks (e.g. xIntent, CapableOf, AtLocation as Mask-Filling; Desires, xAttr, isAfter as Map-Phrasal), using head-tail pairs as features and converting relation names to natural phrases appended to the head when needed. Apply this trained classifier to the samples of each new task so that every sample receives one of the two category labels. Select the majority-voted category as the task-level category, guaranteeing each task receives exactly one label. Finally, map each task's category to its training objective — denoising for Mask-Filling tasks and language modeling (LM) for Map-Phrasal tasks — and write the assignments to task_objective_assignments for downstream adaptation and fine-tuning in the MTO framework.

## Related tools

- **Match Task to Objective (MTO) framework** (The overall framework in which the classifier-based task categorization, majority vote, and objective mapping are performed) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder pre-trained model to which the assigned objectives (denoising vs LM) are applied during adaptation and fine-tuning)

## Evaluation signals

- Every task receives exactly one category from the majority vote of its samples (no ties or multi-label outputs)
- Example tasks match reference labels: xIntent -> Mask-Filling, xAttr -> Map-Phrasal
- Mapped objective follows correctly from the category: Mask-Filling -> denoising, Map-Phrasal -> LM
- Classifier training set contains the 5,000 labeled samples with the listed example relations (xIntent, CapableOf, AtLocation, Desires, xAttr, isAfter)
- All samples of a task are classified before the vote; no sample is skipped

## Limitations

- Only two categories (Mask-Filling/Map-Phrasal) are supported, tied to the two objectives (denoising/LM); other objectives or task types (e.g. classification) are not covered per the paper's discussion of future work
- MTO repository code is under refinement and the release was only promised in the paper, so reproduction may require contacting the authors (pouramini at gmail per README)
- Results are specific to encoder-decoder architectures (T5 variants); the labeling scheme has not been validated for other architectures

## Evidence

- [methods] we classify a task to certain task categories to identify the appropriate objective for training the model: [evidence span withheld — anchor status: not_found]
