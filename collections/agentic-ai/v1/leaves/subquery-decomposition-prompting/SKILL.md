---
name: subquery-decomposition-prompting
description: Use when a complex, multi-faceted user question must be broken into tractable
  sub-problems before retrieval, e.g. to fan out parallel FAISS + web searches in
  a configurable RAG pipeline.
when_to_use_negative:
- Query is already simple and unambiguous — decomposition adds fan-out cost and can
  dilute retrieval precision; refinement alone (Stage-1, T=0.1) suffices
- Framework is deployed in a domain where a single sub-topic covers the information
  need and 3–5 sub-queries would generate redundant retrievals
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: Gemma3:1B
  role: Decomposition model that expands the refined query into 3–5 sub-queries at
    temperature 0.5
- name: Ollama
  role: Serves the Gemma3:1B decomposition model locally for low-resource deployment
  repo: https://github.com/ollama/ollama
- name: AgriIR
  role: Framework whose Stage-2 declarative configuration controls this decomposition
    without code modification
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:a3610f8e7d740528407c5f4e690cf979d1ac8e797383a00d9180187ce66e47bc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ollama/ollama
  tools:
  - AgriIR
  - DuckDuckGo
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  - Ollama
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# subquery-decomposition-prompting

## Summary

Stage 2 of the AgriIR six-stage RAG pipeline: a temperature-0.5 LLM call (Gemma3:1B via Ollama) expands a single refined user query into 3–5 sub-queries so that downstream parallel retrieval over FAISS and the web covers complementary facets of the information need.

## When to use

Use when a complex, multi-faceted user question must be broken into tractable sub-problems before retrieval, e.g. to fan out parallel FAISS + web searches in a configurable RAG pipeline.

## When NOT to use

- Query is already simple and unambiguous — decomposition adds fan-out cost and can dilute retrieval precision; refinement alone (Stage-1, T=0.1) suffices
- Framework is deployed in a domain where a single sub-topic covers the information need and 3–5 sub-queries would generate redundant retrievals

## Inputs

- Refined query string from Stage-1 (Gemma3:1B, temperature 0.1 output)
- Stage-2 configuration entry specifying decomposition model (e.g. gemma3:1b via Ollama) and temperature 0.5

## Outputs

- 3–5 sub-query strings ready to drive Stage-3 parallel retrieval (FAISS over AgriIR_dataset and DuckDuckGo web search)

## How to apply

After Stage-1 query refinement removes ambiguity at temperature 0.1, feed the refined query to a small LLM (Gemma3:1B served via Ollama) with sampling temperature set to 0.5 — higher than refinement to encourage diverse facets, lower than free generation to stay on-topic — and prompt it to emit 3–5 sub-queries that decompose the information need. Each sub-query should target one tractable sub-problem (e.g. crop, disease, economics, climate aspects) that can independently drive Stage-3 parallel retrieval against the FAISS-indexed AgriIR corpus (15,247 entries embedded with Qwen3-Embedding-8B) and DuckDuckGo web search. The sub-query count is bounded (3–5) to keep retrieval fan-out and synthesis context manageable. In AgriIR this stage is a declarative configuration entry rather than hard-coded logic, so model choice and temperature are reconfigured per domain without code modification.

## Related tools

- **Gemma3:1B** (Decomposition model that expands the refined query into 3–5 sub-queries at temperature 0.5)
- **Ollama** (Serves the Gemma3:1B decomposition model locally for low-resource deployment) — https://github.com/ollama/ollama
- **AgriIR** (Framework whose Stage-2 declarative configuration controls this decomposition without code modification) — https://github.com/Shuvam-Banerji-Seal/AgriIR

## Examples

```
curl http://localhost:11434/api/generate -d '{"model":"gemma3:1b","options":{"temperature":0.5},"prompt":"Decompose into 3-5 sub-queries: <refined_query>"}'
```

## Evaluation signals

- Decomposition output contains between 3 and 5 sub-queries
- Each sub-query is a self-contained, tractable sub-problem derived from the refined Stage-1 query (no ambiguity carried over from the raw input)
- Sub-queries are topically distinct enough that Stage-3 retrieval over each returns complementary corpus D and web W evidence rather than duplicates
- Changing the decomposition model or temperature in configuration changes the output without any code edit, confirming the declarative-stage contract

## Limitations

- Decomposition quality is bounded by the 1B-parameter model; the paper's hypothesis is that architectural design (controlled sampling, multi-stage reasoning) rather than model scale provides reliability, so very obscure facets may be missed
- The 3–5 sub-query cap may under-decompose extremely broad questions or over-decompose narrow ones
- Output depends on temperature 0.5 sampling, so sub-queries are not deterministic across runs

## Evidence

- [results] Query refinement and decomposition break inputs into tractable sub-problems: [evidence span withheld — anchor status: not_found]
- [intro] Controlled sampling via temperature tuning enables small models: [evidence span withheld — anchor status: not_found]
