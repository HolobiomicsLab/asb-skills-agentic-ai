---
name: few-shot-qa-fine-tuning
description: Use when reproducing or comparing few-shot QA accuracy for T5 models
  on CommonsenseQA or OpenBookQA across adaptation objectives (Denoising, LM, Mixed)
  and prompting templates.
when_to_use_negative:
- Full-dataset (non-few-shot) fine-tuning scenarios where the few-shot n=30 sampling
  and alignment strategy are not the target
- 'Non-encoder-decoder architectures: the article''s results are specific to the encoder-decoder
  architecture and alternative architectures were not explored'
- Classification or other task types whose impact under the proposed objective was
  not investigated in this work
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3376
tools:
- name: Match Task to Objective (MTO) framework
  role: Framework used to select the pre-training/adaptation objective matched to
    the QA task before fine-tuning with aligned templates
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder pre-trained language model family fine-tuned on the few-shot
    QA samples
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: T5 checkpoint pre-trained exclusively on C4 with a denoising objective; tests
    denoising-alignment hypotheses
- name: T5-lm
  role: T5-v1 derivative given 100k additional prefix-LM objective steps; tests LM-alignment
    hypotheses
- name: T5-base
  role: Widely adopted downstream model used in few-shot fine-tuning runs with mini-batch
    size 8
- name: T5-large
  role: Higher-capacity counterpart used in few-shot fine-tuning runs with mini-batch
    size 4
- name: Ada Factor optimizer
  role: Optimizer for all fine-tuning at a consistent learning rate of 0.0001
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:49e7dc0d0c89dd43e45c1a5c55b3240c29590e87ea096636a52dd3a1f1443c75
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
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# few-shot-qa-fine-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tunes T5-family encoder-decoder models on n=30 few-shot question-answering samples (CommonsenseQA, Open BookQA) using MTO templates that align the fine-tuning input format with the unsupervised pre-training/adaptation objective, with Masked Choice Prompting (MCP) expected to rank highest.

## When to use

Use when reproducing or comparing few-shot QA accuracy for T5 models on CommonsenseQA or OpenBookQA across adaptation objectives (Denoising, LM, Mixed) and prompting templates.

## When NOT to use

- Full-dataset (non-few-shot) fine-tuning scenarios where the few-shot n=30 sampling and alignment strategy are not the target
- Non-encoder-decoder architectures: the article's results are specific to the encoder-decoder architecture and alternative architectures were not explored
- Classification or other task types whose impact under the proposed objective was not investigated in this work

## Inputs

- Official CommonsenseQA train/validation/test splits
- Official Open BookQA train/validation/test splits
- Few-shot training samples (n=30) per random seed
- T5 checkpoints (T5-v1, T5-lm, T5-base, T5-large), with or without adaptation-stage training

## Outputs

- Per-configuration accuracy on the QA test split
- Mean and standard deviation accuracy across 3 seeds for every adaptation-condition x template combination
- Table B5-style CSV of few-shot QA results with the best-performing template identified (expected: Masked Choice Prompting with adaptation)

## How to apply

Load the official CommonsenseQA and Open BookQA splits and sample n=30 few-shot training examples for each of 3 random seeds. Prepare T5 checkpoints (T5-v1, T5-lm, T5-base, T5-large) and optionally apply the adaptation stage with Denoising, LM, or Mixed unsupervised objectives per the MTO framework. Fine-tune each model on the few-shot samples with the AdaFactor optimizer (learning rate 0.0001, mini-batch size 8 for T5-base / 4 for T5-large, 3 epochs) using the four QA templates: Answer Prompting, Masked Answer Prompting, Choice Prompting, and Masked Choice Prompting. Evaluate accuracy on the QA test split, repeat over 3 seeds, and aggregate mean and standard deviation per adaptation-condition x template combination into a Table B5-style CSV. The rationale is that a fine-tuning methodology closely aligned with the unsupervised objective enhances few-shot performance; predicting the choice number alongside choice text under a denoising adaptation objective is particularly effective on CommonsenseQA.

## Related tools

- **Match Task to Objective (MTO) framework** (Framework used to select the pre-training/adaptation objective matched to the QA task before fine-tuning with aligned templates) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder pre-trained language model family fine-tuned on the few-shot QA samples) — https://github.com/puraminy/MTO/
- **T5-v1** (T5 checkpoint pre-trained exclusively on C4 with a denoising objective; tests denoising-alignment hypotheses)
- **T5-lm** (T5-v1 derivative given 100k additional prefix-LM objective steps; tests LM-alignment hypotheses)
- **T5-base** (Widely adopted downstream model used in few-shot fine-tuning runs with mini-batch size 8)
- **T5-large** (Higher-capacity counterpart used in few-shot fine-tuning runs with mini-batch size 4)
- **Ada Factor optimizer** (Optimizer for all fine-tuning at a consistent learning rate of 0.0001)

## Examples

```
git clone https://github.com/puraminy/MTO/ && python train.py --task commonsenseqa --few_shot 30 --template masked_choice_prompting --objective denoising --optimizer adafactor --lr 0.0001 --epochs 3
```

## Evaluation signals

- Mean accuracy ordering across adaptation conditions and templates matches the Table B5 trend
- Masked Choice Prompting (MTO-MCP) with adaptation ranks highest among the four templates
- Predicting the choice number alongside choice text performs particularly well on CommonsenseQA with a denoising adaptation objective
- Standard deviations are computed from the same 3 random seeds rather than a single run, matching the three-repeat protocol
- Optimizer settings reproduce the article's configuration: AdaFactor, learning rate 0.0001, mini-batch 8 (T5-base) / 4 (T5-large), 3 epochs

## Limitations

- Few-shot QA experiments were repeated three times with mean scores reported; seed values / full variance details are not reported in the results section
- Code release was promised publicly but the README states the code is 'under refinement and improvement'; availability may need to be requested by email (pouramini at gmail)
- Results are specific to the encoder-decoder architecture; scaling with model parameters and varying amounts of adaptation-phase training data was not investigated

## Evidence
