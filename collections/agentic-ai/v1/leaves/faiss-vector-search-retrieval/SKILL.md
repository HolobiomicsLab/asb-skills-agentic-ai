---
name: faiss-vector-search-retrieval
description: Use when a RAG pipeline needs fast local evidence retrieval over a domain-specific
  embedded corpus (e.g., agricultural knowledge entries), especially when retrieval
  latency across multi-subquery workloads must be minimized.
when_to_use_negative:
- 'When the task requires only web-sourced evidence: FAISS covers only the local corpus
  D; web corpus W requires DuckDuckGo retrieval and is a complementary evidence source
  in AgriIR'
- When queries are single-turn and latency is irrelevant — sequential execution avoids
  ThreadPoolExecutor concurrency overhead
- 'When the domain corpus differs (e.g., health or climate adaptation): AgriIR adaptation
  is ''through reconfiguration alone'', but the embedded index must be rebuilt for
  the new domain''s data'
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FAISS
  role: Local vector similarity search engine backing Stage-3 retrieval over the embedded
    AgriIR dataset to build corpus D
  repo: https://github.com/facebookresearch/faiss
- name: AgriIR
  role: Configurable RAG framework whose Stage-3 parallel retrieval invokes FAISS
    search over the local knowledge base
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: AgriIR_dataset
  role: The 15,247-entry agricultural knowledge corpus that is embedded and indexed
    by FAISS
  repo: https://huggingface.co/datasets/ShuvBan/AgriIR_dataset
- name: Qwen3-Embedding-8B
  role: Embedding model used to vectorize the AgriIR dataset for FAISS index construction
- name: ollama/ollama
  role: Runtime for the LLM stages (refinement, decomposition, synthesis) surrounding
    the FAISS retrieval stage
  repo: https://github.com/ollama/ollama
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:d698e6a9fec3f99485ac567f4b24d5c5bf9f415c8609bb6ccd9cbcb0704efe0c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AgriIR
  - AgriIR_dataset
  - DuckDuckGo
  - FAISS
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  - Qwen3-Embedding-8B
  - ollama/ollama
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# faiss-vector-search-retrieval

## Summary

Performs Stage-3 local corpus retrieval in the AgriIR framework using FAISS vector search over the 15,247-entry AgriIR dataset (embedded with Qwen3-Embedding-8B), optionally parallelized with a ThreadPoolExecutor, to build local evidence corpus D that feeds downstream agent enhancement and synthesis.

## When to use

Use when a RAG pipeline needs fast local evidence retrieval over a domain-specific embedded corpus (e.g., agricultural knowledge entries), especially when retrieval latency across multi-subquery workloads must be minimized.

## When NOT to use

- When the task requires only web-sourced evidence: FAISS covers only the local corpus D; web corpus W requires DuckDuckGo retrieval and is a complementary evidence source in AgriIR
- When queries are single-turn and latency is irrelevant — sequential execution avoids ThreadPoolExecutor concurrency overhead
- When the domain corpus differs (e.g., health or climate adaptation): AgriIR adaptation is 'through reconfiguration alone', but the embedded index must be rebuilt for the new domain's data

## Inputs

- AgriIR dataset (15,247 agricultural knowledge entries, HuggingFace dataset ShuvBan/AgriIR_dataset) embedded with Qwen3-Embedding-8B into a FAISS index
- Refined/decomposed sub-queries from Stage-2 (3-5 sub-queries per user query at temperature 0.5)
- Query embeddings generated with the same Qwen3-Embedding-8B model used for corpus embedding

## Outputs

- Local evidence corpus D (FAISS search results per sub-query)
- Per-mode (sequential vs parallel) retrieval latency measurements and speedup ratio for benchmarking (e.g., retrieval_latency_results.csv with per-mode total latency and per-query latency columns)

## How to apply

First embed the AgriIR dataset (15,247 agricultural knowledge entries from HuggingFace, embedded with Qwen3-Embedding-8B) into a FAISS index. During Stage-3 retrieval, issue FAISS vector searches for each decomposed sub-query (AgriIR Stage-2 produces 3-5 sub-queries at temperature 0.5) to collect the local corpus D, alongside DuckDuckGo web retrieval for corpus W. For multi-subquery workloads, dispatch sub-query retrievals concurrently via a ThreadPoolExecutor instead of sequential execution; the authors report this reduces total retrieval latency from ~180 s to ~50 s for a four-subquery workload (~3.5x speedup). The retrieved corpus D is passed to Stage-4 agent enhancement (domain-agent catalogue, max_agents: 6) and Stage-5 synthesis at temperature 0.2. FAISS supports cosine similarity via dot product on normalized vectors, which is relevant to the downstream citation-insertion threshold of similarity >= 0.75.

## Related tools

- **FAISS** (Local vector similarity search engine backing Stage-3 retrieval over the embedded AgriIR dataset to build corpus D) — https://github.com/facebookresearch/faiss
- **AgriIR** (Configurable RAG framework whose Stage-3 parallel retrieval invokes FAISS search over the local knowledge base) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **AgriIR_dataset** (The 15,247-entry agricultural knowledge corpus that is embedded and indexed by FAISS) — https://huggingface.co/datasets/ShuvBan/AgriIR_dataset
- **Qwen3-Embedding-8B** (Embedding model used to vectorize the AgriIR dataset for FAISS index construction)
- **ollama/ollama** (Runtime for the LLM stages (refinement, decomposition, synthesis) surrounding the FAISS retrieval stage) — https://github.com/ollama/ollama

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR  # then run Stage-3 retrieval with ThreadPoolExecutor over FAISS index of the AgriIR_dataset
```

## Evaluation signals

- retrieval_latency_results.csv exists and contains latency fields for both sequential and parallel modes
- Parallel-mode total latency falls within the expected reduction band implied by the reported ~180 s (sequential) vs ~50 s (parallel) figures, i.e., speedup ratio roughly 0.25-0.31 (~3.5x)
- Retrieved corpus D is non-empty for each sub-query and combined with web corpus W in the synthesized_answer output, which parses as JSON with a non-empty answer field
- FAISS index covers all 15,247 entries of the AgriIR dataset; cosine similarity is computed as dot product on normalized vectors

## Limitations

- Latency figures (~180 s sequential, ~50 s parallel) are 'as reported in the authors' testing' and depend on the benchmark setup (191-query, four-subquery workload); they may not transfer to other hardware or workloads
- FAISS search time depends on index type and memory trade-offs; approximate indexes trade precision for speed
- No discussion-section text was extractable from the source document, so additional limitations, missing artifacts, or reproducibility gaps could not be verified

## Evidence

- [readme] FAISS similarity semantics: [evidence span withheld — anchor status: external_doc]
