---
name: omcs-sentence-splitting-adaptation-data-preparation
description: Use when fine-tuning or adapting a T5 variant for few-shot KB completion
  (e.g., ATOMIC relations, n=30) requires task-related unsupervised adaptation data
  derived from natural-language commonsense sentences like the OMCS corpus.
when_to_use_negative:
- The target task is not a generation/KB-completion style task — the article notes
  the impact of the proposed objective on classification and other task types was
  not investigated
- The downstream architecture is not encoder-decoder — results are specific to the
  encoder-decoder architecture and alternatives were not explored
- Sufficient supervised few-shot data already exists and an unsupervised adaptation
  stage (and its data preparation cost) is unnecessary
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: T5
  role: Encoder-decoder PLM that consumes the split OMCS sentences in the unsupervised
    adaptation stage prior to fine-tuning
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: Pre-trained on C4 with the unsupervised denoising objective; denoising-suitable
    split samples align with its pre-training
  repo: https://github.com/puraminy/MTO/
- name: T5-lm
  role: Derivative of T5-v1 with 100k additional prefix-LM steps; LM-suitable split
    samples align with its pre-training
  repo: https://github.com/puraminy/MTO/
- name: T5-base
  role: Downstream T5 variant fine-tuned with AdaFactor (lr 0.0001, batch size 8)
    on the prepared adaptation data
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer used with a consistent learning rate of 0.0001 throughout fine-tuning
    after adaptation
  repo: https://github.com/puraminy/MTO/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:c9cd2710712d8982206e32d62c71abff0098c813ab2f61166366b8d66f7b3123
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
  - T5
  - T5-base
  - T5-large
  - T5-lm
  - T5-v1
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# omcs-sentence-splitting-adaptation-data-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares task-related unsupervised adaptation data for encoder-decoder PLMs by training a Sentence Splitter on knowledge-graph tuples (e.g., ATOMIC/ATOMIC2020) and applying it to natural OMCS sentences, optionally labeling each split sample with its suitable objective (denoising vs. LM) to form Mixed adaptation data.

## When to use

Use when fine-tuning or adapting a T5 variant for few-shot KB completion (e.g., ATOMIC relations, n=30) requires task-related unsupervised adaptation data derived from natural-language commonsense sentences like the OMCS corpus.

## When NOT to use

- The target task is not a generation/KB-completion style task — the article notes the impact of the proposed objective on classification and other task types was not investigated
- The downstream architecture is not encoder-decoder — results are specific to the encoder-decoder architecture and alternatives were not explored
- Sufficient supervised few-shot data already exists and an unsupervised adaptation stage (and its data preparation cost) is unnecessary

## Inputs

- Knowledge graph (head, relation, tail) tuples from ATOMIC2020 for training the Sentence Splitter
- Open Mind Common Sense (OMCS) corpus sentences (>700,000 natural-language commonsense sentences); top 8,000 scored sentences selected
- T5 checkpoints (T5-v1, T5-lm, T5-base, T5-large) to receive the adaptation data

## Outputs

- Split natural sentences serving as task-related unsupervised adaptation data
- Mixed adaptation set with per-sample objective labels (denoising-suitable vs. LM-suitable)
- Adapted T5 checkpoints ready for fine-tuning on few-shot KB completion templates (Mapping, Prompting, Masked Mapping, Masked Prompting)

## How to apply

First train a Sentence Splitter model on (head, relation, tail) tuples extracted from the knowledge graph (ATOMIC2020 relations formatted with the Table A1 templates). Then split the top 8,000 scored OMCS sentences with the trained Sentence Splitter to produce natural sentences suitable for unsupervised training. Optionally train/apply a classifier to label each produced sample as denoising-suitable or LM-suitable, yielding Mixed adaptation data. Use this data in an unsupervised adaptation stage (e.g., Denoising, LM, or Mixed objective) before task fine-tuning with the AdaFactor optimizer (learning rate 0.0001, batch size 8 for T5-base / 4 for T5-large) for 3 epochs. Model-generated completions can form new natural sentences, bootstrapping the unsupervised data.

## Related tools

- **T5** (Encoder-decoder PLM that consumes the split OMCS sentences in the unsupervised adaptation stage prior to fine-tuning) — https://github.com/puraminy/MTO/
- **T5-v1** (Pre-trained on C4 with the unsupervised denoising objective; denoising-suitable split samples align with its pre-training) — https://github.com/puraminy/MTO/
- **T5-lm** (Derivative of T5-v1 with 100k additional prefix-LM steps; LM-suitable split samples align with its pre-training) — https://github.com/puraminy/MTO/
- **T5-base** (Downstream T5 variant fine-tuned with AdaFactor (lr 0.0001, batch size 8) on the prepared adaptation data) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer used with a consistent learning rate of 0.0001 throughout fine-tuning after adaptation) — https://github.com/puraminy/MTO/

## Evaluation signals

- The Sentence Splitter must be trained on knowledge-graph tuples and, when applied to OMCS sentences, produce samples each assignable to a suitable objective (denoising or LM) for the Mixed configuration
- Adaptation uses the top 8,000 scored OMCS sentences; counts and per-sample objective labels should be auditable
- Downstream validation: reproduced ROUGE/BERTScore values per model/objective/template combination should match Tables B3/B4, with n=30 samples over 3 epochs
- Expected best Mask-Filling configuration for the full pipeline is T5-large + Mixed + Masked Prompting
- Few-shot experiments repeated three times with mean scores reported

## Limitations

- Code is under refinement and improvement; full project materials may need to be requested from the authors (pouramini at gmail)
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated
- Impact of the proposed objective on classification and other task types not investigated

## Evidence

- [methods] Apply splitter to natural sentences: [evidence span withheld — anchor status: not_found]
- [methods] Mixed objective data preparation from OMCS: [evidence span withheld — anchor status: not_found]
