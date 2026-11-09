---
name: prompt-tuning-template-design
description: Use when adapting T5-family encoder-decoder models with prompt tuning
  and you need soft-prompt placement and template structure (masked vs prompting)
  matched to a chosen pre-training objective (denoising vs LM).
when_to_use_negative:
- The downstream task is classification or another non-generative task type — the
  article states the impact of the proposed objective on classification was not investigated
- The target model is not an encoder-decoder architecture — results are specific to
  encoder-decoder pre-trained language models
- The task input is already a structured feature table or requires no input-format
  alignment with a pre-training objective
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: Match Task to Objective (MTO) framework
  role: Determines the objective for a given task, which dictates which prompt-tuning
    template (masked vs prompting, PrePT vs PostPT) to use
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: Checkpoint pre-trained with the unsupervised denoising objective; paired with
    masked template variants (Masked Prompting, Masked Answer/Choice Prompting)
- name: T5-lm
  role: T5-v1 derivative further trained 100k steps on the prefix LM objective; paired
    with prompting-style templates (Prompting, Answer Prompting, Choice Prompting)
- name: T5-base
  role: Widely adopted baseline encoder-decoder model used for the fine-tuning and
    prompt-tuning experiments
- name: T5-large
  role: Higher-capacity counterpart of T5-base used when increased capacity is needed
- name: Ada Factor optimizer
  role: Optimizer with consistent learning rate 0.0001 used throughout fine-tuning/adaptation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:54fa5337d16b0508b41c4aca2eb28f8e4cffa2323569b7d952e450696a76485d
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
yes_rate: 1.0
n_claims: 11
---

# prompt-tuning-template-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design and attach soft-prompt templates (PrePT/PostPT and QA variants) to encoder-decoder pre-trained language models such as T5, aligned with the unsupervised pre-training objective selected by the MTO framework, to enhance few-shot and full-dataset task performance.

## When to use

Use when adapting T5-family encoder-decoder models with prompt tuning and you need soft-prompt placement and template structure (masked vs prompting) matched to a chosen pre-training objective (denoising vs LM).

## When NOT to use

- The downstream task is classification or another non-generative task type — the article states the impact of the proposed objective on classification was not investigated
- The target model is not an encoder-decoder architecture — results are specific to encoder-decoder pre-trained language models
- The task input is already a structured feature table or requires no input-format alignment with a pre-training objective

## Inputs

- Task category (Mask-Filling, Map-Phrasal, or Question-Answering) from the MTO task classifier
- Selected pre-training objective (denoising or prefix LM) and matching T5 checkpoint (T5-v1, T5-lm, T5-base, T5-large)
- Adaptation data: natural sentences from the OMCS corpus and ATOMIC2020 tuples converted via templates, split by the Sentence Splitter
- Downstream task data (e.g. CommonsenseQA, Open BookQA, ATOMIC20 20 head-tail tuples)

## Outputs

- Prompt-tuning templates: PrePT and PostPT for KB-completion tasks; Answer Prompting, Masked Answer Prompting, Choice Prompting, Masked Choice Prompting for QA
- Fine-tuning templates keyed to the objective (Mapping, Prompting, Masked Mapping, Masked Prompting)
- Prompt-tuned model checkpoints evaluated under few-shot (n = 30) and full-dataset settings

## How to apply

After MTO assigns a task category (Mask-Filling, Map-Phrasal, or Question-Answering) and its objective, select the corresponding prompt-tuning template family: PrePT or PostPT for KB-completion-style tasks, and Answer Prompting, Masked Answer Prompting, Choice Prompting, or Masked Choice Prompting for QA. PrePT prepends prompt tokens to the beginning of the input sequence; PostPT appends prompt tokens at the end of the input, just before the placeholder or mask token. For denoising-aligned tasks use the masked variants, which replace the target element with a unique mask token ("X"), so the template's mask/prompt-token structure matches the objective used during adaptation (e.g. T5-v1 denoising vs T5-lm prefix LM). Run adaptation with unsupervised task-related data (e.g. Sentence Splitter output on OMCS sentences and ATOMIC2020 templates) using the AdaFactor optimizer at learning rate 0.0001 before prompt tuning.

## Related tools

- **Match Task to Objective (MTO) framework** (Determines the objective for a given task, which dictates which prompt-tuning template (masked vs prompting, PrePT vs PostPT) to use) — https://github.com/puraminy/MTO/
- **T5-v1** (Checkpoint pre-trained with the unsupervised denoising objective; paired with masked template variants (Masked Prompting, Masked Answer/Choice Prompting))
- **T5-lm** (T5-v1 derivative further trained 100k steps on the prefix LM objective; paired with prompting-style templates (Prompting, Answer Prompting, Choice Prompting))
- **T5-base** (Widely adopted baseline encoder-decoder model used for the fine-tuning and prompt-tuning experiments)
- **T5-large** (Higher-capacity counterpart of T5-base used when increased capacity is needed)
- **Ada Factor optimizer** (Optimizer with consistent learning rate 0.0001 used throughout fine-tuning/adaptation)

## Examples

```
git clone https://github.com/puraminy/MTO/ && cd MTO  # then configure prompt-tuning with PrePT/PostPT templates for the selected objective (code under refinement per README)
```

## Evaluation signals

- Verify each emitted template's mask/prompt-token structure matches the objective it is assigned to (masked variants for denoising, prompting variants for LM)
- Verify PrePT templates place prompt tokens before the input sequence and PostPT templates place them just before the placeholder or mask token
- Check that category-to-objective mapping holds: Mask-Filling→denoising (T5-v1), Map-Phrasal→LM (T5-lm)
- Few-shot runs (FS n = 30) repeated three times with mean scores reported; expect the reported performance gain of over 120% versus conventional methods in few-shot settings
- Confirm templates for QA tasks follow the question/choice prompt structure of Answer Prompting and Choice Prompting

## Limitations

- Results are specific to the encoder-decoder architecture; alternative architectures not explored
- Scaling with model parameters and varying amounts of adaptation-phase training data not investigated
- Impact of the proposed objective on classification and other task types not investigated
- Code availability was limited at the time: the README notes the code is under refinement and improvement

## Evidence

- [abstract] Prompt-tuning gains: [evidence span withheld — anchor status: not_found]
