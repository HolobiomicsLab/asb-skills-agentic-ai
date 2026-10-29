---
name: task-category-classification
description: Use when a new downstream task must be mapped to a pre-training objective
  (denoising vs LM) in the MTO pipeline and labeled head-tail sample tasks are available
  for classifier training.
when_to_use_negative:
- The task is a question-answering task handled through the QA-specific templates
  (Answer Prompting, Choice Prompting and masked variants) rather than the Mask-Filling/Map-Phrasal
  dichotomy
- Task-related unlabeled data has already been produced and only needs per-sample
  objective labeling via the optional classifier step on Sentence Splitter output,
  not full task-level classification
- The target architecture is not an encoder-decoder PLM; the paper's results are specific
  to encoder-decoder models such as T5
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Match Task to Objective (MTO) framework
  role: Framework within which the task category classifier is trained and applied
    to assign tasks to objectives
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder pre-trained language model whose pre-training objective (denoising
    vs LM) is selected based on the classifier's category assignment
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: T5 variant pre-trained with the unsupervised denoising objective, selected
    for Mask-Filling tasks
  repo: https://github.com/puraminy/MTO/
- name: T5-lm
  role: T5-v1 derivative given 100k additional prefix-LM training steps, selected
    for Map-Phrasal tasks
  repo: https://github.com/puraminy/MTO/
- name: ATOMIC2020
  role: Source of labeled task samples (e.g. x Intent, x Attr, Desires) used to train
    the task category classifier
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:f6632817f77405a6c96481d6859f06a30d1c34d1733a0fe8a215485c2107a505
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
  - ATOMIC2020
  - Ada Factor optimizer
  - Match Task to Objective (MTO) framework
  - T5
  - T5-base
  - T5-lm
  - T5-v1
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# task-category-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains and applies a task-category classifier within the Match Task to Objective (MTO) framework so that each downstream task is assigned to a category (Mask-Filling or Map-Phrasal) that determines its unsupervised pre-training objective.

## When to use

Use when a new downstream task must be mapped to a pre-training objective (denoising vs LM) in the MTO pipeline and labeled head-tail sample tasks are available for classifier training.

## When NOT to use

- The task is a question-answering task handled through the QA-specific templates (Answer Prompting, Choice Prompting and masked variants) rather than the Mask-Filling/Map-Phrasal dichotomy
- Task-related unlabeled data has already been produced and only needs per-sample objective labeling via the optional classifier step on Sentence Splitter output, not full task-level classification
- The target architecture is not an encoder-decoder PLM; the paper's results are specific to encoder-decoder models such as T5

## Inputs

- 5,000 labeled head-tail pair samples from selected ATOMIC2020 tasks with category labels (Mask-Filling vs Map-Phrasal) per Table A2
- Head-tail pair samples of each new downstream task to be categorized
- Relation names from the knowledge graph (converted to natural phrases appended to the head)

## Outputs

- Per-sample category predictions (Mask-Filling or Map-Phrasal) for each new task
- Majority-voted task category per task
- Objective assignment per task (denoising vs LM) written to task_objective_assignments

## How to apply

Draw 5,000 labeled samples from selected ATOMIC2020 tasks per Table A2 (e.g. x Intent, Capable Of, At Location as Mask-Filling; Desires, x Attr, is After as Map-Phrasal), converting relation names to natural phrases appended to the head when needed. Train a classifier on the head-tail pairs using these category labels, then apply it to every sample of the new task so each sample receives one of the two categories. Select the category chosen by the majority of samples as the task's category. Map the category to its objective — denoising for Mask-Filling and language-model (LM) for Map-Phrasal — and select the matching T5 version (T5-v1 for denoising, T5-lm for the LM objective). Write the assignments (e.g. to task_objective_assignments) and use them downstream to configure adaptation-stage unsupervised pre-training with the AdaFactor optimizer at learning rate 0.0001.

## Related tools

- **Match Task to Objective (MTO) framework** (Framework within which the task category classifier is trained and applied to assign tasks to objectives) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder pre-trained language model whose pre-training objective (denoising vs LM) is selected based on the classifier's category assignment) — https://github.com/puraminy/MTO/
- **T5-v1** (T5 variant pre-trained with the unsupervised denoising objective, selected for Mask-Filling tasks) — https://github.com/puraminy/MTO/
- **T5-lm** (T5-v1 derivative given 100k additional prefix-LM training steps, selected for Map-Phrasal tasks) — https://github.com/puraminy/MTO/
- **ATOMIC2020** (Source of labeled task samples (e.g. x Intent, x Attr, Desires) used to train the task category classifier)

## Evaluation signals

- Every task receives exactly one category, determined by majority vote over its classified samples
- Reference example tasks classify correctly: x Intent -> Mask-Filling and x Attr -> Map-Phrasal
- Category-to-objective mapping is consistent: Mask-Filling -> denoising (T5-v1) and Map-Phrasal -> LM (T5-lm)
- Written task_objective_assignments contain one objective per task with no missing assignments
- Relation names used in training samples are converted to natural phrases appended to the head, matching the Table A2 task-category pairings

## Limitations

- Only two categories (Mask-Filling and Map-Phrasal) plus a QA path are covered; impact of the proposed objective on classification and other task types was not investigated
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated
- The code repository is under refinement; availability of project materials may require contacting the authors

## Evidence

- [other] The MTO framework determines the objective suitable for a task either by considering the nature of the task itself or by using a classifier specifically designed to predict the appropriate objective.: [evidence span withheld — anchor status: not_found]
