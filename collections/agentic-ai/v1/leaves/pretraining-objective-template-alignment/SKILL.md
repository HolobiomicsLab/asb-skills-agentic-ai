---
name: pretraining-objective-template-alignment
description: Use when fine-tuning T5 variants (T5-v1, T5-lm, T5-base, T5-large) few-shot
  (n=30, 3 epochs) on KB-completion tasks like ATOMIC Mask-Filling/Map-Phrasal and
  you must choose which adaptation objective and fine-tuning template to pair with
  each checkpoint.
when_to_use_negative:
- The task is classification or another non-generation task type — the article notes
  the impact of the proposed objective on classification was not investigated
- You need results for architectures other than encoder-decoder models — findings
  are specific to encoder-decoder pre-trained language models
- You need scaling analysis across model sizes or varying adaptation-phase data amounts
  — the article explicitly leaves this uninvestigated
edam_operation: http://edamontology.org/operation_3281
edam_topics:
- http://edamontology.org/topic_3376
tools:
- name: T5-v1
  role: Checkpoint pre-trained exclusively on C4 with the unsupervised denoising objective;
    aligns with Denoising objective and masked-token templates
  repo: https://github.com/puraminy/MTO/
- name: T5-lm
  role: Derivative of T5-v1 with an added 100k prefix-LM steps; pairs with LM-aligned
    non-masked templates
  repo: https://github.com/puraminy/MTO/
- name: T5-base
  role: Widely adopted downstream checkpoint (C4 + WikiDPR multitask); fine-tuned
    with AdaFactor at batch size 8
  repo: https://github.com/puraminy/MTO/
- name: T5-large
  role: Higher-capacity counterpart of T5-base; with Mixed objective + Masked Prompting
    it emerges as best for Mask-Filling; batch size 4
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer used with a consistent learning rate of 0.0001 throughout fine-tuning
  repo: https://github.com/puraminy/MTO/
- name: Match Task to Objective (MTO) framework
  role: Framework that identifies the appropriate pre-training objective for a task
    and automates adaptation-data preparation
  repo: https://github.com/puraminy/MTO/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:df9b4ea616e7d4ca13791a2c738b293e46afd680ca98dfc91c7f03c9781ee45e
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
  edam_operation: http://edamontology.org/operation_3281
  edam_topics:
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# pretraining-objective-template-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns the fine-tuning template and unsupervised adaptation objective of T5 encoder-decoder models with their pre-training objective to boost few-shot KB-completion performance on ATOMIC relations. Grounded in the Match Task to Objective (MTO) framework, it maps masked-token templates (Masked Mapping, Masked Prompting) to denoising-style models and non-masked templates to LM-style models.

## When to use

Use when fine-tuning T5 variants (T5-v1, T5-lm, T5-base, T5-large) few-shot (n=30, 3 epochs) on KB-completion tasks like ATOMIC Mask-Filling/Map-Phrasal and you must choose which adaptation objective and fine-tuning template to pair with each checkpoint.

## When NOT to use

- The task is classification or another non-generation task type — the article notes the impact of the proposed objective on classification was not investigated
- You need results for architectures other than encoder-decoder models — findings are specific to encoder-decoder pre-trained language models
- You need scaling analysis across model sizes or varying adaptation-phase data amounts — the article explicitly leaves this uninvestigated

## Inputs

- ATOMIC2020 (head, relation, tail) tuples for selected Mask-Filling and Map-Phrasal relations
- Few-shot training sets of n=30 samples per relation formatted with Mapping / Prompting / Masked Mapping / Masked Prompting templates from ATOMIC relation templates
- T5 checkpoints: T5-v1, T5-lm, T5-base, T5-large
- Top 8,000 scored OMCS sentences plus Sentence Splitter output for Mixed-objective adaptation data

## Outputs

- Per-model, per-objective/template ROUGE and BERTScore results table mirroring Tables B3 and B4
- Best per-model objective/template assignment for Mask-Filling and Map-Phrasal (expected best: T5-large + Mixed + Masked Prompting for Mask-Filling)
- Objective-labeled Mixed adaptation samples from Sentence Splitter output

## How to apply

First identify each model's pre-training objective: T5-v1 uses the denoising replace-spans objective on C4, T5-lm adds 100k prefix-LM steps, T5-base/T5-large use supervised multitask pre-training on C4 + WikiDPR. Select fine-tuning templates (Mapping, Prompting, Masked Mapping, Masked Prompting) built from ATOMIC relation templates so that masked-token templates (using an 'X' placeholder) align with denoising-style models, and non-masked mapping templates align with LM-style models. For adaptation, use the Denoising objective for T5-v1 and T5-lm to reach their best Mask-Filling performance; for Mixed-objective configurations, split the top 8,000 scored OMCS sentences with the trained Sentence Splitter and label each sample as denoising- or LM-suitable. Fine-tune with the AdaFactor optimizer at learning rate 0.0001, batch size 8 for T5-base and 4 for T5-large, for 3 epochs over n=30 samples per relation. Validate by generating tails for 100 unique heads (~300 instances per relation) and computing ROUGE and BERTScore, assigning each instance the highest score over its head's tails.

## Related tools

- **T5-v1** (Checkpoint pre-trained exclusively on C4 with the unsupervised denoising objective; aligns with Denoising objective and masked-token templates) — https://github.com/puraminy/MTO/
- **T5-lm** (Derivative of T5-v1 with an added 100k prefix-LM steps; pairs with LM-aligned non-masked templates) — https://github.com/puraminy/MTO/
- **T5-base** (Widely adopted downstream checkpoint (C4 + WikiDPR multitask); fine-tuned with AdaFactor at batch size 8) — https://github.com/puraminy/MTO/
- **T5-large** (Higher-capacity counterpart of T5-base; with Mixed objective + Masked Prompting it emerges as best for Mask-Filling; batch size 4) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer used with a consistent learning rate of 0.0001 throughout fine-tuning) — https://github.com/puraminy/MTO/
- **Match Task to Objective (MTO) framework** (Framework that identifies the appropriate pre-training objective for a task and automates adaptation-data preparation) — https://github.com/puraminy/MTO/

## Examples

```
git clone https://github.com/puraminy/MTO/ && cd MTO  # then run per-config fine-tuning: AdaFactor optimizer, lr=0.0001, 3 epochs, n=30 ATOMIC samples per relation
```

## Evaluation signals

- Reproduced mean ROUGE/BERTScore per model/objective/template combination should match values reported in Tables B3 and B4
- Each evaluation must use n=30 samples over 3 epochs, matching the few-shot protocol in Figure 4
- T5-large + Mixed + Masked Prompting should emerge as the best configuration for Mask-Filling
- For T5-v1 and T5-lm, Denoising adaptation + Masked Mapping/Masked Prompting fine-tuning should yield the highest Mask-Filling scores
- Masked-token templates should consistently outperform non-masked mapping methods across models

## Limitations

- Few-shot QA experiments report only the mean of three repeats; random seeds and variance are not reported in the results section
- Code was under refinement at README time and only promised for future public release on GitHub
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated

## Evidence

- [full_text] For T5-v1 and T5-LM, the highest Mask-Filling performance is achieved when the adaptation stage uses the Denoising objective and the fine-tuning stage employs Masked Mapping or Masked Prompting templates: [evidence span withheld — anchor status: not_found]
