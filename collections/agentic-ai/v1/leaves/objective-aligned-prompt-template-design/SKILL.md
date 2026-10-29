---
name: objective-aligned-prompt-template-design
description: Use when few-shot fine-tuning an encoder-decoder PLM (e.g., T5) on generation
  or KB-completion tasks and you need templates matching the pre-training objective
  (denoising vs LM) to boost accuracy.
when_to_use_negative:
- The task is pure classification or a non-generative task type — the article notes
  the impact of the proposed objective on classification and other task types was
  not investigated.
- You need results for architectures other than encoder-decoder PLMs — the authors
  state results are specific to the encoder-decoder architecture.
- You require validated, publicly released code before reuse — the README states the
  code is 'under refinement and improvement' and availability is by contact.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3376
- http://edamontology.org/topic_3474
tools:
- name: Match Task to Objective (MTO) framework
  role: Framework for classifying tasks and selecting the pre-training objective and
    template strategy
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: Checkpoint pre-trained solely on C4 with a denoising objective; tests objective-template
    alignment
  repo: https://github.com/puraminy/MTO/
- name: T5-lm
  role: T5-v1 derivative given 100k extra prefix-LM steps; tests LM-objective alignment
  repo: https://github.com/puraminy/MTO/
- name: T5-base
  role: Widely adopted downstream checkpoint fine-tuned with the aligned templates
- name: Ada Factor optimizer
  role: Optimizer used with consistent learning rate 0.0001 throughout fine-tuning
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:8f82b539611fd37bc53e35931d66db84213f6a5bb052c23d61c497c73cfcc4a1
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
  - T5-lm
  - T5-v1
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3376
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# objective-aligned-prompt-template-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design fine-tuning input templates (Mapping, Prompting, Masked Mapping, Masked Prompting) that align a task's format with the encoder-decoder PLM's unsupervised pre-training objective, combined with an unsupervised adaptation stage, to maximize few-shot performance on commonsense KB completion and QA tasks.

## When to use

Use when few-shot fine-tuning an encoder-decoder PLM (e.g., T5) on generation or KB-completion tasks and you need templates matching the pre-training objective (denoising vs LM) to boost accuracy.

## When NOT to use

- The task is pure classification or a non-generative task type — the article notes the impact of the proposed objective on classification and other task types was not investigated.
- You need results for architectures other than encoder-decoder PLMs — the authors state results are specific to the encoder-decoder architecture.
- You require validated, publicly released code before reuse — the README states the code is 'under refinement and improvement' and availability is by contact.

## Inputs

- ATOMIC2020 head-tail tuples for Mask-Filling and Map-Phrasal relations
- Few-shot training splits (n = 30 samples per relation/task)
- Task-related natural sentences (e.g., OMCS corpus) for the adaptation stage
- Pre-trained T5 checkpoints: T5-v1, T5-lm, T5-base, T5-large

## Outputs

- Generated tails/answers per template strategy
- Per-strategy ROUGE scores on 100 unique heads per relation (~300 instances)
- Relative gain of each aligned strategy over the Mapping baseline (e.g., written to mto_fewshot_rouge_results.csv)

## How to apply

Classify the task into categories to select the matching pre-training objective. For Mask-Filling tasks on ATOMIC2020 relations, format head-tail pairs with Table 3 templates: Mapping (supervised seq2seq), Prompting (natural language), Masked Mapping and Masked Prompting (placing the 'X' mask token before the target fragment). Run an adaptation stage of unsupervised training on task-related natural sentences — the top 8,000 relevance-ranked OMCS sentences split by the Sentence Splitter — applying the denoising (masked-token) objective for Mask-Filling tasks and the LM objective for Map-Phrasal tasks. Fine-tune T5-v1, T5-lm, and T5-base/T5-large with the AdaFactor optimizer (learning rate 0.0001, mini-batch size 8 for T5-base / 4 for T5-large, 3 epochs) on few-shot splits of n = 30 samples per relation. For prompt-tuning variants, place soft prompt tokens before the input (PrePT) or just before the mask placeholder (PostPT). Compare each aligned strategy against the conventional Mapping baseline and compute relative gain.

## Related tools

- **Match Task to Objective (MTO) framework** (Framework for classifying tasks and selecting the pre-training objective and template strategy) — https://github.com/puraminy/MTO/
- **T5-v1** (Checkpoint pre-trained solely on C4 with a denoising objective; tests objective-template alignment) — https://github.com/puraminy/MTO/
- **T5-lm** (T5-v1 derivative given 100k extra prefix-LM steps; tests LM-objective alignment) — https://github.com/puraminy/MTO/
- **T5-base** (Widely adopted downstream checkpoint fine-tuned with the aligned templates)
- **Ada Factor optimizer** (Optimizer used with consistent learning rate 0.0001 throughout fine-tuning)

## Examples

```
python train.py --model t5-base --task masked_prompting --few_shot 30 --lr 1e-4 --batch_size 8 --epochs 3 --adaptation_omcs_top 8000
```

## Evaluation signals

- Best MTO-aligned strategy achieves a relative ROUGE gain of over 120% over the conventional Mapping baseline in the few-shot (n = 30) setting
- Masked templates place the 'X' mask token immediately before the target fragment, matching the span-corruption/denoising format
- Adaptation stage uses exactly the top 8,000 relevance-ranked OMCS sentences split by the Sentence Splitter, with denoising objective for Mask-Filling and LM objective for Map-Phrasal tasks
- Few-shot experiments repeated three times with mean scores reported across seeds
- ROUGE computed on 100 unique heads per relation, each with at least 3 gold tails (~300 instances), attributing each instance the highest score against its gold tails

## Limitations

- Results are specific to the encoder-decoder architecture; alternative architectures were not explored
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated
- Random seeds/variance for few-shot experiments are only summarized as the mean of three repeats
- Repository code is 'under refinement and improvement' and materials are made available on request, so reproduction is not turnkey

## Evidence
