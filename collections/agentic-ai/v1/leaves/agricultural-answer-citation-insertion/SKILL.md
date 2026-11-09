---
name: agricultural-answer-citation-insertion
description: Use when a generated answer must be made auditable by attaching verifiable
  source citations, and you already hold the source chunk pool (local DB + web) alongside
  the synthesized text.
when_to_use_negative:
- You need generative or LLM-judged citation attribution rather than deterministic
  embedding-similarity matching — AgriIR deliberately uses a fixed threshold, not
  model reasoning
- Your answer sentences are not segmentable by a sentence tokenizer (e.g. bullet lists
  or tables) — the method operates per-sentence
- Your source pool is not chunked into two-level (ij) DB/WEB indices, so the [DB_ij]/[WEB_ij]
  token scheme cannot address sources
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3372
- http://edamontology.org/topic_0091
tools:
- name: FAISS
  role: Indexes D∪W chunk embeddings for fast cosine-similarity search against each
    answer sentence
  repo: https://github.com/facebookresearch/faiss
- name: AgriIR
  role: Host framework; Stage-6 'Citation Insert' implements the ≥ 0.75 similarity
    citation-attachment step
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Sentence Transformer
  role: Embeds answer sentences and source chunks so cosine similarity can be computed
    for the 0.75 threshold
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:ec832b19e1bc579eb596a394286331e515bd6de4ed38402a9a26ba3a4c2d9df3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AgriIR
  - FAISS
  - Sentence Transformer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3372
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# agricultural-answer-citation-insertion

## Summary

Stage 6 of the AgriIR pipeline deterministically inserts inline citations ([DB_ij] / [WEB_ij]) into a synthesized answer by matching each sentence against retrieved source chunks with a Sentence Transformer and keeping matches whose cosine similarity is ≥ 0.75.

## When to use

Use when a generated answer must be made auditable by attaching verifiable source citations, and you already hold the source chunk pool (local DB + web) alongside the synthesized text.

## When NOT to use

- You need generative or LLM-judged citation attribution rather than deterministic embedding-similarity matching — AgriIR deliberately uses a fixed threshold, not model reasoning
- Your answer sentences are not segmentable by a sentence tokenizer (e.g. bullet lists or tables) — the method operates per-sentence
- Your source pool is not chunked into two-level (ij) DB/WEB indices, so the [DB_ij]/[WEB_ij] token scheme cannot address sources

## Inputs

- Synthesized answer A (Stage-5 output, plain text)
- Retrieved chunk pool D∪W (local FAISS-indexed DB chunks plus DuckDuckGo web chunks)
- Input JSON binding the answer and chunk pool

## Outputs

- Cited answer A' with inline [DB_ij] / [WEB_ij] tokens
- Citation index mapping each inserted token to its source chunk
- cited_answer JSON object containing both

## How to apply

Load the synthesized answer A and the chunk pool D∪W (FAISS-indexed local database chunks plus DuckDuckGo web chunks) from the input JSON, and split A into sentences with a sentence tokenizer. Embed every sentence and every chunk with a Sentence Transformer encoder, building a FAISS index over D∪W, since cosine similarity on normalized vectors is then a fast dot-product search. For each sentence, compute cosine similarity against all chunks in D∪W and record the best-matching source from each collection. Any sentence whose similarity to a matching chunk is ≥ 0.75 (the paper's 'Citation Insert sources where similarity ≥ 0.75' filter) receives an inline citation token — [DB_ij] for local database chunks, [WEB_ij] for web chunks — using the two-level chunk index ij. Finally assemble the cited answer A' plus a citation index mapping each inserted token back to its source chunk, and write both to cited_answer.

## Related tools

- **FAISS** (Indexes D∪W chunk embeddings for fast cosine-similarity search against each answer sentence) — https://github.com/facebookresearch/faiss
- **AgriIR** (Host framework; Stage-6 'Citation Insert' implements the ≥ 0.75 similarity citation-attachment step) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Sentence Transformer** (Embeds answer sentences and source chunks so cosine similarity can be computed for the 0.75 threshold)

## Examples

```
python stage6_citation_insert.py --input synthesized_answer.json --output cited_answer.json --threshold 0.75
```

## Evaluation signals

- Every inserted citation token matches the pattern [DB_ij] or [WEB_ij] with a valid two-level chunk index present in D∪W
- No sentence with best-matching cosine similarity below 0.75 carries a citation token
- The citation index resolves every token in A' to exactly one source chunk (no dangling references)
- Downstream citation quality is measurable — AgriIR models achieved 73–84% perfect citations on the 191-query benchmark, a capability absent in baseline models
- Each cited sentence's similarity to its matched chunk is retrievable for audit (threshold check ≥ 0.75)

## Limitations

- The 0.75 threshold is a fixed filter step; sentences paraphrasing sources weakly may be left uncited or wrongly matched
- No discussion/limitations section text was present in the source document, so further caveats could not be extracted
- Citation grounding depends on the chunk pool D∪W quality — if the correct source was not retrieved in Stage 3, Stage 6 cannot cite it

## Evidence

- [methods] Stage 6 matches each sentence of the generated answer against source chunks in D ∪ W by embedding both with a Sentence Transformer and collecting as citation matches all chunks whose cosine similarity with the sentence exceeds 0.75: [evidence span withheld — anchor status: not_found]
- [methods] Citation | Insert sources where similarity ≥ 0.75: "Citation | Insert sources where similarity ≥ 0.75"
- [readme] It combines local vector search (FAISS): "AgriIR is a 6-stage retrieval-augmented generation pipeline for agricultural Q&A. It combines local vector search (FAISS)"
- [readme] Faiss ... supports cosine similarity, since this is a dot product on normalized vectors.: [evidence span withheld — anchor status: external_doc]
- [results] AgriIR models excel in citation quality (73-84% perfect citations), a capability absent in baseline models.: "Critically, AgriIR models excel in citation quality (73-84% perfect citations), a capability absent in baseline models."
