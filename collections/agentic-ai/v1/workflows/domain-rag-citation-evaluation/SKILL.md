---
name: domain-rag-citation-evaluation-workflow
description: 'Use when a fixed set of agricultural policy questions must be decomposed,
  grounded in local and web evidence, answered with deterministic source citations,
  and scored from human answer and citation annotations.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 6
  member_skills:
  - subquery-decomposition-prompting
  - agricultural-query-refinement-llm
  - six-stage-retrieval-augmented-answer-synthesis
  - parallel-multi-source-retrieval-orchestration
  - agricultural-query-decomposition-into-subqueries
  - web-retrieval-integration-with-vector-search
  - faiss-vector-search-retrieval
  - domain-agent-keyword-enhancement
  - web-search-llm-filtering
  - temperature-controlled-answer-synthesis
  - llm-response-quality-annotation-aggregation
  - llm-multi-agent-orchestration
  - agricultural-answer-citation-insertion
  - sentence-embedding-similarity-scoring
  - citation-source-attribution
  - post-hoc-answer-grounding
  - citation-quality-scoring
  - composite-score-compilation-across-model-conditions
  - per-query-composite-score-analysis
  member_tools:
  - AgriIR
  - DuckDuckGo
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  - Ollama
  - FAISS
  - Sentence Transformer
  - ChatGPT-4o
  - GPT-OSS-120B
  - Gemini 2.5 Flash
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/agentic-ai/v1
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Agricultural Policy RAG with Deterministic Citations and Human Evaluation

## Summary

Decompose agricultural policy questions, retrieve and enhance complementary evidence, synthesize answers, insert auditable citations, and aggregate grounded-answer scores.


## When to use

Use when a fixed set of agricultural policy questions must be decomposed, grounded in local and web evidence, answered with deterministic source citations, and scored from human answer and citation annotations.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — decompose_queries

**Goal:** split each complex domain question into bounded retrieval subqueries

**EDAM operation:** http://edamontology.org/operation_0226

**Inputs:** jsonl/refined-agricultural-policy-queries, yaml/decomposition-configuration · **Outputs:** jsonl/retrieval-subqueries

**Candidate leaf skills:** `subquery-decomposition-prompting` (primary), `agricultural-query-refinement-llm`, `six-stage-retrieval-augmented-answer-synthesis`

**Tools (primary):** AgriIR, DuckDuckGo, Gemma3:1B, Gemma3:27B, Llama3.2:3B, Ollama

**Other candidate tools:** FAISS

**Grounding:** 1 KB(s); DOIs: 10.1007/978-3-032-21324-2_37

### Stage 2 — retrieve_evidence

**Goal:** retrieve local and web evidence for every decomposed subquery

**EDAM operation:** http://edamontology.org/operation_0306

**Inputs:** jsonl/retrieval-subqueries, index/agriir-faiss, yaml/agricultural-web-source-policy · **Outputs:** jsonl/retrieved-local-chunks, jsonl/retrieved-web-chunks

**Candidate leaf skills:** `parallel-multi-source-retrieval-orchestration` (primary), `agricultural-query-decomposition-into-subqueries`, `web-retrieval-integration-with-vector-search`, `faiss-vector-search-retrieval`

**Tools (primary):** AgriIR, FAISS

**Other candidate tools:** Llama3.2:3B, ollama, AgriIR_dataset, DuckDuckGo, Gemma3:1B, Gemma3:27B, Qwen3-Embedding-8B

**Grounding:** 1 KB(s); DOIs: 10.1007/978-3-032-21324-2_37

### Stage 3 — enhance_evidence

**Goal:** inject selected domain-agent keywords into both retrieved evidence pools

**EDAM operation:** http://edamontology.org/operation_2422

**Inputs:** jsonl/refined-agricultural-policy-queries, jsonl/retrieval-subqueries, jsonl/retrieved-local-chunks, jsonl/retrieved-web-chunks, yaml/domain-agent-catalogue · **Outputs:** jsonl/agent-enhanced-local-chunks, jsonl/agent-enhanced-web-chunks, json/agent-keyword-context

**Candidate leaf skills:** `domain-agent-keyword-enhancement` (primary), `web-search-llm-filtering`

**Tools (primary):** AgriIR, DuckDuckGo, FAISS, Gemma3:1B, Gemma3:27B, Llama3.2:3B, Ollama


**Grounding:** 1 KB(s); DOIs: 10.1007/978-3-032-21324-2_37

### Stage 4 — synthesize_answers

**Goal:** generate evidence-grounded answers before citation insertion

**EDAM operation:** http://edamontology.org/operation_3778

**Inputs:** jsonl/refined-agricultural-policy-queries, jsonl/agent-enhanced-local-chunks, jsonl/agent-enhanced-web-chunks, json/agent-keyword-context, yaml/answer-generation-configuration · **Outputs:** json/synthesized-answer-a-d-w

**Candidate leaf skills:** `temperature-controlled-answer-synthesis` (primary), `llm-response-quality-annotation-aggregation`, `llm-multi-agent-orchestration`

**Tools (primary):** AgriIR, DuckDuckGo, FAISS, Gemma3:1B, Gemma3:27B, Llama3.2:3B, Ollama

**Other candidate tools:** ChatGPT-4o, GPT-OSS-120B, Gemini 2.5 Flash, DeepInfra API, OpenRouter, Python, WattCouncil, conda, pip

**Grounding:** 2 KB(s); DOIs: 10.1007/978-3-032-21324-2_37, 10.1145/3744256.3812595

### Stage 5 — insert_citations

**Goal:** attach each supported answer sentence to a retrieved source chunk

**EDAM operation:** http://edamontology.org/operation_0282

**Inputs:** json/synthesized-answer-a-d-w · **Outputs:** json/cited-answer-with-index

**Candidate leaf skills:** `agricultural-answer-citation-insertion` (primary), `sentence-embedding-similarity-scoring`, `citation-source-attribution`, `post-hoc-answer-grounding`

**Tools (primary):** AgriIR, FAISS, Sentence Transformer

**Other candidate tools:** Ollama

**Grounding:** 1 KB(s); DOIs: 10.1007/978-3-032-21324-2_37

### Stage 6 — score_grounded_answers

**Goal:** aggregate human answer and citation judgments for the fixed query set

**EDAM operation:** http://edamontology.org/operation_0306

**Inputs:** json/cited-answer-with-index, csv/agricultural-answer-citation-human-annotations · **Outputs:** csv/grounded-answer-scores

**Candidate leaf skills:** `citation-quality-scoring` (primary), `composite-score-compilation-across-model-conditions`, `per-query-composite-score-analysis`

**Tools (primary):** AgriIR, ChatGPT-4o, FAISS, GPT-OSS-120B, Gemini 2.5 Flash, Gemma3:1B, Gemma3:27B, Llama3.2:3B


**Grounding:** 1 KB(s); DOIs: 10.1007/978-3-032-21324-2_37

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
