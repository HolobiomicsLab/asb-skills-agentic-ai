---
name: knowledge-graph-tuple-sentence-templating
description: Use when raw knowledge-graph tuples (e.g., ATOMIC2020 head/relation/tail
  triples) must be verbalized into sentence-form training examples for seq2seq fine-tuning
  or prompt-tuning of T5-style encoder-decoder models.
when_to_use_negative:
- Input is already natural-language text (e.g., OMCS corpus sentences) rather than
  structured tuples — use the Sentence Splitter adaptation path instead
- The target model is not an encoder-decoder PLM; the article's results are specific
  to encoder-decoder architectures like T5
- The relation of interest has no template listed in Table A1, so no fixed natural
  phrase exists to ground the verbalization
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: Match Task to Objective (MTO) framework
  role: Provides the template definitions (Table A1) and fine-tuning strategies (Mapping,
    Prompting, masked variants) that the tuple-to-sentence module implements
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder model whose input format the templated sentences are designed
    to align with pre-training objectives
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:60dcb4dffb09bd0ae9c63ced1c14f0f446940d3da249202146f8ecd8bebd9d86
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# knowledge-graph-tuple-sentence-templating

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts ATOMIC2020 (head, relation, tail) tuples into natural-language input-target pairs using the MTO framework's fixed per-relation templates from appendix Table A1, enabling Mapping, Prompting, Masked Mapping, Masked Prompting, and QA Answer Prompting fine-tuning formats.

## When to use

Use when raw knowledge-graph tuples (e.g., ATOMIC2020 head/relation/tail triples) must be verbalized into sentence-form training examples for seq2seq fine-tuning or prompt-tuning of T5-style encoder-decoder models.

## When NOT to use

- Input is already natural-language text (e.g., OMCS corpus sentences) rather than structured tuples — use the Sentence Splitter adaptation path instead
- The target model is not an encoder-decoder PLM; the article's results are specific to encoder-decoder architectures like T5
- The relation of interest has no template listed in Table A1, so no fixed natural phrase exists to ground the verbalization

## Inputs

- ATOMIC2020 (head, relation, tail) tuples for selected relations (At Location, Object Use, Capable Of, Has Property, is Filled By, x Attr, x Intent, x Need, QA)
- Table A1 template definitions mapping each relation to its fixed natural phrase
- CommonsenseQA / Open BookQA question-choice records for the QA Answer Prompting template

## Outputs

- Formatted (input, target) sentence pairs per relation for Mapping and Prompting templates
- Masked variants with the unique 'X' mask-token placeholder replacing the target fragment
- QA Answer Prompting pairs with answer choices first, question at end, correct choice text as target
- Per-relation match flags against Table 2 / Table A1 reference sentences

## How to apply

Load the ATOMIC2020 (head, relation, tail) tuples for the selected relations (At Location, Object Use, Capable Of, Has Property, is Filled By, x Attr, x Intent, x Need, QA). Encode the Table A1 template definitions so each relation label is converted to its fixed natural phrase appended after the head (e.g., x Intent yields 'if PersonX cooks, he intends to satisfy hunger'), with the tail as target. Implement the masked variants by replacing the target fragment with the unique 'X' mask-token placeholder positioned before the target in the output sequence. For QA, implement the Answer Prompting template that places the answer choices first and the question at the end of the input, with the correct choice text as target. Run the module over sample tuples for every relation and compare each generated sentence against the reference examples in Table 2 and Table A1 via substring matching to verify correctness.

## Related tools

- **Match Task to Objective (MTO) framework** (Provides the template definitions (Table A1) and fine-tuning strategies (Mapping, Prompting, masked variants) that the tuple-to-sentence module implements) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder model whose input format the templated sentences are designed to align with pre-training objectives)

## Examples

```
python mto/format_templates.py --dataset ATOMIC2020 --relations xIntent,xNeed,AtLocation --template prompting --mask_token X --validate_against tableA1
```

## Evaluation signals

- Formatted outputs contain the exact reference example sentences from Table 2 / Table A1 (e.g., 'if PersonX cooks, he intends to satisfy hunger' for x Intent), verified via substring matching
- Every selected relation yields at least one formatted input-target pair with a per-relation match flag recorded
- Masked variants place the unique 'X' mask-token placeholder before the target fragment in the output sequence
- QA Answer Prompting inputs have answer choices first and the question at the end, with the correct choice text as target

## Limitations

- Templates are fixed per relation as listed in Table A1; relations without a defined template cannot be verbalized
- Results are specific to the encoder-decoder architecture; alternative architectures were not explored in the article
- Repository code was under refinement with resources available on request (contact pouramini at gmail), limiting direct reuse at publication time

## Evidence

- [methods] Prompting template structures input tuples as natural language prompts: [evidence span withheld — anchor status: not_found]
- [methods] x Intent template example: [evidence span withheld — anchor status: not_found]
- [methods] mask token placeholder: [evidence span withheld — anchor status: not_found]
