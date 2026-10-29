---
name: ollama-local-llm-deployment
description: Use when a RAG or multi-stage retrieval pipeline must run in a low-resource,
  compute-constrained setting and small (1B–3B parameter) models are pulled and served
  locally through Ollama rather than cloud APIs.
when_to_use_negative:
- When the goal is state-of-the-art open-ended generation without grounded citations
  — standalone large baselines (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B) are the
  appropriate comparison/choice, not local 1B–3B models
- When domain adaptation requires fine-tuning or training data — this skill supports
  configuration-only retargeting and deliberately involves no fine-tuning or data
  collection
- When the host has neither the RAM/disk for local model weights nor network access
  to pull them from the Ollama registry
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: Ollama
  role: Local model runtime that pulls and serves gemma3:1b and llama3.2:3b for all
    pipeline stages, replacing cloud LLM APIs in the low-resource deployment
  repo: https://github.com/ollama/ollama
- name: Gemma3:1B
  role: Stage-1 Query Refinement model (T=0.1) served locally via Ollama
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Llama3.2:3B
  role: Stage-5 Synthesis model (T=0.2) served locally via Ollama; also assessed with
    and without FAISS database integration
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: FAISS
  role: Local vector search component that pairs with Ollama-served models in Stage-3
    Parallel Retrieval
  repo: https://github.com/facebookresearch/faiss
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:c72b5b5ad2e05187ab0a590e511785afac69a777109ce26a9d404969271c7eef
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ollama/ollama
  tools:
  - AgriIR
  - FAISS
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  - Ollama
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ollama-local-llm-deployment

## Summary

Provision small local LLMs (gemma3:1b for query refinement, llama3.2:3b for synthesis) via Ollama so the AgriIR 6-stage RAG pipeline runs fully offline without GPU-heavy infrastructure or API keys.

## When to use

Use when a RAG or multi-stage retrieval pipeline must run in a low-resource, compute-constrained setting and small (1B–3B parameter) models are pulled and served locally through Ollama rather than cloud APIs.

## When NOT to use

- When the goal is state-of-the-art open-ended generation without grounded citations — standalone large baselines (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B) are the appropriate comparison/choice, not local 1B–3B models
- When domain adaptation requires fine-tuning or training data — this skill supports configuration-only retargeting and deliberately involves no fine-tuning or data collection
- When the host has neither the RAM/disk for local model weights nor network access to pull them from the Ollama registry

## Inputs

- Ollama model tags gemma3:1b and llama3.2:3b (pulled from the Ollama registry)
- AgriIR configuration files config.yml and config/enhanced_rag_config.yaml (agri_bot_searcher/)
- Target-domain queries submitted to the POST /api/query endpoint

## Outputs

- Locally served LLM endpoints on the Ollama REST API (localhost:11434)
- JSON query responses from the 6-stage pipeline with deterministic citations restricted to sources of similarity >= 0.75

## How to apply

Install Ollama for the target platform (e.g. curl -fsSL https://ollama.com/install.sh | sh on Linux, or the Docker image ollama/ollama), then pull the two models the AgriIR pipeline expects: gemma3:1b (Stage-1 Query Refinement, T=0.1) and llama3.2:3b (Stage-5 Synthesis, T=0.2). Model names must match the tags referenced in config.yml and config/enhanced_rag.yaml, and per-stage temperatures are set in configuration, not code. Once Ollama serves the models on its local REST API (default port 11434), start the full-RAG server (python3 agri_bot_searcher/src/enhanced_web_ui.py) and submit queries to POST /api/query. The rationale, per the paper, is that architectural intelligence — domain specialization, controlled sampling, and multi-stage reasoning — compensates for parameter count, letting 1B-parameter models produce reliable, evidence-grounded responses comparable to much larger standalone baselines.

## Related tools

- **Ollama** (Local model runtime that pulls and serves gemma3:1b and llama3.2:3b for all pipeline stages, replacing cloud LLM APIs in the low-resource deployment) — https://github.com/ollama/ollama
- **Gemma3:1B** (Stage-1 Query Refinement model (T=0.1) served locally via Ollama) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Llama3.2:3B** (Stage-5 Synthesis model (T=0.2) served locally via Ollama; also assessed with and without FAISS database integration) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **FAISS** (Local vector search component that pairs with Ollama-served models in Stage-3 Parallel Retrieval) — https://github.com/facebookresearch/faiss

## Examples

```
ollama pull gemma3:1b && ollama pull llama3.2:3b
```

## Evaluation signals

- ollama list (or equivalent) shows both gemma3:1b and llama3.2:3b pulled and available before the server starts
- The full-RAG pipeline executes end-to-end with no cloud API calls or GPU requirement, matching the low-resource deployment premise
- Per-stage sampling temperatures match configuration: T=0.1 (Stage-1 refinement), T=0.5 (Stage-2 sub-query decomposition, 3–5 sub-queries), T=0.2 (Stage-5 synthesis)
- Saved JSON answers contain deterministic citations limited to sources with similarity >= 0.75 (Stage-6 Citation Insert)
- AgriIR variants using these local models achieve citation quality in the 73–84% perfect-citation range reported in the paper, absent in baseline models

## Limitations

- Models are deliberately small (1B–3B parameters); reliability depends on architectural design (domain specialization, controlled sampling, multi-stage reasoning), so raw model quality alone may underperform larger baselines on unstructured tasks
- Ollama setup is a manual step in the AgriIR workflow (explicit ollama pull commands) and is not automated by the framework
- No discussion-section text was present in the source document, so no additional limitations or reproducibility gaps could be extracted from the paper

## Evidence

- [intro] Architecture over parameter scale: [evidence span withheld — anchor status: not_found]
- [readme] Ollama installation path: [evidence span withheld — anchor status: not_found]
