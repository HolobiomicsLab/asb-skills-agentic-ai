---
name: parallel-multi-source-retrieval-orchestration
description: Use when a RAG pipeline's Stage-2 decomposition yields 3-5 sub-queries
  per query and sequential execution of local FAISS search plus web retrieval per
  sub-query dominates end-to-end latency.
when_to_use_negative:
- The pipeline has only one sub-query per query, leaving no concurrency to exploit
  across Stage-3 retrievals
- Retrieval is dominated by a single serial component (e.g., a rate-limited web API)
  that cannot be parallelized with threads
- You need exact reproducibility of retrieval order, since concurrent dispatch makes
  per-sub-query completion order nondeterministic
edam_operation: http://edamontology.org/operation_0306
tools:
- name: FAISS
  role: Local vector similarity search backend executed per sub-query within the parallel
    retrieval stage
  repo: https://github.com/facebookresearch/faiss
- name: AgriIR
  role: Six-stage RAG framework whose Stage-3 Parallel Retrieval step is being orchestrated;
    provides the FAISS index and web-retrieval combination
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:3ccc1475bb59071b012014d74e3a91158a43217844ed67ea0b05a3be3eb9ec1b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AgriIR
  - FAISS
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Parallel Multi-Source Retrieval Orchestration

## Summary

Reduces AgriIR Stage-3 retrieval latency by dispatching each decomposed sub-query's retrieval (local FAISS vector search over the 15,247-entry AgriIR dataset plus DuckDuckGo web retrieval) concurrently via a ThreadPoolExecutor, cutting reported wall-clock time from ~180 s sequential to ~50 s (~3.5x speedup) for a four-subquery workload.

## When to use

Use when a RAG pipeline's Stage-2 decomposition yields 3-5 sub-queries per query and sequential execution of local FAISS search plus web retrieval per sub-query dominates end-to-end latency.

## When NOT to use

- The pipeline has only one sub-query per query, leaving no concurrency to exploit across Stage-3 retrievals
- Retrieval is dominated by a single serial component (e.g., a rate-limited web API) that cannot be parallelized with threads
- You need exact reproducibility of retrieval order, since concurrent dispatch makes per-sub-query completion order nondeterministic

## Inputs

- 191-query agricultural policy benchmark (AgriIR annotated queries)
- AgriIR knowledge base of 15,247 agricultural entries backing the FAISS index (huggingface.co/datasets/ShuvBan/AgriIR_dataset)
- Stage-2 decomposed sub-queries (3-5 per query, generated at T=0.5)

## Outputs

- retrieval_latency_results.csv with per-mode total latency, per-query latency columns, and the sequential-to-parallel speedup ratio
- Retrieved documents per sub-query from FAISS local vector search and DuckDuckGo web retrieval, passed to Stage-4 Agent Enhancement

## How to apply

After Stage-2 sub-query decomposition (T=0.5) produces the expected 3-5 sub-queries per query, execute Stage-3 retrieval by submitting each sub-query to a ThreadPoolExecutor so that local FAISS vector search over the AgriIR knowledge base (15,247 agricultural knowledge entries) and DuckDuckGo web retrieval run concurrently rather than serially. Timestamp the start and end of the retrieval block per mode to compute total wall-clock latency and the sequential-to-parallel speedup ratio; the reported baseline is ~180 s sequential vs ~50 s parallel (ratio roughly 0.25-0.31). Keep per-query and per-mode latency columns so that throughput gains can be attributed to concurrency rather than caching. Write results to retrieval_latency_results.csv and validate that parallel-mode latency falls within the expected reduction band implied by the reported figures.

## Related tools

- **FAISS** (Local vector similarity search backend executed per sub-query within the parallel retrieval stage) — https://github.com/facebookresearch/faiss
- **AgriIR** (Six-stage RAG framework whose Stage-3 Parallel Retrieval step is being orchestrated; provides the FAISS index and web-retrieval combination) — https://github.com/Shuvam-Banerji-Seal/AgriIR

## Examples

```
from concurrent.futures import ThreadPoolExecutor; with ThreadPoolExecutor(max_workers=4) as ex: retrieved = list(ex.map(retrieve, subqueries))
```

## Evaluation signals

- retrieval_latency_results.csv exists and contains latency fields for both sequential and parallel modes
- Parallel-mode total latency falls in the expected reduction band implied by ~180 s vs ~50 s (speedup ratio roughly 0.25-0.31)
- Every sub-query from the Stage-2 decomposition (3-5 per query) produced retrieval results in both modes
- Per-query latency columns are present so individual workloads can be compared across modes

## Limitations

- The ~180 s sequential vs ~50 s parallel figures are authors' reported testing numbers; absolute latency depends on hardware, network conditions for web retrieval, and FAISS index size
- Parallelism speeds up retrieval wall-clock time only; it does not change retrieval quality or the downstream Stage-4/5/6 behavior
- No discussion-section limitations were extractable from the source document (no changelog/limitations text present)

## Evidence
