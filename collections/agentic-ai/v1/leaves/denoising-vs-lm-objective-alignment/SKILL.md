---
name: denoising-vs-lm-objective-alignment
description: Use when few-shot fine-tuning an encoder-decoder PLM (T5 family) on question
  answering or generation and you must choose which unsupervised adaptation objective
  and which fine-tuning template (e.g., Masked Choice Prompting) to pair.
when_to_use_negative:
- Task type is classification or another non-QA/non-generation category — the article
  notes the impact of the proposed objective on classification and other task types
  was not investigated.
- Model is not encoder-decoder — the discussion states results are specific to the
  encoder-decoder architecture and alternatives were not explored.
- You need scaling behavior across model sizes or varying amounts of adaptation data
  — the article flags this as un-investigated.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: Match Task to Objective (MTO) framework
  role: Classifies the task and selects the matching pre-training/adaptation objective
    and PLM; provides automated task-related adaptation data preparation
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: Denoising-only pre-trained checkpoint (C4) used to isolate denoising alignment
    effects
- name: T5-lm
  role: Checkpoint additionally trained 100k steps on the prefix LM objective, used
    to test LM-objective alignment
- name: T5-base
  role: Standard baseline checkpoint fine-tuned with lr=0.0001 and mini-batch size
    8
- name: T5-large
  role: Higher-capacity counterpart fine-tuned with mini-batch size 4
- name: Ada Factor optimizer
  role: Optimizer with consistent learning rate 0.0001 across all fine-tuning runs
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:65a1e1473288875f454b092f97868d88ad4caa9dbeed05d39e5dba36e02fb03c
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
  - T5-large
  - T5-lm
  - T5-v1
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# denoising-vs-lm-objective-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A decision-and-implementation recipe from the MTO framework for aligning fine-tuning templates and adaptation-stage objectives (denoising vs. prefix LM vs. Mixed) with the target task on T5 encoder-decoder models, which yields >120% few-shot gains over conventional fine-tuning on QA tasks like CommonsenseQA and OpenBookQA.

## When to use

Use when few-shot fine-tuning an encoder-decoder PLM (T5 family) on question answering or generation and you must choose which unsupervised adaptation objective and which fine-tuning template (e.g., Masked Choice Prompting) to pair.

## When NOT to use

- Task type is classification or another non-QA/non-generation category — the article notes the impact of the proposed objective on classification and other task types was not investigated.
- Model is not encoder-decoder — the discussion states results are specific to the encoder-decoder architecture and alternatives were not explored.
- You need scaling behavior across model sizes or varying amounts of adaptation data — the article flags this as un-investigated.

## Inputs

- Few-shot training splits (n=30) from CommonsenseQA or Open BookQA for 3 random seeds
- T5 checkpoints (T5-v1, T5-lm, T5-base, T5-large)
- Task-related natural sentences for adaptation (e.g., OMCS corpus sentences, ATOMIC2020-derived sentences)
- Mixed adaptation data records with two split parts and an objective label ('denoising' or 'LM')

## Outputs

- Accuracy (mean over 3 seeds, with variance) for each adaptation-condition x template combination
- A Table B5-style results CSV ranking objectives and templates
- Identification of the best-performing template per task (expected: Masked Choice Prompting with adaptation)

## How to apply

First select the objective matched to the task category using the MTO framework: classification of the task maps it to a suitable pre-training objective (denoising, prefix LM, or Mixed). Before supervised fine-tuning, run the adaptation stage — unsupervised training on task-related natural sentences (e.g., OMCS sentences processed into Mixed data with per-sample objective labels) — using the chosen objective. Then fine-tune with the AdaFactor optimizer at learning rate 0.0001 (mini-batch 8 for T5-base, 4 for T5-large, 3 epochs), choosing a template aligned with that objective: Answer Prompting/Masked Answer Prompting and Choice Prompting/Masked Choice Prompting from Table 4. For few-shot QA (FS n=30), Masked Choice Prompting (MTO-MCP) is expected to win, and predicting the choice number alongside the choice text works especially well on CommonsenseQA with a denoising adaptation objective. Repeat over 3 random seeds, report mean (and standard deviation) accuracy, and compare against the Table B5 ordering.

## Related tools

- **Match Task to Objective (MTO) framework** (Classifies the task and selects the matching pre-training/adaptation objective and PLM; provides automated task-related adaptation data preparation) — https://github.com/puraminy/MTO/
- **T5-v1** (Denoising-only pre-trained checkpoint (C4) used to isolate denoising alignment effects)
- **T5-lm** (Checkpoint additionally trained 100k steps on the prefix LM objective, used to test LM-objective alignment)
- **T5-base** (Standard baseline checkpoint fine-tuned with lr=0.0001 and mini-batch size 8)
- **T5-large** (Higher-capacity counterpart fine-tuned with mini-batch size 4)
- **Ada Factor optimizer** (Optimizer with consistent learning rate 0.0001 across all fine-tuning runs)

## Evaluation signals

- The reproduced mean-accuracy ordering across adaptation conditions x templates matches the Table B5 trend.
- Masked Choice Prompting (MTO-MCP) under adaptation ranks highest in few-shot QA across seeds.
- Predicting the choice number with the choice text yields the strongest CommonsenseQA result under a denoising adaptation objective.
- All runs use AdaFactor with lr 0.0001, mini-batch 8 (T5-base) or 4 (T5-large), 3 epochs, FS n=30 over 3 seeds — deviations invalidate comparability.
- Few-shot performance improvement exceeds the reported >120% gain over conventional methods when strategies are applied correctly.

## Limitations

- Results are specific to the encoder-decoder architecture; alternative architectures were not explored.
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated.
- Impact of the proposed objective on classification and other task types was not investigated.
- Random-seed variance for few-shot QA was limited (three repeats, mean reported), limiting statistical power.
- Code availability is uncertain: the README says code is 'under refinement and improvement' and asks users to contact the author for project materials.

## Evidence
