---
name: agricultural-query-decomposition-into-subqueries
description: Use when a complex agricultural policy query is too broad for a single
  retrieval pass and downstream Stage-3 retrieval must be fanned out across local
  FAISS search and web retrieval per sub-query.
when_to_use_negative:
- Query is already a narrow, single-fact lookup where a single refined query suffices
  — decomposition would add redundant retrieval cost
- Running an evaluation that requires comparing a single-query baseline, since the
  paper's four-subquery workload is specific to Stage-3 latency and multi-source retrieval
  experiments
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3489
- http://edamontology.org/topic_3372
tools:
- name: AgriIR
  role: Framework hosting the six-stage pipeline; sub-query decomposition is its Stage-2
    step at T=0.5
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Llama3.2:3B
  role: Ollama-hosted LLM variant used in AgriIR configurations to perform sub-query
    decomposition and other pipeline stages
  repo: https://github.com/ollama/ollama
- name: FAISS
  role: 'Downstream consumer of sub-queries: each decomposed sub-query drives local
    FAISS vector search over the 15,247-entry AgriIR dataset'
  repo: https://github.com/facebookresearch/faiss
- name: ollama
  role: Runtime serving the LLM that performs the Stage-2 decomposition call
  repo: https://github.com/ollama/ollama
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:cb0a0db5e28294ee109e6e31785e7e76d27ff3e0db54160e1bb675d691d59a85
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Shuvam-Banerji-Seal/AgriIR
  tools:
  - AgriIR
  - FAISS
  - Llama3.2:3B
  - ollama
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3489
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# agricultural-query-decomposition-into-subqueries

## Summary

Decomposes an agricultural policy question into multiple parallel retrieval sub-queries using AgriIR's Stage-2 sub-query decomposition step (an LLM call at temperature 0.5), enabling multi-source retrieval over a FAISS-backed knowledge base and web sources.

## When to use

Use when a complex agricultural policy query is too broad for a single retrieval pass and downstream Stage-3 retrieval must be fanned out across local FAISS search and web retrieval per sub-query.

## When NOT to use

- Query is already a narrow, single-fact lookup where a single refined query suffices — decomposition would add redundant retrieval cost
- Running an evaluation that requires comparing a single-query baseline, since the paper's four-subquery workload is specific to Stage-3 latency and multi-source retrieval experiments

## Inputs

- Refined agricultural policy query (output of AgriIR Stage-1 Query Refinement, T=0.1)
- LLM configuration with temperature=0.5 for sub-query decomposition
- Ollama-hosted model (e.g. Llama3.2:3B or Gemma3 variants used by AgriIR)

## Outputs

- 3-5 sub-queries per parent query (four sub-queries per query in the benchmark workload)
- Sub-query list feeding Stage-3 Parallel Retrieval (FAISS local search + web retrieval per sub-query)

## How to apply

Place the decomposition step as Stage-2 of the AgriIR six-stage pipeline, after Stage-1 query refinement (T=0.1) and before Stage-3 parallel retrieval. Invoke an Ollama-hosted LLM with temperature set to 0.5 — the higher temperature versus refinement is deliberate, encouraging diverse sub-query phrasings rather than a single canonical rewrite. Target the decomposition range of 3-5 sub-queries per query, matching the four-subquery workload used in the paper's latency and retrieval experiments. Each emitted sub-query should be independently answerable by either local vector search (FAISS over the 15,247-entry AgriIR dataset) or DuckDuckGo web retrieval, so that Stage-3 can dispatch them concurrently via ThreadPoolExecutor. Validate decomposition quality downstream: sub-queries that cannot be grounded by either retrieval source signal over-decomposition or drift from the parent query.

## Related tools

- **AgriIR** (Framework hosting the six-stage pipeline; sub-query decomposition is its Stage-2 step at T=0.5) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Llama3.2:3B** (Ollama-hosted LLM variant used in AgriIR configurations to perform sub-query decomposition and other pipeline stages) — https://github.com/ollama/ollama
- **FAISS** (Downstream consumer of sub-queries: each decomposed sub-query drives local FAISS vector search over the 15,247-entry AgriIR dataset) — https://github.com/facebookresearch/faiss
- **ollama** (Runtime serving the LLM that performs the Stage-2 decomposition call) — https://github.com/ollama/ollama

## Examples

```
ollama run llama3.2:3b "Decompose this agricultural policy question into 3-5 retrieval sub-queries: <query>" --temperature 0.5
```

## Evaluation signals

- Each parent query yields between 3 and 5 sub-queries, consistent with the Stage-2 decomposition range cited in the task workflow
- Decomposition runs at temperature 0.5, distinct from the T=0.1 refinement and T=0.2 synthesis settings of neighbouring stages
- Every emitted sub-query is retrievable via at least one of the two Stage-3 sources (FAISS local vector search or DuckDuckGo web retrieval)
- Sub-queries remain topically aligned with the parent agricultural policy query, with no degenerate duplicates across sub-queries

## Limitations

- Decomposed sub-queries increase retrieval cost proportionally; the paper's own latency figures (~180 s sequential vs ~50 s parallel for a four-subquery workload over 191 queries) show decomposition only remains practical when Stage-3 dispatch is parallelized
- Over-decomposition beyond the 3-5 sub-query range could dilute retrieval quality or break the ThreadPoolExecutor fan-out assumptions in Stage-3
- The article's discussion section is absent from the extracted source, so known failure modes of the decomposition stage (e.g. sub-query drift) are not documented by the authors

## Evidence

- [methods] Stage-2 sub-query decomposition at temperature 0.5: "Stage-2<br/>Sub-Query<br/>T=0.5"
- [methods] Four-subquery workload within 3-5 range feeds parallel retrieval: "Stage-3<br/>Parallel Retrieval"
- [intro] AgriIR decomposes retrieval into declarative stages including query refinement: "decomposes the end-to-end process into modular, declarative stages i.e., query refinement, retrieval, synthesis, and evaluation"
- [methods] Local FAISS vector search backs each sub-query retrieval: "local vector search (FAISS) with web retrieval"
- [results] Benchmark of 191 agricultural policy queries defines the evaluation workload: "Evaluation on 191 agricultural policy queries with 30 human annotators validates our approach."
