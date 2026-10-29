---
name: six-stage-retrieval-augmented-answer-synthesis
description: Use when a query must be answered with grounded, cited JSON responses
  from a domain knowledge base plus restricted web search, and reliability must come
  from architectural design (controlled sampling, multi-stage reasoning) rather than
  model scale.
when_to_use_negative:
- The task requires open-ended generation or chat without evidence-grounded, traceable
  citations
- No FAISS vector index or domain allow-list sources can be supplied for the target
  domain, so Stage-3 parallel retrieval has nothing to ground on
- A GPU-heavy or API-only commercial setup is already acceptable and the goal is raw
  model scale rather than architectural reliability in resource-constrained settings
edam_operation: http://edamontology.org/operation_0306
tools:
- name: AgriIR
  role: Configurable 6-stage RAG framework whose declarative stages (refinement, decomposition,
    retrieval, enhancement, synthesis, citation) execute the pipeline
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: FAISS
  role: Local vector similarity search backend used in Stage-3 parallel retrieval
  repo: https://github.com/facebookresearch/faiss
- name: Ollama
  role: Serves the local gemma3:1b and llama3.2:3b models used for query refinement
    and synthesis
  repo: https://github.com/ollama/ollama
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:7411791ecb2aa3038fd8be2a9bf215f4aeb84c8215de5996fe3bfb7b917d1ea8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Shuvam-Banerji-Seal/AgriIR
  tools:
  - AgriIR
  - FAISS
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  - Ollama
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# six-stage-retrieval-augmented-answer-synthesis

## Summary

Runs AgriIR's 6-stage RAG pipeline (query refinement, sub-query decomposition, parallel retrieval, agent enhancement, synthesis, citation insert) to produce evidence-grounded, deterministically cited answers using small local models (Gemma3:1B, Llama3.2:3B) served via Ollama.

## When to use

Use when a query must be answered with grounded, cited JSON responses from a domain knowledge base plus restricted web search, and reliability must come from architectural design (controlled sampling, multi-stage reasoning) rather than model scale.

## When NOT to use

- The task requires open-ended generation or chat without evidence-grounded, traceable citations
- No FAISS vector index or domain allow-list sources can be supplied for the target domain, so Stage-3 parallel retrieval has nothing to ground on
- A GPU-heavy or API-only commercial setup is already acceptable and the goal is raw model scale rather than architectural reliability in resource-constrained settings

## Inputs

- Natural-language domain query submitted to POST /api/query
- FAISS vector index built from an agricultural knowledge corpus (e.g. AgriIR_dataset, 15,247 entries on HuggingFace)
- Configuration files: agri_bot_searcher/config.yml and agri_bot_searcher/config/enhanced_rag_config.yaml
- Local Ollama models: gemma3:1b (query refinement) and llama3.2:3b (synthesis)

## Outputs

- JSON response per query containing the synthesized answer with deterministic citations
- Cited answers file (e.g. retargeted_rag_answers.csv) restricted to sources with similarity >= 0.75

## How to apply

Clone the AgriIR repository and load config.yml and config/enhanced_rag_config.yaml from agri_bot_searcher/, pulling local models via Ollama (ollama pull gemma3:1b && ollama pull llama3.2:3b). Start the server with python3 agri_bot_searcher/src/enhanced_web_ui.py and submit queries to POST /api/query, which triggers the 6-stage pipeline: Stage-1 Query Refinement at T=0.1, Stage-2 Sub-Query Decomposition at T=0.5 generating 3-5 sub-queries, Stage-3 Parallel Retrieval over the FAISS vector DB and DuckDuckGo web search, Stage-4 Agent Enhancement with domain keyword injection, Stage-5 Synthesis at T=0.2, and Stage-6 Citation Insert of sources with similarity >= 0.75. Domain artifacts (agent role vocabulary such as crop_specialist/disease_expert/economics_analyst/climate_researcher, max_agents: 6, and the search.agriculture_domains allow-list of extension.org, fao.org, usda.gov) are injected via configuration only, so retargeting to a new domain requires no code changes or fine-tuning. Collect JSON responses containing the synthesized answer plus citations, and save cited answers (e.g. to retargeted_rag_answers.csv) for validation.

## Related tools

- **AgriIR** (Configurable 6-stage RAG framework whose declarative stages (refinement, decomposition, retrieval, enhancement, synthesis, citation) execute the pipeline) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **FAISS** (Local vector similarity search backend used in Stage-3 parallel retrieval) — https://github.com/facebookresearch/faiss
- **Ollama** (Serves the local gemma3:1b and llama3.2:3b models used for query refinement and synthesis) — https://github.com/ollama/ollama

## Examples

```
ollama pull gemma3:1b && ollama pull llama3.2:3b && python3 agri_bot_searcher/src/enhanced_web_ui.py
```

## Evaluation signals

- Every saved answer contains deterministic citations restricted to sources with similarity >= 0.75
- Retrieved web sources come only from the configured domain allow-list (e.g. extension.org, fao.org, usda.gov)
- All six stages executed with the prescribed sampling temperatures (T=0.1 refinement, T=0.5 decomposition, T=0.2 synthesis) and 3-5 sub-queries in Stage-2
- Citation quality in the expected range: AgriIR models achieved 73-84% perfect citations in the paper's evaluation over 191 agricultural policy queries
- No out-of-domain roles or domains remain in the executed configuration after retargeting

## Limitations

- Reliability depends on domain specialization and small models (1B-3B parameters); composite scores carry high variance (e.g. 0.820 ± 0.208 for AgriIR_Gemma-3-27B), so per-query quality can vary widely
- Citation gating at similarity >= 0.75 and the allow-list restrict recall; queries whose evidence lives outside indexed or allow-listed sources will be under-supported
- No discussion/limitations section text was present in the source document and no changelog was found, so additional reproducibility gaps could not be extracted

## Evidence

- [intro] Architectural rationale over parameter scale: [evidence span withheld — anchor status: not_found]
