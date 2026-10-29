---
name: encoder-decoder-text-to-text-evaluation
description: Use when reproducing or comparing few-shot QA accuracy of T5-style encoder-decoder
  models across pre-training/adaptation objectives and prompting templates.
when_to_use_negative:
- The task is classification or another non-QA task type — the article states impact
  of the proposed objective on classification and other task types was not investigated
- You need non-encoder-decoder architectures — results are reported as specific to
  the encoder-decoder architecture
- You need scaling analysis across model sizes or adaptation-data amounts — this was
  explicitly not investigated
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3376
- http://edamontology.org/topic_3474
tools:
- name: Match Task to Objective (MTO) framework
  role: Framework that selects the appropriate unsupervised objective (Denoising,
    LM, or Mixed) for the QA task and prepares task-related adaptation data
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder text-to-text model used for all fine-tuning and evaluation
    experiments
- name: T5-v1
  role: Checkpoint pre-trained exclusively on C4 with a denoising objective
- name: T5-lm
  role: Derivative of T5-v1 given 100k additional prefix-LM training steps
- name: T5-base
  role: Widely adopted downstream model; fine-tuned with AdaFactor, mini-batch size
    8
- name: T5-large
  role: Higher-capacity counterpart; fine-tuned with AdaFactor, mini-batch size 4
- name: Ada Factor optimizer
  role: Optimizer with a consistent learning rate of 0.0001 throughout fine-tuning
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:2ac0b529848903fdf2eb6343b13e376db9b615b9857fca69be8f91095fe3dce2
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
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3376
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# encoder-decoder-text-to-text-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates encoder-decoder text-to-text models (T5 family) on few-shot question answering by aligning fine-tuning templates with unsupervised adaptation objectives under the Match Task to Objective (MTO) framework, using accuracy on CommonsenseQA and Open BookQA test splits.

## When to use

Use when reproducing or comparing few-shot QA accuracy of T5-style encoder-decoder models across pre-training/adaptation objectives and prompting templates.

## When NOT to use

- The task is classification or another non-QA task type — the article states impact of the proposed objective on classification and other task types was not investigated
- You need non-encoder-decoder architectures — results are reported as specific to the encoder-decoder architecture
- You need scaling analysis across model sizes or adaptation-data amounts — this was explicitly not investigated

## Inputs

- CommonsenseQA train/validation/test splits
- Open BookQA train/validation/test splits
- T5 checkpoints (T5-v1, T5-lm, T5-base, T5-large)
- Few-shot sample of n=30 training examples per seed
- QA prompt templates from Table 4 (Answer Prompting, Masked Answer Prompting, Choice Prompting, Masked Choice Prompting)

## Outputs

- Accuracy per configuration on the QA test split
- Mean and standard deviation accuracy aggregated over 3 random seeds
- Table B5-style CSV of adaptation-condition x template results
- Identification of the best-performing template (expected: Masked Choice Prompting with adaptation)

## How to apply

Load the official CommonsenseQA and Open BookQA splits and sample n=30 few-shot training examples for each of 3 random seeds. Prepare T5 checkpoints (T5-v1, T5-lm, T5-base, T5-large) and optionally run the adaptation stage with Denoising, LM, or Mixed unsupervised objectives on task-related data per the MTO framework. Fine-tune with the AdaFactor optimizer (learning rate 0.0001, mini-batch size 8 for T5-base / 4 for T5-large, 3 epochs) using the four QA templates: Answer Prompting, Masked Answer Prompting, Choice Prompting, and Masked Choice Prompting. Evaluate accuracy on the test split, repeat across seeds, and aggregate mean and standard deviation per adaptation-condition x template combination. The expected outcome is that Masked Choice Prompting with a denoising adaptation objective ranks highest, matching the Table B5 trend.

## Related tools

- **Match Task to Objective (MTO) framework** (Framework that selects the appropriate unsupervised objective (Denoising, LM, or Mixed) for the QA task and prepares task-related adaptation data) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder text-to-text model used for all fine-tuning and evaluation experiments)
- **T5-v1** (Checkpoint pre-trained exclusively on C4 with a denoising objective)
- **T5-lm** (Derivative of T5-v1 given 100k additional prefix-LM training steps)
- **T5-base** (Widely adopted downstream model; fine-tuned with AdaFactor, mini-batch size 8)
- **T5-large** (Higher-capacity counterpart; fine-tuned with AdaFactor, mini-batch size 4)
- **Ada Factor optimizer** (Optimizer with a consistent learning rate of 0.0001 throughout fine-tuning)

## Examples

```
git clone https://github.com/puraminy/MTO/ && python train.py --model t5-base --objective denoising --template masked_choice_prompting --lr 0.0001 --batch_size 8 --epochs 3 --few_shot 30
```

## Evaluation signals

- Mean accuracy ordering across adaptation conditions and templates matches the Table B5 trend
- Masked Choice Prompting (MTO-MCP) ranks highest among the four templates, particularly on CommonsenseQA with a denoising adaptation objective
- Standard deviation is computed over exactly 3 random seeds with n=30 few-shot examples each
- Fine-tuning hyperparameters match the paper: AdaFactor, learning rate 0.0001, mini-batch size 8 (T5-base) / 4 (T5-large), 3 epochs
- Few-shot results show the reported performance gain of over 120% compared to conventional methods

## Limitations

- Random seeds / variance for few-shot QA experiments are not fully reported in the results section (only mean of three repeats noted)
- The repository README states the code is under refinement and improvement; project resources may need to be requested from the authors (pouramini at gmail)
- Results are specific to the encoder-decoder architecture; alternative architectures are not explored
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated
- Impact of the proposed objective on classification and other task types was not investigated

## Evidence

- [other] In few-shot QA, Masked Choice Prompting (MTO-MCP) consistently outperforms other proposed methods: [evidence span withheld — anchor status: not_found]
