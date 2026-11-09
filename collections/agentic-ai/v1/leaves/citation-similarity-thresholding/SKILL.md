---
name: citation-similarity-thresholding
description: Use when a RAG synthesis stage must emit citations restricted to a verifiable
  similarity cutoff, so that every generated answer carries only sources that meet
  a retriever-score threshold rather than hallucinated references.
when_to_use_negative:
- You need generative citation attribution by the LLM itself rather than a deterministic
  retriever-score filter (baseline standalone models like ChatGPT-4o lack this capability).
- The retrieval corpus lacks normalized similarity scores, so a ≥ 0.75 cutoff cannot
  be computed consistently across FAISS and web sources.
- Recall matters more than citation precision — the 0.75 threshold deliberately excludes
  lower-similarity sources from citations.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: AgriIR
  role: Framework whose Stage-6 Citation Insert applies the similarity ≥ 0.75 threshold
    to sources before answer delivery
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: FAISS
  role: Local vector search producing the similarity scores that the 0.75 threshold
    filters on
  repo: https://github.com/facebookresearch/faiss
- name: Llama3.2:3B
  role: Synthesis model whose Stage-5 output receives the thresholded citations
  repo: https://github.com/ollama/ollama
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:d243ec3c30f5b7f706edf465b16cce4ff014ce448b27dc956973c3708e62a637
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Shuvam-Banerji-Seal/AgriIR
  tools:
  - AgriIR
  - FAISS
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# citation-similarity-thresholding

## Summary

A deterministic citation-insertion filter from the AgriIR 6-stage RAG pipeline (Stage-6 Citation Insert) in which only retrieved sources whose similarity score is ≥ 0.75 are inserted as citations into the synthesized answer, yielding grounded, auditable responses from small (1B–3B parameter) local models.

## When to use

Use when a RAG synthesis stage must emit citations restricted to a verifiable similarity cutoff, so that every generated answer carries only sources that meet a retriever-score threshold rather than hallucinated references.

## When NOT to use

- You need generative citation attribution by the LLM itself rather than a deterministic retriever-score filter (baseline standalone models like ChatGPT-4o lack this capability).
- The retrieval corpus lacks normalized similarity scores, so a ≥ 0.75 cutoff cannot be computed consistently across FAISS and web sources.
- Recall matters more than citation precision — the 0.75 threshold deliberately excludes lower-similarity sources from citations.

## Inputs

- Stage-3 candidate sources with similarity scores from FAISS vector DB and DuckDuckGo web retrieval
- Stage-5 synthesized answer (Llama3.2:3B, T=0.2)
- Retrieval configuration (config/enhanced_rag_config.yaml) specifying the 0.75 similarity cutoff and domain allow-list

## Outputs

- JSON response with citations restricted to sources of similarity ≥ 0.75
- Saved cited-answer files (e.g. retargeted_rag_answers) containing deterministic, auditable citations

## How to apply

Run the threshold as the final pipeline stage: after Stage-5 Synthesis (temperature 0.2) produces an answer from Llama3.2:3B, compute similarity scores for all candidate sources retrieved in Stage-3 (parallel retrieval over the FAISS vector DB and DuckDuckGo web search). Insert only sources where similarity ≥ 0.75 as citations in the JSON response before saving cited answers (e.g. to retargeted_rag_answers). Keep the cutoff declarative (config/enhanced_rag_config.yaml), consistent with the framework's design that domain artifacts and scoring heuristics are injected via configuration rather than hard-coded logic. Validation is deterministic: every saved answer must contain citations drawn exclusively from ≥ 0.75-similarity sources, and web sources must come only from the domain allow-list (e.g. extension.org, fao.org, usda.gov).

## Related tools

- **AgriIR** (Framework whose Stage-6 Citation Insert applies the similarity ≥ 0.75 threshold to sources before answer delivery) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **FAISS** (Local vector search producing the similarity scores that the 0.75 threshold filters on) — https://github.com/facebookresearch/faiss
- **Llama3.2:3B** (Synthesis model whose Stage-5 output receives the thresholded citations) — https://github.com/ollama/ollama

## Examples

```
python3 agri_bot_searcher/src/enhanced_web_ui.py  # start full-RAG server; POST /api/query triggers Stage-5 synthesis then Stage-6 citation insert of sources with similarity ≥ 0.75
```

## Evaluation signals

- Each saved answer contains only citations whose source similarity ≥ 0.75 (no sub-threshold citations present).
- All retrieved web sources cited come exclusively from the configured domain allow-list (e.g. extension.org, fao.org, usda.gov for agriculture).
- Citation quality reaches the framework's reported range of 73–84% perfect citations rather than the absent baseline behavior.
- No post-processing code changes were needed to move the threshold between domains — it lives in configuration only.

## Limitations

- The similarity cutoff (0.75) was tuned in the agricultural domain; retargeting to other domains (health, climate) via configuration alone may require re-validating that the cutoff still separates relevant from irrelevant sources.
- No discussion/limitations section text was present in the source document, so additional failure modes (e.g. behavior when no source reaches 0.75) could not be extracted.

## Evidence
