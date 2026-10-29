---
name: commonsense-corpus-sentence-splitting
description: 'Use when preparing task-related adaptation data for unsupervised training
  of an encoder-decoder PLM: natural-language sentences must be split into head-like
  and tail-like parts to enable denoising and LM objectives.'
when_to_use_negative:
- The task is classification or another non-generation task type where impact of this
  objective has not been investigated (results are specific to generation/QA with
  encoder-decoder architectures).
- You already have naturally occurring head/tail sentence pairs and do not need a
  learned splitter trained on knowledge graph tuples.
- The codebase is under refinement — per the README the code is 'under refinement
  and improvement', so verify availability or contact the authors before relying on
  released scripts.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Match Task to Objective (MTO) framework
  role: 'Provides the adaptation-data strategy: identifies words/phrases excludable
    from sentences so denoising and LM objectives can be trained concurrently on the
    Mixed data'
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Backbone seq2seq model for training the Sentence Splitter to segment natural
    sentences into head-like and tail-like parts
  repo: https://github.com/puraminy/MTO/
- name: Ada Factor optimizer
  role: Optimizer used for training (learning rate 0.0001) during the fine-tuning/adaptation
    pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:3998d870314842e7aeec8325b48e049afcbcb3394bc77f3578ab58d12f7ee69e
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# commonsense-corpus-sentence-splitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a seq2seq (T5-based) Sentence Splitter on knowledge-graph tuples converted to natural sentences, then applies it to natural-language commonsense sentences (e.g., top-ranked OMCS corpus sentences) to split each into two parts, yielding data that supports concurrent unsupervised training on denoising and LM objectives (Mixed adaptation data).

## When to use

Use when preparing task-related adaptation data for unsupervised training of an encoder-decoder PLM: natural-language sentences must be split into head-like and tail-like parts to enable denoising and LM objectives.

## When NOT to use

- The task is classification or another non-generation task type where impact of this objective has not been investigated (results are specific to generation/QA with encoder-decoder architectures).
- You already have naturally occurring head/tail sentence pairs and do not need a learned splitter trained on knowledge graph tuples.
- The codebase is under refinement — per the README the code is 'under refinement and improvement', so verify availability or contact the authors before relying on released scripts.

## Inputs

- 5,000 (head, relation, tail) tuple samples from selected ATOMIC2020 tasks
- Top 8,000 OMCS corpus sentences ranked by relevance score (from >700,000 OMCS sentences)
- Paper's tuple-to-natural-sentence templates (tail as target)

## Outputs

- Trained T5-based Sentence Splitter model
- mixed_adaptation_data file: ~8,000 records, one per OMCS sentence, each with two split parts plus an objective label ('denoising' or 'LM')

## How to apply

Load 5,000 (head, relation, tail) tuple samples from selected ATOMIC2020 tasks and the top 8,000 OMCS corpus sentences ranked by relevance score. Convert the knowledge graph tuples into natural sentences using the paper's templates (tail as target) and train a T5-based seq2seq Sentence Splitter to segment natural sentences into head-like and tail-like parts. Apply the trained splitter to the 8,000 OMCS sentences so each is split into two parts, designating the second part as the target. Then run the trained task category classifier on each split sample to add a label specifying whether the sample suits the denoising objective (second part replaced by a mask token) or the LM objective (no mask tokens), producing the 'Mixed' adaptation data serialized with both split parts and objective labels. Rationale: this creates task-related data for extended unsupervised training aligned with the identified objective, per the MTO framework.

## Related tools

- **Match Task to Objective (MTO) framework** (Provides the adaptation-data strategy: identifies words/phrases excludable from sentences so denoising and LM objectives can be trained concurrently on the Mixed data) — https://github.com/puraminy/MTO/
- **T5** (Backbone seq2seq model for training the Sentence Splitter to segment natural sentences into head-like and tail-like parts) — https://github.com/puraminy/MTO/
- **Ada Factor optimizer** (Optimizer used for training (learning rate 0.0001) during the fine-tuning/adaptation pipeline)

## Examples

```
python mto/sentence_splitter.py --tuples atomic2020_samples_5000. --omcs omcs_top8000.l --out mixed_adaptation_data
```

## Evaluation signals

- mixed_adaptation_data file exists after the run
- File contains one record per OMCS sentence (~8,000 rows)
- Each record has two split parts plus an objective label of 'denoising' or 'LM'
- Split boundaries form plausible head/tail fragments (llm_judge check)
- Classifier-assigned labels are consistent with sample structure: denoising samples imply a maskable second part, LM samples contain no mask tokens

## Limitations

- The approach was evaluated only on encoder-decoder architectures; alternative architectures were not explored.
- Code release was promised for the future and, per the README, is still 'under refinement and improvement'; project materials may need to be requested from the authors.
- Scaling with model parameters and varying amounts of adaptation-phase training data was not investigated.

## Evidence

- [methods] we train a model known as the Sentence Splitter using tuples extracted from a knowledge graph: "we train a model known as the Sentence Splitter using tuples extracted from a knowledge graph"
- [methods] apply splitter to natural sentences: [evidence span withheld — anchor status: not_found]
- [methods] optional classifier labels objective: "Optionally, we can use a classifier to determine the suitable objective for each produced sample"
- [abstract] automated adaptation data preparation: "This framework offers automated methods to prepare task-related data for adaptation through unsupervised training, based on the identified objective."
- [methods] OMCS corpus description: "we relied on the Open Mind Common Sense (OMCS) corpus, a large-scale collection of commonsense knowledge consisting of over 700,000 sentences in natural language"
- [readme] README code status: "The code is under refinement and improvement"
