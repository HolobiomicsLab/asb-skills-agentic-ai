---
name: masked-choice-prompt-template-design
description: Use when fine-tuning an encoder-decoder PLM (e.g., T5-base/T5-large)
  on few-shot (n=30) multiple-choice QA such as CommonsenseQA or Open BookQA and the
  adaptation stage used a denoising objective, so the fine-tuning template should
  mask the target choice.
when_to_use_negative:
- The downstream task is KB completion or generation rather than multiple-choice question
  answering — the KB templates (Mapping, Prompting, Masked Mapping, Masked Prompting)
  apply there instead
- The adaptation or pre-training objective is a prefix LM objective without denoising,
  where a non-masked template (e.g., Choice Prompting) may align better
- The model is not an encoder-decoder PLM — the article states results are specific
  to the encoder-decoder architecture
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Match Task to Objective (MTO) framework
  role: Framework that determines the appropriate unsupervised adaptation objective
    (denoising) and provides automated preparation of task-related adaptation data
    before the masked template is applied
  repo: https://github.com/puraminy/MTO/
- name: T5-base
  role: Encoder-decoder PLM fine-tuned with the Masked Choice Prompting template (mini-batch
    size 8)
- name: T5-large
  role: Higher-capacity encoder-decoder PLM fine-tuned with the Masked Choice Prompting
    template (mini-batch size 4)
- name: Ada Factor optimizer
  role: Optimizer used for fine-tuning with the template, at a consistent learning
    rate of 0.0001
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:6c4a77195a78b1a69c4f7db240b2054e90370e2874b9077eaa26dfbcde71e46e
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# Masked Choice Prompt Template Design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Designs multiple-choice QA fine-tuning templates that align with denoising adaptation objectives, where the model predicts the choice number alongside the choice text under a masked-token format. In the MTO framework this Masked Choice Prompting template consistently outperformed Answer Prompting, Masked Answer Prompting, and Choice Prompting on few-shot CommonsenseQA and Open BookQA.

## When to use

Use when fine-tuning an encoder-decoder PLM (e.g., T5-base/T5-large) on few-shot (n=30) multiple-choice QA such as CommonsenseQA or Open BookQA and the adaptation stage used a denoising objective, so the fine-tuning template should mask the target choice.

## When NOT to use

- The downstream task is KB completion or generation rather than multiple-choice question answering — the KB templates (Mapping, Prompting, Masked Mapping, Masked Prompting) apply there instead
- The adaptation or pre-training objective is a prefix LM objective without denoising, where a non-masked template (e.g., Choice Prompting) may align better
- The model is not an encoder-decoder PLM — the article states results are specific to the encoder-decoder architecture

## Inputs

- Few-shot (n=30) CommonsenseQA or Open BookQA training examples sampled per random seed
- T5 checkpoints (T5-v1, T5-lm, T5-base, T5-large), optionally adapted via Denoising/LM/Mixed objectives
- QA test splits for accuracy evaluation

## Outputs

- Fine-tuned model predictions of choice number plus choice text per test question
- Mean and standard deviation accuracy per adaptation-condition x template combination (Table B5-style CSV)

## How to apply

Format each few-shot example as a prompt that presents the question followed by its candidate choices, and replace the target with a mask placeholder so the model outputs the correct choice number together with the choice text — predicting the choice number is especially effective on CommonsenseQA under a denoising adaptation objective. Apply the adaptation stage first (Denoising, LM, or Mixed unsupervised objective, or skip adaptation) per the MTO framework, then fine-tune with the AdaFactor optimizer at learning rate 0.0001, mini-batch size 8 for T5-base / 4 for T5-large, over 3 epochs. Compare against the three alternative QA templates from Table 4 (Answer Prompting, Masked Answer Prompting, Choice Prompting) and evaluate accuracy on the test split, repeating over 3 random seeds and reporting mean and standard deviation.

## Related tools

- **Match Task to Objective (MTO) framework** (Framework that determines the appropriate unsupervised adaptation objective (denoising) and provides automated preparation of task-related adaptation data before the masked template is applied) — https://github.com/puraminy/MTO/
- **T5-base** (Encoder-decoder PLM fine-tuned with the Masked Choice Prompting template (mini-batch size 8))
- **T5-large** (Higher-capacity encoder-decoder PLM fine-tuned with the Masked Choice Prompting template (mini-batch size 4))
- **Ada Factor optimizer** (Optimizer used for fine-tuning with the template, at a consistent learning rate of 0.0001)

## Examples

```
python train.py --task commonsenseqa --template masked_choice_prompting --adaptation denoising --model t5-base --lr 0.0001 --batch_size 8 --epochs 3 --n_fewshot 30 --seeds 3
```

## Evaluation signals

- Masked Choice Prompting with adaptation ranks highest in mean accuracy across the four QA templates and adaptation conditions (Table B5 trend)
- Predicting the choice number alongside the choice text yields particularly strong accuracy on CommonsenseQA under the denoising adaptation objective
- Reproduced accuracy ordering across adaptation conditions x templates matches the published Table B5 ordering over 3 random seeds (mean ± std reported)
- Few-shot results show the reported performance gain of over 120% relative to conventional fine-tuning methods in few-shot settings

## Limitations

- In few-shot settings (FS n=30) the experiments were repeated three times and only mean scores were reported in the main results section; seed-level variance handling is not fully documented
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored
- The code repository was 'under refinement and improvement' and users must contact the authors (pouramini at gmail) to expedite availability of project materials
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated

## Evidence

- [results] In few-shot QA, Masked Choice Prompting (MTO-MCP) consistently outperforms other proposed methods; predicting the choice number alongside the choice text performs particularly well on CommonsenseQA with a denoising adaptation objective.: [evidence span withheld — anchor status: not_found]
- [methods] Masked Answer Prompting uses masked tokens under a denoising objective: [evidence span withheld — anchor status: not_found]
