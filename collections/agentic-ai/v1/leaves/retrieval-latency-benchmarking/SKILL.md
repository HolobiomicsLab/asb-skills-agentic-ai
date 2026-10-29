---
name: retrieval-latency-benchmarking
description: Use when you need to measure or reproduce retrieval-stage latency of
  the AgriIR pipeline (Stage-3) and quantify the sequential-to-parallel speedup for
  multi-subquery workloads.
when_to_use_negative:
- You are benchmarking LLM synthesis or citation stages (Stage-5 Synthesis, Stage-6
  Citation Insert) rather than Stage-3 retrieval latency.
- Your workload does not decompose into multiple sub-queries, so there is no concurrency
  to exploit from ThreadPoolExecutor dispatch.
- You have no local FAISS index built from the AgriIR dataset (15,247 entries) and
  must rely solely on web retrieval, making latency figures incomparable to the paper's
  configuration.
tools:
- name: AgriIR
  role: RAG framework whose Stage-3 retrieval is benchmarked; run in both sequential
    and ThreadPoolExecutor-parallel modes
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: FAISS
  role: Backend for local vector similarity search over the AgriIR dataset during
    Stage-3 retrieval in both timing modes
  repo: https://github.com/facebookresearch/faiss
- name: ShuvBan/AgriIR_dataset
  role: 15,247-entry agricultural knowledge base that populates the FAISS index backing
    retrieval
  repo: https://huggingface.co/datasets/ShuvBan/AgriIR_dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:6a169ebb4fcfe8db7885c80939ddedb02d3995c626c2abcd57738169cde612c5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Shuvam-Banerji-Seal/AgriIR
  tools:
  - AgriIR
  - FAISS
  - ShuvBan/AgriIR_dataset
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# retrieval-latency-benchmarking

## Summary

Benchmark the wall-clock latency of AgriIR's Stage-3 multi-source retrieval (local FAISS vector search over the 15,247-entry AgriIR dataset plus DuckDuckGo web retrieval) under sequential vs ThreadPoolExecutor-parallel execution across a four-subquery workload, to quantify the speedup from parallelizing sub-query retrieval.

## When to use

Use when you need to measure or reproduce retrieval-stage latency of the AgriIR pipeline (Stage-3) and quantify the sequential-to-parallel speedup for multi-subquery workloads.

## When NOT to use

- You are benchmarking LLM synthesis or citation stages (Stage-5 Synthesis, Stage-6 Citation Insert) rather than Stage-3 retrieval latency.
- Your workload does not decompose into multiple sub-queries, so there is no concurrency to exploit from ThreadPoolExecutor dispatch.
- You have no local FAISS index built from the AgriIR dataset (15,247 entries) and must rely solely on web retrieval, making latency figures incomparable to the paper's configuration.

## Inputs

- AgriIR benchmark of 191 annotated agricultural policy queries
- AgriIR_dataset (ShuvBan/AgriIR_dataset, 15,247 agricultural knowledge entries) loaded into a FAISS index
- Stage-2 decomposed sub-queries (3-5 per query; four-subquery workload used for this benchmark)

## Outputs

- retrieval_latency_results.csv with per-mode total latency, per-query latency columns, and the sequential-to-parallel speedup ratio

## How to apply

Load the 191-query agricultural policy benchmark and the AgriIR knowledge base (15,247 entries) backing the FAISS index. Issue each query as a four-subquery workload consistent with Stage-2 decomposition (3-5 sub-queries per query, T=0.5). Run Stage-3 retrieval sequentially for all 191 queries — local FAISS vector search for each sub-query plus DuckDuckGo web retrieval — timestamping the start and end of the retrieval block; then repeat identically with sub-query retrievals dispatched concurrently via ThreadPoolExecutor (mirroring the paper's 'Stage-3 Parallel Retrieval' design). Compute total wall-clock latency per mode and the speedup ratio, and write per-mode total latency, per-query latency columns, and the ratio to retrieval_latency_results.csv. Judge success by whether the parallel total latency falls in the reported reduction band (~180s sequential vs ~50s parallel, i.e. speedup ratio roughly 0.25-0.31 for parallel/sequential, about 3.5x).

## Related tools

- **AgriIR** (RAG framework whose Stage-3 retrieval is benchmarked; run in both sequential and ThreadPoolExecutor-parallel modes) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **FAISS** (Backend for local vector similarity search over the AgriIR dataset during Stage-3 retrieval in both timing modes) — https://github.com/facebookresearch/faiss
- **ShuvBan/AgriIR_dataset** (15,247-entry agricultural knowledge base that populates the FAISS index backing retrieval) — https://huggingface.co/datasets/ShuvBan/AgriIR_dataset

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR
```

## Evaluation signals

- retrieval_latency_results.csv exists and contains latency fields for both sequential and parallel modes plus a speedup ratio column.
- Parallel-mode total latency falls within the expected reduction band implied by the reported figures: parallel/sequential ratio roughly 0.25-0.31 (approximately 3.5x speedup).
- Sequential-mode total latency is on the order of the reported ~180s for the 191-query four-subquery workload.
- Both runs use identical inputs: the same 191-query benchmark and the same four-subquery workloads derived from Stage-2 decomposition.
- Per-query latency columns are populated for all 191 queries in both modes.

## Limitations

- Latency figures (~180s sequential, ~50s parallel) are as reported in the authors' testing and depend on the local compute environment, network conditions for DuckDuckGo web retrieval, and FAISS index configuration.
- The benchmark assumes a four-subquery workload matching the Stage-2 decomposition range of 3-5 sub-queries per query; other decompositions will change the speedup ratio.
- No discussion-section content is available in the source document, so limitations and reproducibility gaps for the timing setup were not documented.

## Evidence
