---
name: t5-encoder-decoder-fine-tuning
description: Use when fine-tuning T5 encoder-decoder models on few-shot commonsense
  generation tasks (e.g., ATOMIC tail generation, n=30) and you must decide which
  adaptation objective and input template to pair with each checkpoint.
when_to_use_negative:
- Tasks outside the encoder-decoder architecture — the authors note the results are
  specific to encoder-decoder generative models
- Classification or other non-generative task types — the impact of the proposed objective
  on classification was not investigated
- Scaling studies over model parameters or varying adaptation-data amounts — this
  was explicitly left uninvestigated
edam_operation: http://edamontology.org/operation_3093
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: T5-v1
  role: Checkpoint pre-trained on C4 with the unsupervised denoising replace-spans
    objective; pairs best with Denoising adaptation and Masked Mapping/Masked Prompting
    fine-tuning
  repo: https://github.com/puraminy/MTO/
- name: T5-lm
  role: T5-v1 derivation given 100k extra prefix-LM training steps; use when aligning
    with language-model objectives
  repo: https://github.com/puraminy/MTO/
- name: T5-base
  role: Widely adopted checkpoint trained on C4 + WikiDPR with supervised multitask
    pre-training; fine-tuned at batch size 8
  repo: https://github.com/puraminy/MTO/
- name: T5-large
  role: Higher-capacity counterpart of T5-base; fine-tuned at batch size 4; T5-large
    + Mixed + Masked Prompting emerges as best for Mask-Filling
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer used for all fine-tuning with a consistent learning rate of 0.0001
  repo: https://github.com/puraminy/MTO/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:8ef4ecb50c9716d910fd408e1eb71a87ac1a8adfff310f8a2843bed6f2bb2a28
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
  - T5
  - T5-base
  - T5-large
  - T5-lm
  - T5-v1
  edam_operation: http://edamontology.org/operation_3093
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# t5-encoder-decoder-fine-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tune T5 encoder-decoder checkpoints (T5-v1, T5-lm, T5-base, T5-large) for few-shot KB completion by aligning the fine-tuning template (Mapping, Prompting, Masked Mapping, Masked Prompting) with the model's unsupervised pre-training/adaptation objective, per the Match Task to Objective (MTO) framework.

## When to use

Use when fine-tuning T5 encoder-decoder models on few-shot commonsense generation tasks (e.g., ATOMIC tail generation, n=30) and you must decide which adaptation objective and input template to pair with each checkpoint.

## When NOT to use

- Tasks outside the encoder-decoder architecture — the authors note the results are specific to encoder-decoder generative models
- Classification or other non-generative task types — the impact of the proposed objective on classification was not investigated
- Scaling studies over model parameters or varying adaptation-data amounts — this was explicitly left uninvestigated

## Inputs

- ATOMIC2020 (head, relation, tail) tuples for selected Mask-Filling and Map-Phrasal relations
- Few-shot training sets (n=30 samples per relation) in Table 3 template formats (Mapping, Prompting, Masked Mapping, Masked Prompting)
- T5 checkpoints: T5-v1, T5-lm, T5-base, T5-large
- Optional: top-8,000 scored OMCS sentences for Mixed-objective adaptation, split by the trained Sentence Splitter
- Evaluation set of 100 unique heads per relation, each with at least three target tails (~300 instances per relation)

## Outputs

- Model-generated tails for each evaluation head
- Mean ROUGE and BERTScore tables per model/objective/template combination (mirroring Tables B3 and B4)
- Best per-model objective/template assignment for Mask-Filling and Map-Phrasal tasks

## How to apply

Load (head, relation, tail) tuples from ATOMIC2020 and format few-shot sets of n=30 samples per relation using the four templates built from the relation templates in Table A1: Mapping (traditional supervised), Prompting (natural-language prompts), Masked Mapping (mask token 'X' placeholder), and Masked Prompting (prompt with masked target). Load T5 checkpoints whose pre-training must match the template: T5-v1 (C4, denoising replace-spans), T5-lm (T5-v1 + 100k prefix-LM steps), T5-base/T5-large (C4 + WikiDPR supervised multitask). Optionally adapt models first on task-related unsupervised data: split top-8,000 scored OMCS sentences with a Sentence Splitter trained on knowledge-graph tuples, and label samples as denoising- or LM-suitable for the Mixed objective. Fine-tune with the AdaFactor optimizer at learning rate 0.0001, batch size 8 (T5-base) or 4 (T5-large), for 3 epochs. Generate tails for 100 unique heads (each with >=3 target tails, ~300 instances per relation) and score with ROUGE and BERTScore, assigning each instance the highest score over its head's tails. Rationale: masked-token templates aligned with the denoising objective consistently outperform non-masked mapping methods for T5-v1 and T5-lm.

## Related tools

- **T5-v1** (Checkpoint pre-trained on C4 with the unsupervised denoising replace-spans objective; pairs best with Denoising adaptation and Masked Mapping/Masked Prompting fine-tuning) — https://github.com/puraminy/MTO/
- **T5-lm** (T5-v1 derivation given 100k extra prefix-LM training steps; use when aligning with language-model objectives) — https://github.com/puraminy/MTO/
- **T5-base** (Widely adopted checkpoint trained on C4 + WikiDPR with supervised multitask pre-training; fine-tuned at batch size 8) — https://github.com/puraminy/MTO/
- **T5-large** (Higher-capacity counterpart of T5-base; fine-tuned at batch size 4; T5-large + Mixed + Masked Prompting emerges as best for Mask-Filling) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer used for all fine-tuning with a consistent learning rate of 0.0001) — https://github.com/puraminy/MTO/

## Evaluation signals

- Reproduced mean ROUGE/BERTScore for each model/objective/template combination match the values in Tables B3 and B4
- Each task evaluation uses n=30 few-shot samples over 3 epochs as in Figure 4
- T5-large + Mixed objective + Masked Prompting emerges as the best configuration for Mask-Filling
- For T5-v1 and T5-lm, Denoising adaptation + Masked Mapping or Masked Prompting outperforms non-masked Mapping/Prompting templates
- Experiments repeated three times with mean scores reported for few-shot settings

## Limitations

- Random seeds and variance for few-shot QA experiments are not reported (only the mean of three repeats)
- Code availability was promised for the future and the repository README states the code is 'under refinement and improvement'
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated

## Evidence
