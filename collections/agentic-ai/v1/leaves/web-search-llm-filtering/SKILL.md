---
name: web-search-llm-filtering
description: Use when a domain-specific RAG pipeline must augment local vector-search
  hits (FAISS over an embedded domain dataset) with fresh web evidence, and raw web
  results need LLM-based relevance filtering before synthesis.
when_to_use_negative:
- The query is fully answerable from the local FAISS-indexed domain corpus and web
  connectivity is unavailable or undesirable
- The task is Stage-6 citation insertion (similarity >= 0.75 threshold) — that is
  a separate, later stage, not part of web corpus collection
- The application is high-resource and can rely on large-scale vector databases and
  GPU-heavy infrastructure instead of lightweight web augmentation
edam_operation: http://edamontology.org/operation_2422
tools:
- name: AgriIR
  role: Framework orchestrating Stage-3 parallel retrieval; web search with LLM filtering
    runs alongside FAISS search and both corpora feed Stage-4 agent enhancement and
    Stage-5 synthesis
  repo: github.com/Shuvam-Banerji-Seal/AgriIR
- name: DuckDuckGo
  role: Web search backend that retrieves candidate pages for the web corpus W
- name: FAISS
  role: Local vector-search branch of the parallel retrieval, producing corpus D as
    the complement to web corpus W
  repo: https://github.com/facebookresearch/faiss
- name: Gemma3:1B
  role: Ollama-served small LLM used in the pipeline stages (refinement/decomposition/filtering)
    under the low-resource design
- name: Ollama
  role: Serves the small open LLMs (Gemma3:1B, Llama3.2:3B, Gemma3:27B) used across
    pipeline stages
  repo: ollama/ollama
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:010988fbb7436bc0d48fff18d74f524777e0640dbdf33f54ddbc2a9816af49f6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - AgriIR
  - DuckDuckGo
  - FAISS
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  - Ollama
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# web-search-llm-filtering

## Summary

Collects web evidence for the AgriIR RAG pipeline by running DuckDuckGo web search in parallel with FAISS local vector search, then using an LLM to filter retrieved pages into a web corpus W that complements the local corpus D. It lets a low-resource, domain-specific RAG framework unify web and database sources as evidence without GPU-heavy infrastructure.

## When to use

Use when a domain-specific RAG pipeline must augment local vector-search hits (FAISS over an embedded domain dataset) with fresh web evidence, and raw web results need LLM-based relevance filtering before synthesis.

## When NOT to use

- The query is fully answerable from the local FAISS-indexed domain corpus and web connectivity is unavailable or undesirable
- The task is Stage-6 citation insertion (similarity >= 0.75 threshold) — that is a separate, later stage, not part of web corpus collection
- The application is high-resource and can rely on large-scale vector databases and GPU-heavy infrastructure instead of lightweight web augmentation

## Inputs

- 3-5 decomposed sub-queries (JSON list from Stage-2 output)
- AgriIR dataset (15,247 agricultural knowledge entries, HuggingFace: ShuvBan/AgriIR_dataset) embedded in a FAISS index
- Domain-agent catalogue (max_agents: 6 keyword/role entries)

## Outputs

- Local corpus D (FAISS retrieval results over the AgriIR dataset)
- Web corpus W (LLM-filtered DuckDuckGo results)
- Both corpora written into the synthesized_answer JSON file alongside the final answer A

## How to apply

Run this as Stage-3 'Parallel Retrieval' of the AgriIR pipeline, immediately after Stage-2 sub-query decomposition (Gemma3:1B at temperature 0.5) produces 3-5 sub-queries. Execute two parallel retrieval branches: FAISS vector search over the AgriIR dataset (15,247 agricultural knowledge entries embedded with Qwen3-Embedding-8B) yields the local corpus D, while DuckDuckGo web search over the same sub-queries yields candidate web pages. Apply LLM filtering to the web candidates to discard irrelevant or low-quality results, producing the web corpus W. Merge D and W downstream: Stage-4 injects keywords from the domain-agent catalogue (max_agents: 6, e.g. crop_specialist, disease_expert, climate_researcher), and Stage-5 synthesizes the answer at temperature 0.2. The rationale, per the paper, is architectural reliability through domain specialization rather than model scale — web retrieval supplies coverage the local corpus lacks while filtering keeps evidence tractable for small (1B-3B parameter) models.

## Related tools

- **AgriIR** (Framework orchestrating Stage-3 parallel retrieval; web search with LLM filtering runs alongside FAISS search and both corpora feed Stage-4 agent enhancement and Stage-5 synthesis) — github.com/Shuvam-Banerji-Seal/AgriIR
- **DuckDuckGo** (Web search backend that retrieves candidate pages for the web corpus W)
- **FAISS** (Local vector-search branch of the parallel retrieval, producing corpus D as the complement to web corpus W) — https://github.com/facebookresearch/faiss
- **Gemma3:1B** (Ollama-served small LLM used in the pipeline stages (refinement/decomposition/filtering) under the low-resource design)
- **Ollama** (Serves the small open LLMs (Gemma3:1B, Llama3.2:3B, Gemma3:27B) used across pipeline stages) — ollama/ollama

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR
```

## Evaluation signals

- The synthesized_answer output file exists, parses as JSON, and contains a non-empty answer field A plus both retrieved corpora D and W
- Web corpus W contains only documents that survived LLM relevance filtering (no raw unfiltered DuckDuckGo dumps)
- Sub-queries from Stage-2 number 3-5 per refined query, matching the decomposition configuration
- Domain-agent keyword injection respects the max_agents: 6 configuration cap
- No citation insertion (similarity >= 0.75 threshold) is applied at this retrieval stage — citations belong to Stage-6

## Limitations

- Web retrieval quality depends on DuckDuckGo availability and LLM filter reliability; filtered web evidence is not validated by a similarity threshold at this stage (citation filtering at >= 0.75 happens later in Stage-6)
- The framework is designed for low-resource, domain-sensitive settings (Indian agriculture: linguistic diversity, fragmented data ecosystems, limited compute) — it is not a substitute for high-resource, GPU-heavy RAG deployments
- The article's discussion section is absent from the extracted source, so no documented failure modes or reproducibility gaps beyond the above are available

## Evidence

- [intro] Configurable modular stages incl. retrieval: [evidence span withheld — anchor status: not_found]
