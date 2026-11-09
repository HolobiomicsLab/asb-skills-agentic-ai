---
name: multi-seed-accuracy-aggregation
description: Use when reporting few-shot QA accuracy (n=30 training samples) across
  multiple pre-training/adaptation objectives and prompting templates, where variance
  from seed selection must be quantified via repeated runs.
when_to_use_negative:
- Full-dataset (non-few-shot) QA experiments where seed variance is negligible and
  single-run evaluation suffices
- Generation or knowledge-base-completion tasks whose metrics are not accuracy over
  multiple-choice answers
- Encoder-only or decoder-only architectures — the MTO findings are specific to the
  encoder-decoder (T5) architecture
tools:
- name: Match Task to Objective (MTO) framework
  role: Defines the adaptation objectives (Denoising, LM, Mixed) whose few-shot accuracy
    is aggregated across seeds
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder PLM fine-tuned per seed and per template before accuracy evaluation
- name: Ada Factor optimizer
  role: Optimizer used for all fine-tuning runs (learning rate 0.0001) so that only
    the seed varies across repeats
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:549e1b217b1b3429d5668a0360d5d068f2e489960d68124d397d38cd5dce8e29
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multi-seed-accuracy-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Repeats few-shot QA fine-tuning experiments over three random seeds and aggregates mean (and standard deviation) accuracy for every adaptation-condition x template combination, following the MTO framework's evaluation protocol for CommonsenseQA and Open BookQA.

## When to use

Use when reporting few-shot QA accuracy (n=30 training samples) across multiple pre-training/adaptation objectives and prompting templates, where variance from seed selection must be quantified via repeated runs.

## When NOT to use

- Full-dataset (non-few-shot) QA experiments where seed variance is negligible and single-run evaluation suffices
- Generation or knowledge-base-completion tasks whose metrics are not accuracy over multiple-choice answers
- Encoder-only or decoder-only architectures — the MTO findings are specific to the encoder-decoder (T5) architecture

## Inputs

- Official CommonsenseQA train/validation/test splits
- Official Open BookQA train/validation/test splits
- Few-shot samples of n=30 per seed drawn for 3 random seeds
- T5 checkpoints (T5-v1, T5-lm, T5-base, T5-large) under Denoising, LM, or Mixed adaptation (or no adaptation)
- Per-run accuracy scores from each seed x objective x template configuration

## Outputs

- Mean and standard deviation accuracy per adaptation-condition x template combination (Table B5-style CSV)
- Ranking of templates/adaptation conditions identifying the best configuration

## How to apply

Sample n=30 few-shot training examples for each of 3 random seeds from the official CommonsenseQA and Open BookQA splits. For each seed, fine-tune the T5 checkpoints (T5-v1, T5-lm, T5-base, T5-large) with the AdaFactor optimizer (learning rate 0.0001, mini-batch size 8 for T5-base / 4 for T5-large, 3 epochs) under each adaptation condition (Denoising, LM, Mixed, or no adaptation) and each of the four QA templates (Answer Prompting, Masked Answer Prompting, Choice Prompting, Masked Choice Prompting). Evaluate accuracy on the QA test split per run, then compute the mean score across the three repeats per configuration and report the standard deviation to express seed variance. Compile the aggregated matrix into a Table B5-style CSV and rank configurations to identify the best template (expected: Masked Choice Prompting with adaptation, with predicting the choice number alongside choice text performing well on CommonsenseQA under a denoising objective).

## Related tools

- **Match Task to Objective (MTO) framework** (Defines the adaptation objectives (Denoising, LM, Mixed) whose few-shot accuracy is aggregated across seeds) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder PLM fine-tuned per seed and per template before accuracy evaluation)
- **Ada Factor optimizer** (Optimizer used for all fine-tuning runs (learning rate 0.0001) so that only the seed varies across repeats)

## Evaluation signals

- Exactly three repeated runs (seeds) per configuration, with both mean and standard deviation reported per cell
- Aggregate ordering matches the paper's trend: Masked Choice Prompting under adaptation ranks highest in few-shot QA
- All runs share fixed hyperparameters: AdaFactor, learning rate 0.0001, mini-batch 8 (T5-base) / 4 (T5-large), 3 epochs
- Accuracy is computed on the official QA test split for every configuration, not on the n=30 few-shot training samples
- Aggregated table covers the full cross-product of adaptation conditions x four QA templates (Answer/Masked Answer/Choice/Masked Choice Prompting)

## Limitations

- Random seeds/variance for the few-shot QA experiments were only noted as a mean of three repeats; the detailed seed list is not reported in the results section
- Results are specific to the encoder-decoder architecture; alternative architectures are not covered
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated
- The repository README states the code is under refinement and availability of materials may require contacting the authors

## Evidence
