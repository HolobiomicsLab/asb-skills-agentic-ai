---
name: agricultural-query-refinement-llm
description: Use when a raw, ambiguous natural-language agricultural question must
  be normalized and decomposed into sub-queries before vector or web retrieval in
  a RAG pipeline.
when_to_use_negative:
- The input is already a well-formed, unambiguous structured query that needs no refinement
  or decomposition
- The task requires non-agricultural domains without reconfiguring the domain-agent
  catalogue and prompts
- Only a large standalone LLM baseline (e.g., ChatGPT-4o) is being run without the
  AgriIR multi-stage pipeline
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3372
- http://edamontology.org/topic_3473
tools:
- name: Gemma3:1B
  role: LLM used for Stage-1 query refinement (T=0.1) and Stage-2 sub-query decomposition
    (T=0.5)
- name: Ollama
  role: Local serving runtime for the Gemma3:1B refinement/decomposition model
  repo: https://github.com/ollama/ollama
- name: AgriIR
  role: Configurable RAG framework hosting the declarative Stage-1 and Stage-2 refinement/decomposition
    steps
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:7ce6a7be9a2e86f4011502165e795a8623d71e650497842bdb6515a11d1d0078
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3372
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# agricultural-query-refinement-llm

## Summary

LLM-based query refinement and sub-query decomposition stages (Stages 1-2) of the AgriIR pipeline, which disambiguate raw agricultural user questions and split them into 3-5 tractable sub-queries before retrieval.

## When to use

Use when a raw, ambiguous natural-language agricultural question must be normalized and decomposed into sub-queries before vector or web retrieval in a RAG pipeline.

## When NOT to use

- The input is already a well-formed, unambiguous structured query that needs no refinement or decomposition
- The task requires non-agricultural domains without reconfiguring the domain-agent catalogue and prompts
- Only a large standalone LLM baseline (e.g., ChatGPT-4o) is being run without the AgriIR multi-stage pipeline

## Inputs

- Raw user natural-language query on an agricultural topic (e.g., Indian agriculture policy questions)
- Pipeline configuration specifying refinement/decomposition models (Gemma3:1B via Ollama) and temperatures (0.1 and 0.5)

## Outputs

- Refined, disambiguated query string (Stage-1, T=0.1)
- 3-5 decomposed sub-queries (Stage-2, T=0.5) passed to parallel retrieval

## How to apply

Run the user's raw query through Gemma3:1B served via Ollama at temperature 0.1 to remove ambiguity and normalize domain phrasing. Then, at temperature 0.5, prompt the same LLM to decompose the refined query into 3-5 sub-queries that break the question into tractable, independently retrievable parts. Keep temperatures low and deterministic because controlled sampling (temperature tuning) is a deliberate architectural choice that lets small 1B-parameter models compensate for parameter count. Treat both stages as declarative, configuration-controlled steps (no code modification), so models and sampling parameters are swapped via configuration when adapting to new domains. The refined query and sub-queries feed Stage-3 parallel retrieval (FAISS over the AgriIR dataset plus DuckDuckGo web search).

## Related tools

- **Gemma3:1B** (LLM used for Stage-1 query refinement (T=0.1) and Stage-2 sub-query decomposition (T=0.5))
- **Ollama** (Local serving runtime for the Gemma3:1B refinement/decomposition model) — https://github.com/ollama/ollama
- **AgriIR** (Configurable RAG framework hosting the declarative Stage-1 and Stage-2 refinement/decomposition steps) — https://github.com/Shuvam-Banerji-Seal/AgriIR

## Examples

```
ollama run gemma3:1b --temp 0.1 "Refine this agricultural query, removing ambiguity: <user_query>"
```

## Evaluation signals

- Refined query is free of the ambiguity of the raw input and preserves the original agricultural information need
- Decomposition yields between 3 and 5 sub-queries covering distinct facets of the refined query
- Stage-1 output generated at temperature 0.1 is stable/deterministic across repeated runs
- Downstream Stage-3 parallel retrieval executes successfully over each sub-query (FAISS local corpus D and DuckDuckGo web corpus W are populated)
- All stage parameters (model names, temperatures) are set in configuration, not hard-coded, confirming the declarative-stage design

## Limitations

- Reliability depends on architectural design (controlled sampling, multi-stage reasoning) rather than model scale; the 1B-parameter Gemma3:1B may still fail on highly ambiguous or out-of-domain queries
- Sub-query count is constrained to 3-5, which may under-decompose very complex multi-part questions
- Framework targets low-resource, domain-sensitive settings such as Indian agriculture; performance in other domains requires reconfiguration, not just reuse

## Evidence

- [methods] Stage-1 query refinement temperature: "Stage-1<br/>Query Refine<br/>T=0.1"
- [methods] Stage-2 sub-query decomposition temperature: "Stage-2<br/>Sub-Query<br/>T=0.5"
- [intro] Modular declarative stages: "decomposes the end-to-end process into modular, declarative stages i.e., query refinement, retrieval, synthesis, and evaluation"
- [intro] Controlled sampling rationale: [evidence span withheld — anchor status: not_found]
- [readme] Configurable stage chain: "AgriIR is a 6-stage retrieval-augmented generation pipeline for agricultural Q&A. It combines local vector search (FAISS)"
