---
name: sentence-embedding-similarity-scoring
description: Use when a synthesized (RAG) answer must be grounded with deterministic
  inline citations by matching each answer sentence against retrieved source chunks
  via embedding cosine similarity.
when_to_use_negative:
- The answer text has not yet been synthesized (Stage 5 output unavailable), so there
  are no sentences to match against source chunks
- No retrieved chunk pool (D∪W) exists — e.g., the retrieval stage produced no DB
  or web chunks — since every match would fall below the 0.75 threshold
- You need generative/LLM-based citation attribution rather than deterministic embedding-threshold
  matching; this skill is deliberately non-generative
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: FAISS
  role: Indexes D∪W chunk embeddings for cosine-similarity search against each answer
    sentence
  repo: https://github.com/facebookresearch/faiss
- name: AgriIR
  role: Framework hosting Stage-6 Citation Insert; repository contains the deterministic
    citation-insertion implementation
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Ollama
  role: Serves the local LLM whose synthesized answer sentences are embedded and matched
    (upstream of this stage)
  repo: https://github.com/ollama/ollama
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:43f6e9902a5d399f07d2016f5b888b3e1d734b0ea0f61538f4ea0d16a8c3d504
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AgriIR
  - FAISS
  - Ollama
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sentence-embedding-similarity-scoring

## Summary

Scores cosine similarity between Sentence Transformer embeddings of generated-answer sentences and source chunks (FAISS-indexed) to deterministically attach citations, inserting source tokens inline only where similarity ≥ 0.75.

## When to use

Use when a synthesized (RAG) answer must be grounded with deterministic inline citations by matching each answer sentence against retrieved source chunks via embedding cosine similarity.

## When NOT to use

- The answer text has not yet been synthesized (Stage 5 output unavailable), so there are no sentences to match against source chunks
- No retrieved chunk pool (D∪W) exists — e.g., the retrieval stage produced no DB or web chunks — since every match would fall below the 0.75 threshold
- You need generative/LLM-based citation attribution rather than deterministic embedding-threshold matching; this skill is deliberately non-generative

## Inputs

- Synthesized answer A (text) from the AgriIR synthesis stage, provided in an input JSON
- Retrieved chunk pool D∪W: local FAISS-indexed DB chunks plus DuckDuckGo web chunks (input JSON)
- Sentence Transformer encoder model

## Outputs

- Cited answer A' with inline citation tokens ([DB_ij] / [WEB_ij]) inserted after qualifying sentences
- Citation index mapping each inserted token to its source chunk (written to cited_answer)

## How to apply

Load the synthesized answer A and the retrieved chunk pool D∪W (local FAISS-indexed database chunks plus DuckDuckGo web chunks) from the input JSON, then split A into sentences with a sentence tokenizer. Embed each sentence and every chunk with a Sentence Transformer encoder, building FAISS-index vectors for D∪W. For each sentence, compute cosine similarity against all chunks in D∪W and record the best-matching source from each collection. Insert the deterministic citation token inline immediately after any sentence whose similarity to a matching chunk is ≥ 0.75 — [DB_ij] for local database chunks and [WEB_ij] for web chunks, using the two-level chunk index ij. Finally assemble the cited answer A' plus a citation index mapping each inserted token to its source chunk and write both to cited_answer.

## Related tools

- **FAISS** (Indexes D∪W chunk embeddings for cosine-similarity search against each answer sentence) — https://github.com/facebookresearch/faiss
- **AgriIR** (Framework hosting Stage-6 Citation Insert; repository contains the deterministic citation-insertion implementation) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Ollama** (Serves the local LLM whose synthesized answer sentences are embedded and matched (upstream of this stage)) — https://github.com/ollama/ollama

## Examples

```
python stage6_citation_insert.py --input answer_and_chunks. --threshold 0.75 --output cited_answer
```

## Evaluation signals

- Every inserted citation token corresponds to a chunk whose cosine similarity with its sentence is ≥ 0.75 (no sub-threshold tokens)
- Token prefixes are correct by source collection: [DB_ij] only for local database chunks, [WEB_ij] only for web chunks, with valid two-level chunk indices ij
- The citation index is complete: every token in A' maps to exactly one source chunk in D∪W
- All sentences in A' are present in the original synthesized answer A (insertion-only transformation, no sentence rewriting)
- End-to-end: AgriIR models achieve 73-84% perfect citations on the 191-query benchmark, so a run deviating far from this citation-quality range warrants inspection

## Limitations

- The 0.75 threshold is a fixed cutoff: sentences paraphrased far from their source chunk embedding fall below it and receive no citation
- Matching is deterministic and embedding-based only — it cannot verify factual consistency beyond surface semantic similarity to the chunk
- No discussion-section limitations were extracted from the source document (no changelog/limitations text present)

## Evidence

- [methods] Stage 6 matches each sentence of the generated answer against source chunks in D ∪ W by embedding both with a Sentence Transformer and collecting as citation matches all chunks whose cosine similarity with the sentence exceeds 0.75: [evidence span withheld — anchor status: not_found]
- [readme] It supports cosine similarity, since this is a dot product on normalized vectors.: [evidence span withheld — anchor status: not_found]
