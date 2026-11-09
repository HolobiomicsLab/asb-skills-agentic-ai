---
name: denoising-objective-adaptation-training
description: Use when few-shot fine-tuning data is tiny (e.g., n=30 samples per relation
  on ATOMIC2020 Mask-Filling tasks) and you need an adaptation stage that aligns the
  model with a denoising objective prior to supervised fine-tuning.
when_to_use_negative:
- The task is a classification-type task or the results are needed for a non-encoder-decoder
  architecture, since the paper's findings are specific to the encoder-decoder architecture
  and classification tasks were not investigated.
- Sufficient full-dataset supervised training data is available and the conventional
  Mapping baseline already performs well — the gain is most pronounced in few-shot
  settings.
- The target task's objective is the language-modeling objective rather than denoising
  (e.g., Map-Phrasal tasks), where the LM objective adaptation applies instead.
edam_operation: http://edamontology.org/operation_3095
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Match Task to Objective (MTO) framework
  role: Framework that identifies the appropriate objective (here denoising) and provides
    automated methods to prepare task-related data for the unsupervised adaptation
    stage
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: Encoder-decoder checkpoint exclusively pre-trained on C4 with an unsupervised
    denoising objective; adapted and fine-tuned under this skill
- name: T5-lm
  role: T5-v1 derivative that underwent 100k additional prefix-LM steps; alternative
    checkpoint for adaptation-stage comparisons
- name: T5-base
  role: Widely adopted checkpoint adapted with the denoising objective; fine-tuned
    with mini-batch size 8 and learning rate 0.0001
- name: Ada Factor optimizer
  role: Optimizer used with a consistent learning rate of 0.0001 throughout adaptation
    and fine-tuning
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:be4e83f7e9da6fcb74721b7a58781afb4463872c9b61283a530a9cc2a30b1cd8
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
  edam_operation: http://edamontology.org/operation_3095
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# denoising-objective-adaptation-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An unsupervised adaptation stage that trains T5 encoder-decoder checkpoints with the denoising (masked-token) objective on task-related natural sentences before supervised fine-tuning, so that the fine-tuning templates align with the model's pre-training objective and few-shot performance on commonsense KB completion is substantially improved.

## When to use

Use when few-shot fine-tuning data is tiny (e.g., n=30 samples per relation on ATOMIC2020 Mask-Filling tasks) and you need an adaptation stage that aligns the model with a denoising objective prior to supervised fine-tuning.

## When NOT to use

- The task is a classification-type task or the results are needed for a non-encoder-decoder architecture, since the paper's findings are specific to the encoder-decoder architecture and classification tasks were not investigated.
- Sufficient full-dataset supervised training data is available and the conventional Mapping baseline already performs well — the gain is most pronounced in few-shot settings.
- The target task's objective is the language-modeling objective rather than denoising (e.g., Map-Phrasal tasks), where the LM objective adaptation applies instead.

## Inputs

- Top 8,000 relevance-ranked OMCS sentences (natural-language commonsense corpus)
- Sentence Splitter model trained on knowledge graph tuples (e.g., ATOMIC/ATOMIC2020)
- T5 checkpoint (T5-v1, T5-lm, T5-base, or T5-large)
- Few-shot training split (n=30 samples per relation) from ATOMIC2020 Mask-Filling relations

## Outputs

- Adapted T5 checkpoint pre-trained on task-related sentences with the denoising objective
- Few-shot ROUGE scores on 100 unique heads per relation (each with at least 3 gold tails), written to mto_fewshot_rouge_results.csv
- Relative gain of the denoising-aligned strategy over the conventional Mapping baseline

## How to apply

First generate task-related adaptation data: split natural sentences from the OMCS corpus (top 8,000 relevance-ranked sentences) with a Sentence Splitter model trained on knowledge graph tuples, so the sentences decompose into knowledge-like fragments suitable for the denoising objective. Continue training the T5 checkpoint (T5-v1, T5-lm, T5-base, or T5-large) on these sentences in an unsupervised manner using the masked-token denoising objective for Mask-Filling tasks (or the prefix LM objective for Map-Phrasal tasks), before supervised fine-tuning on the target task. Then fine-tune with the AdaFactor optimizer at learning rate 0.0001, mini-batch size 8 for T5-base / 4 for T5-large, for 3 epochs, using templates whose 'X' mask placeholder matches the denoising objective. The rationale is the MTO conjecture that fine-tuning aligned with the unsupervised pre-training approach enhances few-shot performance.

## Related tools

- **Match Task to Objective (MTO) framework** (Framework that identifies the appropriate objective (here denoising) and provides automated methods to prepare task-related data for the unsupervised adaptation stage) — https://github.com/puraminy/MTO/
- **T5-v1** (Encoder-decoder checkpoint exclusively pre-trained on C4 with an unsupervised denoising objective; adapted and fine-tuned under this skill)
- **T5-lm** (T5-v1 derivative that underwent 100k additional prefix-LM steps; alternative checkpoint for adaptation-stage comparisons)
- **T5-base** (Widely adopted checkpoint adapted with the denoising objective; fine-tuned with mini-batch size 8 and learning rate 0.0001)
- **Ada Factor optimizer** (Optimizer used with a consistent learning rate of 0.0001 throughout adaptation and fine-tuning)

## Examples

```
python train.py --model t5-base --objective denoising --adaptation-data omcs_top8000_split.txt --lr 0.0001 --batch_size 8 --epochs 3 --few_shot 30
```

## Evaluation signals

- Best MTO-aligned objective-template strategy (including the denoising adaptation stage) achieves a relative ROUGE gain of over 120% compared to the conventional Mapping baseline (marked '-' in Figure 4) in the few-shot setting.
- Per-strategy ROUGE scores are computed on 100 unique heads per relation, each head having at least 3 gold tails (~300 instances), attributing each instance the highest score against its gold tails and averaging over instances.
- Few-shot experiments (FS n=30) are repeated three times and mean scores are reported; verify the mean over repeats is recorded in mto_fewshot_rouge_results.csv.
- Adaptation data consists of exactly the top 8,000 relevance-ranked OMCS sentences split by the Sentence Splitter, confirming the adaptation stage used task-related unsupervised data.
- Fine-tuning hyperparameters match the protocol: AdaFactor optimizer, learning rate 0.0001, mini-batch size 8 (T5-base) / 4 (T5-large), 3 epochs.

## Limitations

- The code was under refinement and only promised for future public release on GitHub; project materials may need to be requested from the authors.
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored.
- Scaling with model parameters and varying amounts of adaptation-phase training data were not investigated, so adaptation-data size (e.g., 8,000 OMCS sentences) was not tuned.
- Impact of the proposed objective on classification and other task types was not investigated.
- Random seeds / variance for few-shot experiments are not fully reported (only the mean of three repeats).

## Evidence

- [methods] Run the adaptation stage by unsupervised pretraining on the top 8,000 relevance-ranked OMCS sentences split by the Sentence Splitter, applying the denoising (masked-token) objective for Mask-Filling tasks: [evidence span withheld — anchor status: not_found]
