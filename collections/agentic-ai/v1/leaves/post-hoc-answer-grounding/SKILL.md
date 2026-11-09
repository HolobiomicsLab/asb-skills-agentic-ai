---
name: post-hoc-answer-grounding
description: Use when a generated answer must be post-hoc grounded against a pool
  of retrieved source chunks (e.g., FAISS-indexed local DB chunks plus web chunks),
  and inline, deterministic citation tokens need to be inserted per sentence.
when_to_use_negative:
- The answer is a free-form response to general-purpose (non-domain) queries where
  the retrieved pool D∪W is irrelevant or empty — sentences will fail the 0.75 similarity
  cutoff and no citations can be inserted.
- You need citations that are absent in baseline standalone LLM systems; this skill
  only works with an explicit retrievable chunk pool with trackable chunk indices,
  not with parametric model memory.
- Input is raw text without sentence boundaries or the chunk pool lacks the two-level
  index structure (ij) required to emit [DB_ij]/[WEB_ij] tokens.
edam_operation: http://edamontology.org/operation_0435
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FAISS
  role: Vector index over the chunk pool D∪W used for cosine-similarity search of
    each answer sentence against source chunks
  repo: https://github.com/facebookresearch/faiss
- name: AgriIR
  role: Framework hosting Stage-6 'Citation Insert', which applies the ≥ 0.75 similarity
    filter and emits [DB_ij]/[WEB_ij] citation tokens
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Sentence Transformer
  role: Encoder used to embed answer sentences and source chunks into FAISS vectors
    for similarity computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:403cb0bab6240f88354f5f791c2e9f8367069b71c68da63d72455b84b19f6c1a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AgriIR
  - FAISS
  - Sentence Transformer
  edam_operation: http://edamontology.org/operation_0435
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# post-hoc-answer-grounding

## Summary

Deterministically grounds each sentence of a synthesized RAG answer to source chunks by embedding both with a Sentence Transformer and matching via FAISS cosine similarity (threshold ≥ 0.75), inserting inline citation tokens ([DB_ij] / [WEB_ij]) after matched sentences. This is Stage 6 ('Citation Insert') of the AgriIR pipeline, responsible for its high citation quality.

## When to use

Use when a generated answer must be post-hoc grounded against a pool of retrieved source chunks (e.g., FAISS-indexed local DB chunks plus web chunks), and inline, deterministic citation tokens need to be inserted per sentence.

## When NOT to use

- The answer is a free-form response to general-purpose (non-domain) queries where the retrieved pool D∪W is irrelevant or empty — sentences will fail the 0.75 similarity cutoff and no citations can be inserted.
- You need citations that are absent in baseline standalone LLM systems; this skill only works with an explicit retrievable chunk pool with trackable chunk indices, not with parametric model memory.
- Input is raw text without sentence boundaries or the chunk pool lacks the two-level index structure (ij) required to emit [DB_ij]/[WEB_ij] tokens.

## Inputs

- Synthesized answer A (Stage-5 output, JSON)
- Retrieved chunk pool D∪W: local FAISS-indexed database chunks and web chunks (e.g., from DuckDuckGo retrieval) with two-level chunk indices ij
- Sentence Transformer encoder model for embedding sentences and chunks

## Outputs

- Cited answer A' with inline citation tokens ([DB_ij] or [WEB_ij]) inserted after matched sentences
- Citation index mapping each inserted citation token to its source chunk (written to cited_answer)

## How to apply

Load the synthesized answer A and the retrieved chunk pool D∪W (local FAISS-indexed database chunks plus DuckDuckGo web chunks) from the input JSON. Split A into sentences with a sentence tokenizer, then embed each sentence and every chunk with a Sentence Transformer encoder, building FAISS-index vectors for D∪W. For each sentence, compute cosine similarity against all chunks and record the best-matching source from each collection. Insert the deterministic citation token inline immediately after any sentence whose similarity to a matching chunk is ≥ 0.75 — [DB_ij] for local database chunks and [WEB_ij] for web chunks, using the two-level chunk index ij. The threshold ≥ 0.75 is the filter step ('Citation | Insert sources where similarity ≥ 0.75') that makes the citation insertion deterministic rather than model-dependent. Assemble the cited answer A' plus a citation index mapping each inserted token to its source chunk, and write both to cited_answer.

## Related tools

- **FAISS** (Vector index over the chunk pool D∪W used for cosine-similarity search of each answer sentence against source chunks) — https://github.com/facebookresearch/faiss
- **AgriIR** (Framework hosting Stage-6 'Citation Insert', which applies the ≥ 0.75 similarity filter and emits [DB_ij]/[WEB_ij] citation tokens) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Sentence Transformer** (Encoder used to embed answer sentences and source chunks into FAISS vectors for similarity computation)

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR
```

## Evaluation signals

- Every inserted citation token's mapped chunk has cosine similarity ≥ 0.75 with the sentence it follows (the article's stated filter).
- Token format is deterministic: [DB_ij] for local database chunks and [WEB_ij] for web chunks, with a two-level chunk index ij.
- A citation index is emitted alongside A' that maps each token to exactly one source chunk from D∪W.
- AgriIR models achieve 73–84% perfect citations on the 191-query benchmark; a correctly grounded run should show high perfect-citation rates, a capability absent in baseline models.
- Each answer sentence either receives exactly the best-matching source token from each collection or no token if no chunk exceeds 0.75.

## Limitations

- The 0.75 cosine similarity threshold is fixed by configuration; sentences paraphrased far from their sources (or from chunks not in D∪W) will remain uncited.
- Citation quality depends on the coverage of the retrieved pool D∪W; no citation can be inserted for claims with no supporting chunk.
- No limitations or reproducibility gaps are discussed in the article: 'No discussion section text present in the source document; no limitations, missing artifacts, or reproducibility gaps could be extracted'.

## Evidence
