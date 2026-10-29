---
name: training-objective-label-classification
description: Use when generating task-related unsupervised adaptation data from natural
  sentences and each split sentence must be tagged as suited to the denoising objective
  (mask-token replacement) or the LM objective (no mask tokens).
when_to_use_negative:
- When the target task category has already been mapped to a single fixed objective
  and no mixed-objective adaptation data is needed (use the objective directly instead).
- When input sentences have not yet been split by the Sentence Splitter into head-like
  and tail-like parts — the classifier operates on split samples.
- 'When evaluating non encoder-decoder architectures: the article notes results are
  specific to the encoder-decoder architecture and alternative architectures were
  not explored.'
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0006
tools:
- name: Match Task to Objective (MTO) framework
  role: Overall framework that uses a classifier to determine the suitable objective
    for each produced sample, generating mixed adaptation data
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder pre-trained model whose pre-training objectives (denoising,
    prefix LM) define the label space assigned to each adaptation sample
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer with a consistent learning rate of 0.0001 used when training the
    models involved in the MTO pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:9765c7fe5d18ac8ef4f9abe69e7c781c6d87527be71735743cb034c811ee5536
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
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0006
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# training-objective-label-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies each produced adaptation sample with the pre-training objective it is suited for (denoising vs. prefix language modeling) within the MTO framework, so that mixed unsupervised adaptation data can combine both objectives in a single training pass.

## When to use

Use when generating task-related unsupervised adaptation data from natural sentences and each split sentence must be tagged as suited to the denoising objective (mask-token replacement) or the LM objective (no mask tokens).

## When NOT to use

- When the target task category has already been mapped to a single fixed objective and no mixed-objective adaptation data is needed (use the objective directly instead).
- When input sentences have not yet been split by the Sentence Splitter into head-like and tail-like parts — the classifier operates on split samples.
- When evaluating non encoder-decoder architectures: the article notes results are specific to the encoder-decoder architecture and alternative architectures were not explored.

## Inputs

- Split sentences produced by the Sentence Splitter (two parts per natural sentence, derived from ~8,000 top-ranked OMCS corpus sentences)
- Trained task category classifier (trained on samples of tasks per category)
- Mask token placeholder used by the denoising objective

## Outputs

- Mixed adaptation data: one record per natural sentence with two split parts and an objective label of 'denoising' or 'LM' (mixed_adaptation_data)

## How to apply

After the T5-based Sentence Splitter has segmented natural sentences (e.g., 8,000 OMCS sentences ranked by relevance score) into head-like and tail-like parts, run a trained task category classifier on each split sample to decide its suitable objective. Samples classified for the denoising objective have their second (target) part replaced by a mask token; samples classified for the LM objective keep the full sentence with no mask tokens. The classifier itself is trained from samples of known task categories, since MTO associates a task category with an appropriate objective. Serialize all records with their two split parts plus an objective label ('denoising' or 'LM') into the Mixed adaptation data artifact, which then supports concurrent unsupervised training on both objectives during the adaptation phase before fine-tuning.

## Related tools

- **Match Task to Objective (MTO) framework** (Overall framework that uses a classifier to determine the suitable objective for each produced sample, generating mixed adaptation data) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder pre-trained model whose pre-training objectives (denoising, prefix LM) define the label space assigned to each adaptation sample) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer with a consistent learning rate of 0.0001 used when training the models involved in the MTO pipeline)

## Evaluation signals

- mixed_adaptation_data exists and contains one record per OMCS sentence (≈8,000 rows).
- Each record has two split parts plus an objective label restricted to 'denoising' or 'LM'.
- Split boundaries form plausible head/tail fragments (checked via llm_judge).
- Denoising-labeled records contain a mask token replacing the second part; LM-labeled records contain no mask tokens.
- The classifier's category-to-objective mapping matches the MTO association of task category with a suitable objective.

## Limitations

- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated.
- Impact of the proposed objective on classification and other task types was not investigated.
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored.

## Evidence

- [methods] we classify a task to certain task categories to identify the appropriate objective for training the model: [evidence span withheld — anchor status: not_found]
