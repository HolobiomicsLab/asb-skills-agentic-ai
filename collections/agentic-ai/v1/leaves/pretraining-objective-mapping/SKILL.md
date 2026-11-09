---
name: pretraining-objective-mapping
description: Use when a downstream task (e.g. ATOMIC2020 relation tasks, CommonsenseQA,
  Open BookQA) must be assigned an unsupervised pre-training objective before adaptation
  or fine-tuning of an encoder-decoder PLM.
when_to_use_negative:
- The task is already known to belong to a category with a fixed objective and a classifier
  would add no information
- The target model is not an encoder-decoder PLM — the paper states results are specific
  to the encoder-decoder architecture (e.g. T5-base/T5-large)
- No labeled sample tasks are available to train the category classifier and the nature
  of the task cannot be inspected directly
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3376
tools:
- name: Match Task to Objective (MTO) framework
  role: Framework providing the classifier-based procedure for assigning a task to
    its pre-training objective
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: Pre-trained model selected when the classifier assigns the denoising (Mask-Filling)
    objective
- name: T5-lm
  role: T5-v1 derivative additionally trained 100k steps on the prefix LM objective,
    selected for Map-Phrasal tasks
- name: T5-base
  role: Baseline encoder-decoder backbone whose variant is chosen per assigned objective
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:10c24e8a89d53d30b28070fbe6f089b38a67ae11702a8c032c6a80327f746f0a
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
  - Ada Factor optimizer
  - Match Task to Objective (MTO) framework
  - T5
  - T5-base
  - T5-lm
  - T5-v1
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# pretraining-objective-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the Match Task to Objective (MTO) decision procedure from Pouramini & Faili (2024): classify each downstream task into Mask-Filling, Map-Phrasal, or Question-Answering categories via a sample-majority classifier, then map the category to its unsupervised pre-training objective (denoising vs LM) and corresponding T5 variant.

## When to use

Use when a downstream task (e.g. ATOMIC2020 relation tasks, CommonsenseQA, Open BookQA) must be assigned an unsupervised pre-training objective before adaptation or fine-tuning of an encoder-decoder PLM.

## When NOT to use

- The task is already known to belong to a category with a fixed objective and a classifier would add no information
- The target model is not an encoder-decoder PLM — the paper states results are specific to the encoder-decoder architecture (e.g. T5-base/T5-large)
- No labeled sample tasks are available to train the category classifier and the nature of the task cannot be inspected directly

## Inputs

- Head-tail pairs from selected ATOMIC2020 tasks with category labels (Mask-Filling vs Map-Phrasal)
- Samples of new downstream tasks to be classified
- Task category-to-objective mapping table (Mask-Filling→denoising, Map-Phrasal→LM)

## Outputs

- task_objective_assignments: one objective (denoising or LM) per task from the majority vote of its classified samples
- Aligned fine-tuning and prompt-tuning template set (PrePT/PostPT and QA variants) keyed to the selected objective

## How to apply

Load head-tail pairs for sample tasks (e.g. 5,000 labeled samples from ATOMIC2020 tasks such as x Intent, Capable Of, At Location as Mask-Filling; Desires, x Attr, is After as Map-Phrasal), converting relation names to natural phrases appended to the head when needed. Train a classifier on these pairs with their category labels, then apply it to samples of each new task and select the majority-voted category per task. Map categories to objectives: Mask-Filling → denoising (use T5-v1), Map-Phrasal → prefix LM objective (use T5-lm, derived from T5-v1 by 100k extra LM training steps); Question-Answering tasks use the QA templates. Emit the aligned fine-tuning templates (Mapping, Prompting, Masked Mapping, Masked Prompting; QA: Answer Prompting, Masked Answer Prompting, Choice Prompting, Masked Choice Prompting) keyed to the chosen objective.

## Related tools

- **Match Task to Objective (MTO) framework** (Framework providing the classifier-based procedure for assigning a task to its pre-training objective) — https://github.com/puraminy/MTO/
- **T5-v1** (Pre-trained model selected when the classifier assigns the denoising (Mask-Filling) objective)
- **T5-lm** (T5-v1 derivative additionally trained 100k steps on the prefix LM objective, selected for Map-Phrasal tasks)
- **T5-base** (Baseline encoder-decoder backbone whose variant is chosen per assigned objective)

## Evaluation signals

- Every task receives exactly one category from the majority vote over its classified samples
- Reference example tasks recover their known categories: x Intent → Mask-Filling, x Attr → Map-Phrasal
- Category-to-objective mapping matches the paper: Mask-Filling → denoising, Map-Phrasal → LM objective
- Selected model version matches the objective (T5-v1 for denoising, T5-lm for LM)
- Each emitted template's mask/prompt-token structure matches the objective it is assigned to

## Limitations

- Code repository is under refinement; project materials may need to be requested from the authors (pouramini at gmail)
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored
- Impact of the proposed objective on classification and other task types was not investigated
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated

## Evidence

- [methods] we classify a task to certain task categories to identify the appropriate objective for training the model: [evidence span withheld — anchor status: not_found]
