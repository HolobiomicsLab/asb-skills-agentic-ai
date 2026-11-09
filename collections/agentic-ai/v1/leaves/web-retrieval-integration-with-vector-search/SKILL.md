---
name: web-retrieval-integration-with-vector-search
description: Use when a RAG pipeline must answer domain-specific queries whose local
  knowledge base is incomplete, by fusing local FAISS vector search with live web
  retrieval and measuring the latency cost/benefit of parallelizing retrieval.
when_to_use_negative:
- The local knowledge base alone fully covers the query and web latency is unnecessary
  overhead
- 'The task is generation/evaluation (AgriIR Stages 4-6: Agent Enhancement, Synthesis,
  Citation Insert) rather than retrieval'
- Retrieval must be deterministic/reproducible offline, since live web retrieval results
  change between runs
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FAISS
  role: Local vector similarity search over the AgriIR knowledge-base embeddings for
    each sub-query
  repo: https://github.com/facebookresearch/faiss
- name: AgriIR
  role: 6-stage RAG pipeline whose Stage-3 'Parallel Retrieval' combines FAISS local
    search with web retrieval and ThreadPoolExecutor dispatch
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: AgriIR_dataset
  role: HuggingFace dataset of 15,247 agricultural knowledge entries that backs the
    FAISS index
  repo: https://huggingface.co/datasets/ShuvBan/AgriIR_dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:c085fe2e6321f4f2bc52efdcf7fb010f66e9d28b0caabc46994fc3aa247b673c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AgriIR
  - AgriIR_dataset
  - FAISS
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Web retrieval integration with vector search

## Summary

Hybrid Stage-3 retrieval from the AgriIR framework: each decomposed sub-query is served by local FAISS vector search over the 15,247-entry AgriIR knowledge base combined with web retrieval, and sub-query retrievals are dispatched concurrently via ThreadPoolExecutor to cut wall-clock latency.

## When to use

Use when a RAG pipeline must answer domain-specific queries whose local knowledge base is incomplete, by fusing local FAISS vector search with live web retrieval and measuring the latency cost/benefit of parallelizing retrieval.

## When NOT to use

- The local knowledge base alone fully covers the query and web latency is unnecessary overhead
- The task is generation/evaluation (AgriIR Stages 4-6: Agent Enhancement, Synthesis, Citation Insert) rather than retrieval
- Retrieval must be deterministic/reproducible offline, since live web retrieval results change between runs

## Inputs

- Refined query decomposed into 3-5 sub-queries (four-subquery workload in the reported benchmark)
- AgriIR knowledge base: 15,247 agricultural knowledge entries (huggingface.co/datasets/ShuvBan/AgriIR_dataset) backing the FAISS index
- AgriIR benchmark of 191 annotated agricultural policy queries
- Web search endpoint (e.g. DuckDuckGo retrieval used by AgriIR)

## Outputs

- Merged per-sub-query retrieval results from FAISS local search plus web sources
- retrieval_latency_results.csv with sequential-mode total latency, parallel-mode total latency, per-query latency columns, and speedup ratio

## How to apply

Decompose the refined query into sub-queries (AgriIR Stage-2 targets 3-5 sub-queries per query, e.g. a four-subquery workload) before retrieval. For each sub-query, perform local FAISS vector search against the AgriIR dataset loaded from HuggingFace (ShuvBan/AgriIR_dataset, 15,247 agricultural knowledge entries) and issue a complementary web retrieval call, merging both result sets per sub-query. Run the retrieval stage sequentially first, timestamping start and end of the retrieval block to get baseline wall-clock time; the authors report ~180 s sequential for a four-subquery, 191-query benchmark. Then repeat the same workload with sub-query retrievals dispatched concurrently via ThreadPoolExecutor (AgriIR's Stage-3 'Parallel Retrieval'), reported to reduce total latency to ~50 s (~3.5x speedup). Write per-mode total latency, per-query latencies, and the sequential-to-parallel speedup ratio (expected ratio roughly 0.25-0.31) to a results CSV such as retrieval_latency_results.csv.

## Related tools

- **FAISS** (Local vector similarity search over the AgriIR knowledge-base embeddings for each sub-query) — https://github.com/facebookresearch/faiss
- **AgriIR** (6-stage RAG pipeline whose Stage-3 'Parallel Retrieval' combines FAISS local search with web retrieval and ThreadPoolExecutor dispatch) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **AgriIR_dataset** (HuggingFace dataset of 15,247 agricultural knowledge entries that backs the FAISS index) — https://huggingface.co/datasets/ShuvBan/AgriIR_dataset

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR
```

## Evaluation signals

- retrieval_latency_results.csv exists and contains latency fields for both sequential and parallel modes plus a speedup ratio
- Parallel-mode total latency falls in the expected reduction band: speedup ratio approximately 3.5x (sequential/parallel ratio roughly 0.25-0.31), consistent with the reported ~180 s vs ~50 s
- Each sub-query retrieval returns results from both local FAISS search and the web source before Stage-4 Agent Enhancement
- Per-query latency columns reconcile with the total wall-clock time for each mode (sum in sequential mode; max/aggregate in parallel mode)

## Limitations

- The ~180 s (sequential) vs ~50 s (parallel) figures are as reported in the authors' own testing on a four-subquery workload; absolute latencies depend on hardware and network conditions
- No discussion-section text is available in the source document, so no official limitations or reproducibility gaps are documented ('No changelog found.')
- Hybrid retrieval inherits web-retrieval nondeterminism, which complicates exact reproduction of result sets even when latency bands reproduce

## Evidence
