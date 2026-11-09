---
name: sentence-splitter-training-on-knowledge-graph-tuples
description: Use when preparing task-related unsupervised adaptation data for encoder-decoder
  pre-training (e.g. T5) and natural sentences must be segmented into head-tail parts
  matching knowledge-graph tuple structure.
when_to_use_negative:
- The downstream task is classification or another task type whose benefit from this
  objective-alignment was not investigated by the paper
- A non-encoder-decoder architecture is targeted — results are specific to encoder-decoder
  models like T5
- Clean parallel head/tail supervision already exists, making a learned sentence splitter
  unnecessary
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Sentence Splitter
  role: Model trained on knowledge-graph tuples and applied to natural sentences to
    split them into head/tail-like parts for unsupervised training
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder model on which the split data is used for adaptation via unsupervised
    pre-training
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer for the adaptation/pre-training stage with learning rate 0.0001
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:b2d2cad8ea08d8a5168fe06fd4cb03369be1fcdf17a83050d787fe55ff529e7c
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
  - Sentence Splitter
  - T5
  - T5-base
  - T5-lm
  - T5-v1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Sentence Splitter training on knowledge-graph tuples

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A component of the MTO framework that trains a Sentence Splitter model on knowledge-graph tuples (ATOMIC2020) converted to natural sentences, then applies it to natural-language corpora (OMCS) to segment sentences into head- and tail-like parts. These split sentences supply task-related unsupervised adaptation data aligned with the selected pre-training objective.

## When to use

Use when preparing task-related unsupervised adaptation data for encoder-decoder pre-training (e.g. T5) and natural sentences must be segmented into head-tail parts matching knowledge-graph tuple structure.

## When NOT to use

- The downstream task is classification or another task type whose benefit from this objective-alignment was not investigated by the paper
- A non-encoder-decoder architecture is targeted — results are specific to encoder-decoder models like T5
- Clean parallel head/tail supervision already exists, making a learned sentence splitter unnecessary

## Inputs

- Knowledge-graph tuples/relations from ATOMIC2020 (head, relation, tail)
- Open Mind Common Sense (OMCS) corpus sentences ranked by relevance score (top 8,000)
- Task-category classifier predictions for optional per-sample objective labels ('Mixed' data)

## Outputs

- Trained Sentence Splitter model
- Head- and tail-like segmented sentences for unsupervised adaptation training
- Optionally labeled 'Mixed' data assigning a suitable objective to each sample
- Bootstrapped unsupervised sentences from model-generated completions

## How to apply

Convert knowledge-graph relations from ATOMIC2020 into natural sentences via templates, and train the Sentence Splitter on 5,000 such samples so it learns to segment a sentence into head- and tail-like segments. Apply the trained model to the top 8,000 Open Mind Common Sense (OMCS) sentences ranked by relevance score. Optionally pass each produced sample through a task-category classifier to label it with a suitable objective, forming 'Mixed' data. Feed the split sentences to the adaptation stage, running unsupervised pre-training with the selected objective (denoising for Mask-Filling, prefix LM for Map-Phrasal) or a blend, using T5 with the AdaFactor optimizer at learning rate 0.0001. Model-generated completions can be converted back into new natural sentences to bootstrap additional unsupervised data.

## Related tools

- **Sentence Splitter** (Model trained on knowledge-graph tuples and applied to natural sentences to split them into head/tail-like parts for unsupervised training) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder model on which the split data is used for adaptation via unsupervised pre-training) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer for the adaptation/pre-training stage with learning rate 0.0001)

## Evaluation signals

- Verify the Sentence Splitter was trained on the specified 5,000 template-converted ATOMIC2020 tuple sentences and produces segments resembling the head/tail structure of the original tuples
- Check that exactly the top 8,000 OMCS sentences by relevance score are processed and each yields head- and tail-like parts
- Verify the split output aligns with the selected objective's template structure (denoising masks vs. prefix-LM prompts) before entering the adaptation stage
- Confirm the adaptation stage runs with the AdaFactor optimizer at learning rate 0.0001 using the split unsupervised data
- If 'Mixed' data labels are produced, verify each sample's assigned objective matches its category (Mask-Filling→denoising, Map-Phrasal→LM)

## Limitations

- The repository code is under refinement and is not fully released; contact the authors (pouramini at gmail) to expedite availability of project materials
- No usage/docs for this component are found in the paper section, so exact model architecture and training hyperparameters of the splitter beyond sample count are not documented
- Results are specific to the encoder-decoder architecture; scaling with model parameters and adaptation-data amounts was not investigated

## Evidence

- [methods] This model is then applied to natural sentences to split them and provide the necessary data for our unsupervised training.: [evidence span withheld — anchor status: not_found]
