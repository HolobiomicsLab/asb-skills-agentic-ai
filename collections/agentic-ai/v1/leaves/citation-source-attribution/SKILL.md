---
name: citation-source-attribution
description: Use when a RAG-synthesized answer must be grounded with deterministic,
  auditable inline citations by embedding its sentences against a known chunk pool
  and thresholding cosine similarity at 0.75.
when_to_use_negative:
- Citations must be generated or justified by the LLM itself rather than matched deterministically
  — AgriIR deliberately uses architectural, non-generative citation insertion
- No retrieved chunk pool exists (the answer cannot be matched against D∪W)
- Expected similarity between answer sentences and sources is far below the 0.75 cutoff,
  e.g. heavily paraphrased or hallucinated content with no true source
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3345
tools:
- name: FAISS
  role: Indexes D∪W chunk embeddings and supports cosine-similarity search of each
    answer sentence against all chunks
  repo: https://github.com/facebookresearch/faiss
- name: AgriIR
  role: Framework hosting the Stage-6 Citation Insert step that performs deterministic
    source attribution
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:2fc42cad09be5eef129f1a6ca7839bd98a0ae9a55b710154741e56a7f85dd7c8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AgriIR
  - FAISS
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# citation-source-attribution

## Summary

AgriIR Stage-6 deterministic citation insertion: each sentence of the synthesized answer A is embedded with a Sentence Transformer and matched by cosine similarity against chunks in D∪W (FAISS-indexed local DB chunks plus DuckDuckGo web chunks), inserting inline tokens [DB_ij] / [WEB_ij] for matches at similarity ≥ 0.75.

## When to use

Use when a RAG-synthesized answer must be grounded with deterministic, auditable inline citations by embedding its sentences against a known chunk pool and thresholding cosine similarity at 0.75.

## When NOT to use

- Citations must be generated or justified by the LLM itself rather than matched deterministically — AgriIR deliberately uses architectural, non-generative citation insertion
- No retrieved chunk pool exists (the answer cannot be matched against D∪W)
- Expected similarity between answer sentences and sources is far below the 0.75 cutoff, e.g. heavily paraphrased or hallucinated content with no true source

## Inputs

- Synthesized answer A (Stage-5 output, loaded from JSON)
- Retrieved chunk pool D∪W (local FAISS-indexed DB chunks plus DuckDuckGo web chunks) from the input JSON
- Sentence Transformer encoder model

## Outputs

- Cited answer A' with inline citation tokens ([DB_ij] / [WEB_ij])
- Citation index mapping each inserted token to its source chunk (written to cited_answer)

## How to apply

Load the synthesized answer A and the retrieved chunk pool D∪W (local FAISS-indexed database chunks plus DuckDuckGo web chunks) from the input JSON, then split A into sentences with a sentence tokenizer. Embed each sentence and every chunk with a Sentence Transformer encoder, building a FAISS index over the D∪W chunk vectors. For each sentence, compute cosine similarity against all chunks in D∪W and record the best-matching source from each collection. Insert the deterministic citation token inline immediately after any sentence whose similarity to a matching chunk is ≥ 0.75 — [DB_ij] for local database chunks and [WEB_ij] for web chunks, using the two-level chunk index ij. Assemble the cited answer A' together with a citation index mapping each inserted token to its source chunk and write both to cited_answer. This deterministic (non-generative) mechanism is what gives AgriIR its 73–84% perfect-citation capability, absent in standalone LLM baselines.

## Related tools

- **FAISS** (Indexes D∪W chunk embeddings and supports cosine-similarity search of each answer sentence against all chunks) — https://github.com/facebookresearch/faiss
- **AgriIR** (Framework hosting the Stage-6 Citation Insert step that performs deterministic source attribution) — https://github.com/Shuvam-Banerji-Seal/AgriIR

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR
```

## Evaluation signals

- Every inserted token is a well-formed two-level index token: [DB_ij] or [WEB_ij], immediately after a sentence
- Each token's similarity to its source chunk is ≥ 0.75 cosine similarity
- The citation index maps every inserted token to an existing chunk in D∪W with no dangling references
- A' is token-for-token identical to A apart from inserted citation tokens (deterministic, non-generative step)
- Citation quality: AgriIR configurations reach 73–84% perfect citations on the 191-query annotated benchmark

## Limitations

- The 0.75 cosine-similarity threshold is a fixed cutoff; borderline paraphrases below it silently receive no citation
- Attribution is only as good as the chunk pool D∪W — sources absent from retrieval can never be cited
- No discussion/limitations section text was extractable from the source document; broader failure modes are not documented

## Evidence
