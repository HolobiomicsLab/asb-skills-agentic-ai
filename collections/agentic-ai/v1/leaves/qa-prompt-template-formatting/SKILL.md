---
name: qa-prompt-template-formatting
description: Use when converting ATOMIC2020 (head, relation, tail) tuples or CommonsenseQA/Open
  BookQA items into T5 fine-tuning templates (Mapping, Prompting, Masked variants,
  Answer Prompting) whose outputs must match Table A1 reference sentences.
when_to_use_negative:
- When the goal is unsupervised adaptation data generation (Sentence Splitter / bootstrapping
  of natural sentences) rather than supervised fine-tuning template construction
- When input is plain natural-language text (e.g., OMCS corpus sentences) rather than
  structured relation tuples or multiple-choice QA items
- When evaluating prompt-tuning configurations (PrePT/PostPT soft prompt placement)
  rather than discrete natural-language template formatting
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Match Task to Objective (MTO) framework
  role: Framework hosting the template module; defines the fine-tuning templates (Mapping,
    Prompting, masked variants, Answer Prompting) that align with pre-training objectives
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Encoder-decoder model the formatted input-target pairs are fed to for fine-tuning
  repo: https://github.com/puraminy/MTO/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:61cca497867ebcd47bff6884ad2bdbb43779644f3e4c5679361eff2f6b10d662
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
  - Match Task to Objective (MTO) framework
  - T5
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# qa-prompt-template-formatting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A formatting module from the MTO framework that converts ATOMIC2020 relation tuples and CommonsenseQA/Open BookQA items into encoder-decoder input-target pairs using fixed natural-language phrases per relation (Table A1), including masked variants and the Answer Prompting template that places answer choices first and the question last.

## When to use

Use when converting ATOMIC2020 (head, relation, tail) tuples or CommonsenseQA/Open BookQA items into T5 fine-tuning templates (Mapping, Prompting, Masked variants, Answer Prompting) whose outputs must match Table A1 reference sentences.

## When NOT to use

- When the goal is unsupervised adaptation data generation (Sentence Splitter / bootstrapping of natural sentences) rather than supervised fine-tuning template construction
- When input is plain natural-language text (e.g., OMCS corpus sentences) rather than structured relation tuples or multiple-choice QA items
- When evaluating prompt-tuning configurations (PrePT/PostPT soft prompt placement) rather than discrete natural-language template formatting

## Inputs

- ATOMIC2020 (head, relation, tail) tuples for relations: At Location, Object Use, Capable Of, Has Property, is Filled By, x Attr, x Intent, x Need, QA
- CommonsenseQA and Open BookQA items (question, answer choices, correct choice)
- Table A1 template definitions mapping each relation label to its fixed natural phrase

## Outputs

- Formatted input-target pairs per template variant (Mapping, Prompting, Masked Mapping, Masked Prompting)
- Answer Prompting inputs with choices-first/question-last ordering and correct-choice targets
- Per-relation match flags from substring comparison against Table 2 and Table A1 reference sentences

## How to apply

Load the ATOMIC2020 (head, relation, tail) tuples for the selected relations (At Location, Object Use, Capable Of, Has Property, is Filled By, x Attr, x Intent, x Need, QA) and encode the Table A1 template definitions so each relation label is replaced by its fixed natural phrase appended after the head (e.g., x Intent yields 'if PersonX cooks, he intends to satisfy hunger'). For masked variants (Masked Mapping, Masked Prompting), replace the target fragment with the unique 'X' mask-token placeholder positioned before the target in the output sequence, enabling a denoising objective. For QA, implement the Answer Prompting template where the answer choices appear first and the question sits at the end of the input, with the correct choice text as target; Masked Answer Prompting swaps in masked tokens for the denoising objective. Run the module over sample tuples for every relation to emit formatted input-target pairs, then validate by substring-matching each generated sentence against the reference examples in Table 2 and Table A1, recording a per-relation match flag and accepting only on exact matches.

## Related tools

- **Match Task to Objective (MTO) framework** (Framework hosting the template module; defines the fine-tuning templates (Mapping, Prompting, masked variants, Answer Prompting) that align with pre-training objectives) — https://github.com/puraminy/MTO/
- **T5** (Encoder-decoder model the formatted input-target pairs are fed to for fine-tuning) — https://github.com/puraminy/MTO/

## Evaluation signals

- Substring match: generated x Intent sentence equals the reference 'if PersonX cooks, he intends to satisfy hunger' from Table A1
- Every relation's formatted output contains its exact Table A1 reference sentence, recorded as a per-relation match flag
- Masked variants contain the unique 'X' mask-token placeholder positioned before the target fragment in the output sequence
- Answer Prompting inputs place answer choices first and the question at the end, with the correct choice text as the target
- All nine listed relations (At Location, Object Use, Capable Of, Has Property, is Filled By, x Attr, x Intent, x Need, QA) yield valid input-target pairs

## Limitations

- Code is under refinement and improvement; project materials must be requested from the authors (pouramini at gmail) per the README
- Templates and reference phrases are fixed per relation as listed in Table A1, so new relations require updating the table before formatting
- Results are specific to the encoder-decoder architecture; the template design was not evaluated on alternative architectures

## Evidence
