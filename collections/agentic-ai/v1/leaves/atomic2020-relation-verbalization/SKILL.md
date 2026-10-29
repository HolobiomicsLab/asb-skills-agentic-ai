---
name: atomic2020-relation-verbalization
description: Use when converting ATOMIC2020 relation tuples into sentence-form inputs
  for fine-tuning or prompt-tuning a T5 model, e.g. to produce 'if PersonX cooks,
  he intends to satisfy hunger' style templates.
when_to_use_negative:
- Input is already natural-language OMCS sentences rather than ATOMIC2020 relation
  tuples — the Sentence Splitter direction of the pipeline applies instead
- Tasks are question-answering datasets like CommonsenseQA or Open BookQA where no
  ATOMIC2020 relation tuple exists to verbalize
- A downstream template requires raw symbolic tuples (e.g. Mapping of un verbalized
  entities) rather than natural-language prompt sentences
edam_operation: http://edamontology.org/operation_3083
tools:
- name: Match Task to Objective (MTO) framework
  role: Framework within which the relation-to-natural-language template formatting
    module is implemented for KB completion fine-tuning templates (Mapping, Prompting,
    masked variants)
  repo: https://github.com/puraminy/MTO/
- name: T5
  role: Target encoder-decoder model on which the verbalized templates are consumed
    as fine-tuning inputs
  repo: https://github.com/puraminy/MTO/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s10489-024-05660-2
    title: 'Matching tasks to objectives: Fine-tuning and prompt-tuning strategies
      for encoder-decoder pre-trained language models'
  source_content_hash: sha256:f56230ced940076ecc0834d16b2743e4a357be3cee599af556edc821af9c3ddb
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
  edam_operation: http://edamontology.org/operation_3083
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# atomic2020-relation-verbalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verbalizes ATOMIC2020 (head, relation, tail) tuples into natural-language sentences using fixed per-relation natural phrases (Table A1 of the MTO paper), so that fine-tuning templates like Mapping, Prompting, Masked Mapping and Masked Prompting can be built for KB completion with T5-family encoder-decoder models.

## When to use

Use when converting ATOMIC2020 relation tuples into sentence-form inputs for fine-tuning or prompt-tuning a T5 model, e.g. to produce 'if PersonX cooks, he intends to satisfy hunger' style templates.

## When NOT to use

- Input is already natural-language OMCS sentences rather than ATOMIC2020 relation tuples — the Sentence Splitter direction of the pipeline applies instead
- Tasks are question-answering datasets like CommonsenseQA or Open BookQA where no ATOMIC2020 relation tuple exists to verbalize
- A downstream template requires raw symbolic tuples (e.g. Mapping of un verbalized entities) rather than natural-language prompt sentences

## Inputs

- ATOMIC2020 (head, relation, tail) tuples for relations: At Location, Object Use, Capable Of, Has Property, is Filled By, x Attr, x Intent, x Need, QA
- Table A1 template definitions mapping each relation label to its fixed natural phrase
- QA answer-choice lists for Answer Prompting template inputs

## Outputs

- Formatted (input, target) sentence pairs per relation, e.g. 'if PersonX cooks, he intends to satisfy hunger'
- Masked variants with the unique 'X' mask-token placeholder replacing the target fragment
- QA prompt inputs with answer choices first, question at the end, and correct choice text as target
- Per-relation match flags comparing generated sentences to Table 2 / Table A1 references

## How to apply

Load the (head, relation, tail) tuples for the selected ATOMIC2020 relations (At Location, Object Use, Capable Of, Has Property, is Filled By, x Attr, x Intent, x Need, QA) and encode the Table A1 template definitions so each relation label is replaced by its fixed natural phrase appended after the head. For the Prompting template, structure the tuple as a natural-language prompt; for Masked Mapping and Masked Prompting, replace the target fragment with the unique 'X' mask-token placeholder positioned before the target in the output sequence. For QA relations, use the Answer Prompting template that places answer choices first and the question at the end of the input, with the correct choice text as target. Run the module over sample tuples for every relation to produce input-target pairs, then verify each generated sentence by substring matching against the reference example sentences in Table 2 and Table A1 (e.g. x Intent must yield 'if PersonX cooks, he intends to satisfy hunger').

## Related tools

- **Match Task to Objective (MTO) framework** (Framework within which the relation-to-natural-language template formatting module is implemented for KB completion fine-tuning templates (Mapping, Prompting, masked variants)) — https://github.com/puraminy/MTO/
- **T5** (Target encoder-decoder model on which the verbalized templates are consumed as fine-tuning inputs) — https://github.com/puraminy/MTO/

## Evaluation signals

- Generated sentences for x Intent tuples must contain the exact reference sentence 'if PersonX cooks, he intends to satisfy hunger' via substring matching against Table A1
- Every relation's formatted output must contain the corresponding reference example sentence from Table 2 and Table A1, recorded as a per-relation match flag
- Masked variants must position the unique 'X' mask-token placeholder before the target fragment in the output sequence
- QA Answer Prompting outputs must place answer choices first and the question at the end, with the correct choice text as target
- Each relation label must map deterministically to exactly one fixed natural phrase from Table A1

## Limitations

- Code is under refinement and the README says materials may need to be requested directly from the authors
- Template coverage is restricted to the relations exemplified in Table A1; other ATOMIC2020 relations may lack verified reference phrases
- The 'no usage/docs found' signal indicates the module lacks usage documentation, so exact formatting details may need reverse-engineering from the paper tables

## Evidence

- [methods] structures input tuples as natural language prompts: [evidence span withheld — anchor status: not_found]
- [methods] this template utilizes a unique mask token (referred to as ”X”) as a placeholder for the target element: "this template utilizes a unique mask token (referred to as ”X”) as a placeholder for the target element"
- [methods] we train a model known as the Sentence Splitter using tuples extracted from a knowledge graph: "we train a model known as the Sentence Splitter using tuples extracted from a knowledge graph"
- [abstract] Code is available at https://github.com/puraminy/MTO/: "Code is available at https://github.com/puraminy/MTO/"
- [readme] The code is under refinement and improvement: "The code is under refinement and improvement"
