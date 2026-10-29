---
name: head-tail-pair-classifier-training
description: Use when a new ATOMIC2020-style task has no manually assigned category
  and must be mapped to a pre-training objective (denoising vs LM) via a classifier
  trained on head-tail pairs of reference tasks.
when_to_use_negative:
- The task already has a manually assigned category from Table A2 — use the reference
  label directly instead of the classifier.
- Input is not head-tail pair data from a commonsense KB such as ATOMIC2020 (e.g.
  multiple-choice QA like CommonsenseQA/Open BookQA), since the two-category head-tail
  scheme does not apply.
- Only one pre-training objective (e.g. a T5-v1 checkpoint trained solely with the
  denoising objective on C4) is available, so category-driven model selection is moot.
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: Match Task to Objective (MTO) framework
  role: The framework within which the classifier assigns tasks to training objectives
    and selects the suitable objective and pre-trained model
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder model for which the classifier's category output determines
    the fine-tuning/adaptation objective (denoising vs LM); T5-v1, T5-lm, T5-base
    and T5-large variants are used downstream
  repo: https://github.com/puraminy/MTO/
- name: ATOMIC2020
  role: Source of the 5,000 labeled head-tail sample pairs used to train the category
    classifier
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:93aa1e6086eb5083b8d54c28d2aa0662885334c31e67c4936316e3df830ca06d
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
  - Match Task to Objective (MTO) framework
  - T5
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# head-tail-pair-classifier-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a classifier on labeled head-tail pairs drawn from reference ATOMIC2020 tasks so that new tasks can be automatically categorized as Mask-Filling vs Map-Phrasal within the Match Task to Objective (MTO) framework, and hence mapped to a denoising or language-modeling pre-training objective for T5.

## When to use

Use when a new ATOMIC2020-style task has no manually assigned category and must be mapped to a pre-training objective (denoising vs LM) via a classifier trained on head-tail pairs of reference tasks.

## When NOT to use

- The task already has a manually assigned category from Table A2 — use the reference label directly instead of the classifier.
- Input is not head-tail pair data from a commonsense KB such as ATOMIC2020 (e.g. multiple-choice QA like CommonsenseQA/Open BookQA), since the two-category head-tail scheme does not apply.
- Only one pre-training objective (e.g. a T5-v1 checkpoint trained solely with the denoising objective on C4) is available, so category-driven model selection is moot.

## Inputs

- 5,000 labeled samples drawn from selected ATOMIC2020 tasks (e.g. x Intent, Capable Of, At Location; Desires, x Attr, is After) with assigned Mask-Filling/Map-Phrasal category labels
- Head-tail pairs from those tasks, with relation names converted to natural phrases appended to the head
- Table A2 reference labels mapping example tasks to Mask-Filling vs Map-Phrasal categories

## Outputs

- Trained task-category classifier (Mask-Filling vs Map-Phrasal)
- Per-task majority-voted category assignment
- task_objective_assignments mapping each task to denoising (Mask-Filling) or LM (Map-Phrasal)

## How to apply

Load 5,000 labeled samples from selected ATOMIC2020 tasks whose categories are known from Table A2 — e.g. x Intent, Capable Of, At Location as Mask-Filling and Desires, x Attr, is After as Map-Phrasal — using their head-tail pairs as features. Convert relation names to natural phrases appended to the head when needed so the classifier sees task-style text. Train the classifier on these pairs using their two-category labels (Mask-Filling vs Map-Phrasal); the rationale is that article-matched categories indicate which unsupervised objective suits the task type. Apply the trained classifier per-sample to each new task and take the majority vote over its samples as the task's single category. Map the winning category to its objective — Mask-Filling → denoising, Map-Phrasal → language modeling (LM) — and write the result to task_objective_assignments for use by the MTO adaptation stage.

## Related tools

- **Match Task to Objective (MTO) framework** (The framework within which the classifier assigns tasks to training objectives and selects the suitable objective and pre-trained model) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder model for which the classifier's category output determines the fine-tuning/adaptation objective (denoising vs LM); T5-v1, T5-lm, T5-base and T5-large variants are used downstream) — https://github.com/puraminy/MTO/
- **ATOMIC2020** (Source of the 5,000 labeled head-tail sample pairs used to train the category classifier)

## Examples

```
git clone https://github.com/puraminy/MTO/
```

## Evaluation signals

- Every task receives exactly one category from the majority vote over its samples (no ties or split assignments).
- Classifier output matches Table A2 reference labels on held example tasks: x Intent → Mask-Filling and x Attr → Map-Phrasal.
- Category-to-objective mapping is consistent: Mask-Filling tasks are assigned the denoising objective and Map-Phrasal tasks the language modeling (LM) objective.
- The 5,000-sample training set is drawn only from the selected ATOMIC2020 tasks listed for each category (x Intent, Capable Of, At Location vs Desires, x Attr, is After), with relation names rendered as natural phrases.
- task_objective_assignments contains an objective entry for every classified task.

## Limitations

- Only two task categories/objectives are supported (Mask-Filling → denoising; Map-Phrasal → LM); other objective types are not covered.
- Results are specific to the encoder-decoder (T5) architecture; alternative architectures are not explored.
- Code availability is limited: the README states the code 'is under refinement and improvement' and the article only promises a future public release at https://github.com/puraminy/MTO.

## Evidence
