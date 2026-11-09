---
name: objective-aligned-t5-few-shot-workflow
description: 'Use when a T5 encoder-decoder model must be adapted on task-related
  commonsense sentences, fine-tuned from a small ATOMIC2020 sample with an aligned
  masked template, and evaluated against a supervised few-shot baseline.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - omcs-sentence-splitting-adaptation-data-preparation
  - seq2seq-sentence-splitter-training
  - sentence-splitter-training-on-knowledge-graph-tuples
  - atomic2020-knowledge-completion-data-formatting
  - task-category-classification
  - head-tail-pair-classifier-training
  - denoising-objective-adaptation-training
  - unsupervised-adaptation-data-preparation
  - objective-aligned-adaptation-pretraining
  - denoising-vs-lm-objective-alignment
  - t5-encoder-decoder-fine-tuning
  - pretraining-objective-template-alignment
  - few-shot-qa-fine-tuning
  - fine-tuning-template-alignment
  - few-shot-evaluation-of-language-models
  - rouge-bertscore-generation-evaluation
  - commonsense-knowledge-completion-evaluation
  - rouge-generation-metric-computation
  - generation-plausibility-metric-scoring
  member_tools:
  - Ada Factor optimizer
  - T5
  - T5-base
  - T5-large
  - T5-lm
  - T5-v1
  - Match Task to Objective (MTO) framework
  - Sentence Splitter
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/agentic-ai/v1
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Objective-Aligned T5 Adaptation and Few-Shot KB Completion

## Summary

Prepare adaptation and task data, continue denoising pre-training, fine-tune a T5 checkpoint for commonsense knowledge completion, and report held-out few-shot gains.


## When to use

Use when a T5 encoder-decoder model must be adapted on task-related commonsense sentences, fine-tuned from a small ATOMIC2020 sample with an aligned masked template, and evaluated against a supervised few-shot baseline.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — prepare_adaptation_data

**Goal:** turn commonsense text into objective-labeled unsupervised adaptation samples

**EDAM operation:** http://edamontology.org/operation_0282

**Inputs:** tsv/atomic2020-training-tuples, text/omcs-top-8000-ranked-sentences, yaml/adaptation-data-policy · **Outputs:** jsonl/objective-labeled-adaptation-samples

**Candidate leaf skills:** `omcs-sentence-splitting-adaptation-data-preparation` (primary), `seq2seq-sentence-splitter-training`, `sentence-splitter-training-on-knowledge-graph-tuples`

**Tools (primary):** Ada Factor optimizer, T5, T5-base, T5-large, T5-lm, T5-v1

**Other candidate tools:** ATOMIC2020, Match Task to Objective (MTO) framework, Open Mind Common Sense (OMCS) corpus, Sentence Splitter

**Grounding:** 1 KB(s); DOIs: 10.1007/s10489-024-05660-2

### Stage 2 — format_few_shot_examples

**Goal:** format knowledge-graph tuples as aligned few-shot input-target examples

**EDAM operation:** http://edamontology.org/operation_0306

**Inputs:** tsv/atomic2020-few-shot-training-tuples, yaml/relation-template-definitions, yaml/few-shot-sampling-policy · **Outputs:** jsonl/few-shot-kb-completion-examples

**Candidate leaf skills:** `atomic2020-knowledge-completion-data-formatting` (primary), `task-category-classification`, `head-tail-pair-classifier-training`

**Tools (primary):** Ada Factor optimizer, Match Task to Objective (MTO) framework, Sentence Splitter, T5, T5-base, T5-large, T5-lm, T5-v1

**Other candidate tools:** ATOMIC2020

**Grounding:** 1 KB(s); DOIs: 10.1007/s10489-024-05660-2

### Stage 3 — adapt_checkpoint

**Goal:** continue pre-training a T5 checkpoint with a task-aligned denoising objective

**EDAM operation:** http://edamontology.org/operation_3095

**Inputs:** jsonl/objective-labeled-adaptation-samples, model/t5-base-checkpoint, yaml/adaptation-training-configuration · **Outputs:** model/denoising-adapted-t5

**Candidate leaf skills:** `denoising-objective-adaptation-training` (primary), `unsupervised-adaptation-data-preparation`, `objective-aligned-adaptation-pretraining`, `denoising-vs-lm-objective-alignment`

**Tools (primary):** Ada Factor optimizer, Match Task to Objective (MTO) framework, T5, T5-base, T5-lm, T5-v1

**Other candidate tools:** T5-large

**Grounding:** 1 KB(s); DOIs: 10.1007/s10489-024-05660-2

### Stage 4 — finetune_kb_completion

**Goal:** fine-tune the adapted checkpoint on masked few-shot knowledge completion

**EDAM operation:** http://edamontology.org/operation_3093

**Inputs:** model/denoising-adapted-t5, jsonl/few-shot-kb-completion-examples, yaml/fine-tuning-configuration · **Outputs:** model/few-shot-kb-completion-t5

**Candidate leaf skills:** `t5-encoder-decoder-fine-tuning` (primary), `pretraining-objective-template-alignment`, `few-shot-qa-fine-tuning`, `fine-tuning-template-alignment`

**Tools (primary):** Ada Factor optimizer, T5, T5-base, T5-large, T5-lm, T5-v1

**Other candidate tools:** Match Task to Objective (MTO) framework

**Grounding:** 1 KB(s); DOIs: 10.1007/s10489-024-05660-2

### Stage 5 — evaluate_few_shot_strategy

**Goal:** compare the aligned checkpoint with a supervised few-shot baseline

**EDAM operation:** http://edamontology.org/operation_2428

**Inputs:** model/few-shot-kb-completion-t5, model/supervised-mapping-baseline-t5, jsonl/atomic2020-heldout-100-heads-multireference, yaml/evaluation-configuration · **Outputs:** csv/few-shot-kb-completion-metrics

**Candidate leaf skills:** `few-shot-evaluation-of-language-models` (primary), `rouge-bertscore-generation-evaluation`, `commonsense-knowledge-completion-evaluation`, `rouge-generation-metric-computation`, `generation-plausibility-metric-scoring`

**Tools (primary):** Ada Factor optimizer, T5, T5-base, T5-lm, T5-v1

**Other candidate tools:** T5-large, Match Task to Objective (MTO) framework

**Grounding:** 1 KB(s); DOIs: 10.1007/s10489-024-05660-2

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
