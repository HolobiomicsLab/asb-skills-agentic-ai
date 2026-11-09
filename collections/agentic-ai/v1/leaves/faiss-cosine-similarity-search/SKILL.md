---
name: faiss-cosine-similarity-search
description: Use when answer sentences must be deterministically matched back to source
  chunks (local DB or web) via embedding similarity, e.g. to insert inline citations
  like [DB_ij]/[WEB_ij] in a RAG pipeline.
when_to_use_negative:
- You need generative or learned citation placement — this mechanism is purely deterministic
  (threshold at similarity ≥ 0.75), not model-driven
- Chunks are not embedding-representable text pairs, or the corpus is too large for
  exact/approximate FAISS indexing on available compute (AgriIR targets low-resource,
  non-GPU-heavy settings)
tools:
- name: FAISS
  role: Vector index over chunk embeddings supporting cosine similarity (dot product
    on normalized vectors) for sentence-to-chunk matching
  repo: https://github.com/facebookresearch/faiss
- name: Sentence Transformer
  role: Encodes answer sentences and source chunks into dense vectors before FAISS
    cosine search
- name: AgriIR
  role: Pipeline in which this FAISS-based similarity search constitutes Stage-6 Citation
    Insert over the D∪W chunk pool
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:cda560315c307f078ba744337679c5f89c14e3420927a053a087248e22584633
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AgriIR
  - FAISS
  - Sentence Transformer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# faiss-cosine-similarity-search

## Summary

Build a FAISS dense-vector index over Sentence Transformer embeddings of source chunks and query it with cosine (inner-product on normalized vectors) similarity to match answer sentences to their grounding sources, as used in AgriIR's Stage-6 deterministic citation insertion with a 0.75 threshold.

## When to use

Use when answer sentences must be deterministically matched back to source chunks (local DB or web) via embedding similarity, e.g. to insert inline citations like [DB_ij]/[WEB_ij] in a RAG pipeline.

## When NOT to use

- You need generative or learned citation placement — this mechanism is purely deterministic (threshold at similarity ≥ 0.75), not model-driven
- Chunks are not embedding-representable text pairs, or the corpus is too large for exact/approximate FAISS indexing on available compute (AgriIR targets low-resource, non-GPU-heavy settings)

## Inputs

- Synthesized answer A (text) from Stage-5 synthesis of the AgriIR pipeline
- Retrieved chunk pool D∪W: local FAISS-indexed database chunks plus DuckDuckGo web chunks
- Sentence Transformer embedding model

## Outputs

- Cited answer A' with inline citation tokens [DB_ij] / [WEB_ij] inserted after sentences with similarity ≥ 0.75
- Citation index mapping each inserted token to its source chunk, written to cited_answer

## How to apply

Load the generated answer A and the chunk pool D∪W (FAISS-indexed local database chunks plus web chunks) and split A into sentences with a sentence tokenizer. Embed each sentence and every chunk with a Sentence Transformer encoder and build a FAISS index over the chunk vectors in D∪W; since FAISS supports cosine similarity as dot product on normalized vectors, normalize embeddings and use an inner-product index. For each sentence, compute cosine similarity against all chunks and record the best-matching source from each collection (local DB and web). Insert the deterministic citation token ([DB_ij] for local chunks, [WEB_ij] for web chunks, using the two-level chunk index ij) inline immediately after any sentence whose similarity to a matching chunk is ≥ 0.75. Finally assemble the cited answer A' and a citation index mapping each token to its source chunk.

## Related tools

- **FAISS** (Vector index over chunk embeddings supporting cosine similarity (dot product on normalized vectors) for sentence-to-chunk matching) — https://github.com/facebookresearch/faiss
- **Sentence Transformer** (Encodes answer sentences and source chunks into dense vectors before FAISS cosine search)
- **AgriIR** (Pipeline in which this FAISS-based similarity search constitutes Stage-6 Citation Insert over the D∪W chunk pool) — https://github.com/Shuvam-Banerji-Seal/AgriIR

## Examples

```
import faiss; index = faiss.IndexFlatIP(emb_dim); index.add(normalized_chunk_vectors); D, I = index.search(sentence_vectors, k)
```

## Evaluation signals

- Every inserted citation token corresponds to a chunk whose cosine similarity with the following sentence is ≥ 0.75
- Both cited answer A' and its citation index are produced and written to cited_answer; each token in A' resolves to exactly one source chunk
- Token namespace is correct: [DB_ij] for local FAISS-indexed DB chunks and [WEB_ij] for web chunks, with two-level chunk indices
- AgriIR-level sanity check: the framework's configurations achieve 73-84% perfect citations, so citation quality can be benchmarked against human-annotated queries

## Limitations

- The 0.75 cosine threshold is a hard cutoff — sentences below it receive no citation, so genuinely grounded paraphrases with weaker embedding similarity may be left uncited
- Only the best-matching source from each collection is recorded per sentence; multiple supporting chunks above threshold are not all surfaced
- No discussion/limitations section text was available in the source document, so additional failure modes (e.g. tokenizer errors, embedding-model domain shift) could not be extracted

## Evidence

- [methods] Stage-6 matches each sentence of the generated answer against source chunks in D ∪ W by embedding both with a Sentence Transformer and collecting as citation matches all chunks whose cosine similarity with the sentence exceeds 0.75: [evidence span withheld — anchor status: not_found]
- [other] FAISS cosine similarity support: [evidence span withheld — anchor status: external_doc]
