---
name: seq2seq-sentence-splitter-training
description: 'Use when preparing task-related adaptation data from a knowledge graph:
  natural sentences must be split into head-like and tail-like parts to build denoising
  or LM training samples.'
when_to_use_negative:
- The source corpus is already structured as (input, target) pairs and no sentence
  segmentation into head/tail fragments is needed
- The task belongs to a category whose objective requires no task-related unsupervised
  adaptation data (no Sentence Splitter stage in the MTO pipeline)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3360
tools:
- name: Match Task to Objective (MTO) framework
  role: Overall framework within which the Sentence Splitter is trained on knowledge
    graph tuples to prepare unsupervised adaptation data
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder architecture used as the base model for the seq2seq Sentence
    Splitter
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer with a consistent learning rate of 0.0001 used during model training
- name: ATOMIC2020
  role: Source of knowledge graph (head, relation, tail) tuples used to train the
    Sentence Splitter
- name: Open Mind Common Sense (OMCS) corpus
  role: Source of natural sentences (>700,000 sentences) to which the trained splitter
    is applied
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:bc13c52b8dafcfe77d34462cdd3d906adafe98f4aa108aa351f8a4939fcab8ff
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
  - Open Mind Common Sense (OMCS) corpus
  - T5
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# seq2seq-sentence-splitter-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a T5-based seq2seq Sentence Splitter model on (head, relation, tail) tuples from a knowledge graph, converting tuples into natural sentences via templates and learning to segment natural sentences into head-like and tail-like parts for unsupervised adaptation data generation in the MTO framework.

## When to use

Use when preparing task-related adaptation data from a knowledge graph: natural sentences must be split into head-like and tail-like parts to build denoising or LM training samples.

## When NOT to use

- The source corpus is already structured as (input, target) pairs and no sentence segmentation into head/tail fragments is needed
- The task belongs to a category whose objective requires no task-related unsupervised adaptation data (no Sentence Splitter stage in the MTO pipeline)

## Inputs

- 5,000 (head, relation, tail) tuple samples from selected ATOMIC2020 tasks
- Natural-language sentences to be split (e.g., top 8,000 OMCS corpus sentences ranked by relevance score)
- Knowledge graph relation templates with the tail as target

## Outputs

- Trained T5-based seq2seq Sentence Splitter model
- Per-sentence two-part splits (head-like part, tail-like part) for OMCS sentences
- Mixed adaptation data records with split parts and objective labels ('denoising' or 'LM')

## How to apply

Convert knowledge graph (head, relation, tail) tuples — e.g., 5,000 samples from selected ATOMIC2020 tasks — into natural sentences using the paper's templates with the tail as target. Train a seq2seq Sentence Splitter model (T5-based, optimized with AdaFactor at learning rate 0.0001) to map each natural sentence to its segmentation into a head-like part and a tail-like part. Apply the trained splitter to natural sentences such as the top 8,000 relevance-ranked OMCS corpus sentences, producing two parts per sentence with the second part designated as the target. The rationale is that excluding words or phrases from sentences enables concurrent unsupervised training on denoising and LM objectives; downstream, a task category classifier labels each split sample for the denoising objective (second part replaced by a mask token) or the LM objective (no mask tokens), yielding the Mixed data.

## Related tools

- **Match Task to Objective (MTO) framework** (Overall framework within which the Sentence Splitter is trained on knowledge graph tuples to prepare unsupervised adaptation data) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder architecture used as the base model for the seq2seq Sentence Splitter) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer with a consistent learning rate of 0.0001 used during model training)
- **ATOMIC2020** (Source of knowledge graph (head, relation, tail) tuples used to train the Sentence Splitter)
- **Open Mind Common Sense (OMCS) corpus** (Source of natural sentences (>700,000 sentences) to which the trained splitter is applied)

## Evaluation signals

- mixed_adaptation_data file exists and contains one record per OMCS sentence (~8,000 rows)
- Each record has exactly two split parts plus an objective label of 'denoising' or 'LM'
- Split boundaries form plausible head/tail fragments (verified via llm_judge)
- The trained splitter correctly reconstructs segmentations of the templated ATOMIC2020 training sentences (second part equals the tail used as target)

## Limitations

- Code is under refinement and improvement; project materials may not be fully available (README instructs contacting the author)
- The approach is specific to encoder-decoder architectures; results do not transfer to alternative architectures
- Sentence splitting quality depends on the templates used to convert tuples to natural sentences; noisy splits propagate into the Mixed adaptation data

## Evidence

- [methods] This model is then applied to natural sentences to split them and provide the necessary data for our unsupervised training.: [evidence span withheld — anchor status: not_found]
