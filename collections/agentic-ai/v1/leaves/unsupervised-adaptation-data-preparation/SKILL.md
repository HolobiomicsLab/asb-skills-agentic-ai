---
name: unsupervised-adaptation-data-preparation
description: Use when a downstream task has been mapped to a pre-training objective
  (denoising or LM) via MTO and task-related natural-language data must be generated
  for an unsupervised adaptation stage before fine-tuning.
when_to_use_negative:
- The task is a plain classification task — the article's discussion notes the impact
  of the proposed objective on classification and other task types was not investigated
- No task-to-objective alignment is needed, e.g. the model will simply be fine-tuned
  on the full downstream dataset without an adaptation stage
- The target architecture is not encoder-decoder (e.g. decoder-only LMs) — the paper's
  results are specific to the encoder-decoder architecture such as T5
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3377
- http://edamontology.org/topic_3474
tools:
- name: Match Task to Objective (MTO) framework
  role: Supplies the identified objective that determines how adaptation data are
    prepared and labeled
  repo: https://github.com/puraminy/MTO/
- name: T5-v1
  role: Pre-trained model used with the denoising (Mask-Filling) objective during
    adaptation
  repo: https://github.com/puraminy/MTO/
- name: T5-lm
  role: T5-v1 derivative pre-trained 100k additional steps on the prefix LM objective,
    used when the Map-Phrasal objective is selected
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer for the adaptation pre-training and fine-tuning stages at learning
    rate 0.0001
  repo: https://github.com/puraminy/MTO/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:715a7a7d6a7b7ff61d0a5700da28b06c52d978453352aa1807611998c3a88015
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3377
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# unsupervised-adaptation-data-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated preparation of task-related unsupervised training data for the adaptation stage of the MTO framework: ATOMIC2020 knowledge-graph tuples are converted into natural sentences to train a Sentence Splitter, which then segments OMCS corpus sentences into head-tail parts for extended unsupervised pre-training under the objective selected for the task.

## When to use

Use when a downstream task has been mapped to a pre-training objective (denoising or LM) via MTO and task-related natural-language data must be generated for an unsupervised adaptation stage before fine-tuning.

## When NOT to use

- The task is a plain classification task — the article's discussion notes the impact of the proposed objective on classification and other task types was not investigated
- No task-to-objective alignment is needed, e.g. the model will simply be fine-tuned on the full downstream dataset without an adaptation stage
- The target architecture is not encoder-decoder (e.g. decoder-only LMs) — the paper's results are specific to the encoder-decoder architecture such as T5

## Inputs

- ATOMIC2020 knowledge-graph tuples (head, relation, tail)
- Open Mind Common Sense (OMCS) corpus sentences (~700,000 natural-language commonsense sentences)
- Objective identified by the MTO task classifier (Mask-Filling → denoising, Map-Phrasal → LM, or Mixed)
- Templates converting tuples into natural sentences

## Outputs

- Trained Sentence Splitter model
- Head-tail-like unsupervised training samples from the top 8,000 relevance-ranked OMCS sentences
- Optionally 'Mixed' data with per-sample objective labels
- Bootstrap-expanded unsupervised sentence set from model-generated completions
- Adaptation-phase pre-training configuration (objective/blend, AdaFactor, lr 0.0001)

## How to apply

First convert ATOMIC2020 knowledge-graph tuples into natural sentences via templates and train the Sentence Splitter model on about 5,000 of them to segment sentences into head- and tail-like parts. Apply the trained Sentence Splitter to the top 8,000 Open Mind Common Sense (OMCS) sentences ranked by relevance score to produce unsupervised training samples; optionally run a classifier over each produced sample to label it with its suitable objective, forming 'Mixed' data that blends denoising and LM targets. Bootstrap additional sentences from model-generated completions to enlarge the unsupervised pool. Finally configure the adaptation stage to pre-train the selected T5 variant (T5-v1 for denoising, T5-lm for the LM objective) with the chosen objective or blend, using the AdaFactor optimizer at learning rate 0.0001, before fine-tuning with the objective-aligned templates.

## Related tools

- **Match Task to Objective (MTO) framework** (Supplies the identified objective that determines how adaptation data are prepared and labeled) — https://github.com/puraminy/MTO/
- **T5-v1** (Pre-trained model used with the denoising (Mask-Filling) objective during adaptation) — https://github.com/puraminy/MTO/
- **T5-lm** (T5-v1 derivative pre-trained 100k additional steps on the prefix LM objective, used when the Map-Phrasal objective is selected) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer for the adaptation pre-training and fine-tuning stages at learning rate 0.0001) — https://github.com/puraminy/MTO/

## Examples

```
git clone https://github.com/puraminy/MTO/ && cd MTO  # then follow repo scripts to train the Sentence Splitter on ATOMIC2020 tuples and run adaptation pre-training
```

## Evaluation signals

- Every task category maps to its paper-reported objective: Mask-Filling → denoising (T5-v1) and Map-Phrasal → LM (T5-lm)
- Sentence Splitter is trained on ~5,000 template-generated sentences from ATOMIC2020 tuples and applied to exactly the top 8,000 relevance-scored OMCS sentences
- Each emitted sample's mask/prompt-token structure matches the objective it is assigned to (mask token 'X' placeholders for denoising variants)
- Adaptation runs use AdaFactor optimizer with learning rate 0.0001, consistent with the paper's setup
- Bootstrapped sentences derived from model-generated completions form valid natural sentences usable as new unsupervised data

## Limitations

- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored
- Code availability was only promised for future release in the discussion; the README states the code is 'under refinement and improvement' and requests contact for project materials
- Only the mean of three repeated few-shot runs is reported; random seeds/variance for few-shot QA experiments are not given

## Evidence

- [methods] This model is then applied to natural sentences to split them and provide the necessary data for our unsupervised training.: [evidence span withheld — anchor status: not_found]
