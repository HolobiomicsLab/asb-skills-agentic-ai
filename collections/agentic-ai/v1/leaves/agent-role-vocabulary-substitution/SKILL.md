---
name: agent-role-vocabulary-substitution
description: Use when adapting an existing modular RAG pipeline to a second domain
  (e.g., health instead of agriculture) and the goal is vertical retargeting purely
  via reconfiguration of agents, prompts, and scoring heuristics.
when_to_use_negative:
- When the task requires modifying the retrieval architecture, adding new pipeline
  stages, or fine-tuning models — the skill explicitly covers reconfiguration alone,
  not architecture or weight changes
- When the underlying pipeline stages (query refinement, retrieval, synthesis, evaluation)
  themselves need to change rather than the domain artifacts injected via configuration
- When sufficient target-domain data exists and training a domain-specific model is
  preferable to configuration-driven retargeting
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: AgriIR
  role: Configurable 6-stage RAG framework being retargeted via declarative configuration
  repo: github.com/Shuvam-Banerji-Seal/AgriIR
- name: FAISS
  role: Local vector search used in Stage-3 parallel retrieval over the domain knowledge
    base
  repo: github.com/facebookresearch/faiss
- name: Gemma3:1B
  role: Local Ollama model for Stage-1 query refinement after retargeting
- name: Llama3.2:3B
  role: Local Ollama model for Stage-5 synthesis after retargeting
- name: Ollama
  role: Runtime for pulling and serving the local models (ollama pull gemma3:1b &&
    ollama pull llama3.2:3b)
  repo: github.com/ollama/ollama
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:1a65778d0723a20bbf35fd914185ae8fc16051affffe43e14146d8f575e4b1da
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - AgriIR
  - FAISS
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  - Ollama
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# agent-role-vocabulary-substitution

## Summary

Retargets a configurable RAG framework (AgriIR) to a new knowledge domain by substituting agent role vocabularies and source allow-lists in declarative configuration files only, with no code changes or model fine-tuning.

## When to use

Use when adapting an existing modular RAG pipeline to a second domain (e.g., health instead of agriculture) and the goal is vertical retargeting purely via reconfiguration of agents, prompts, and scoring heuristics.

## When NOT to use

- When the task requires modifying the retrieval architecture, adding new pipeline stages, or fine-tuning models — the skill explicitly covers reconfiguration alone, not architecture or weight changes
- When the underlying pipeline stages (query refinement, retrieval, synthesis, evaluation) themselves need to change rather than the domain artifacts injected via configuration
- When sufficient target-domain data exists and training a domain-specific model is preferable to configuration-driven retargeting

## Inputs

- AgriIR configuration files: config.yml and config/enhanced_rag_config.yaml (agri_bot_searcher/)
- Target-domain agent role vocabulary (replacing crop_specialist, disease_expert, economics_analyst, climate_researcher)
- Target-domain source allow-list (replacing extension.org, fao.org, usda.gov)
- Local Ollama models: gemma3:1b (query refinement) and llama3.2:3b (synthesis)
- Target-domain queries submitted to POST /api/query

## Outputs

- Retargeted RAG answers saved to retargeted_rag_answers with deterministic citations
- Executed configuration showing substituted agent roles and domain allow-list
- JSON responses from the 6-stage pipeline with web sources restricted to the retargeted domain allow-list

## How to apply

Clone the AgriIR repository and edit the configuration files config.yml and config/enhanced_rag_config.yaml under agri_bot_searcher/. Replace the agricultural agent role vocabulary (crop_specialist, disease_expert, economics_analyst, climate_researcher, max_agents: 6) with target-domain roles, and replace the search.agriculture_domains allow-list (extension.org, fao.org, usda.gov) with target-domain sources — editing configuration only. Pull the local Ollama models (gemma3:1b for query refinement, llama3.2:3b for synthesis), then start the full-RAG server (python3 agri_bot_searcher/src/enhanced_web_ui.py) and submit target-domain queries to the POST /api/query endpoint, which triggers the 6-stage pipeline (Query Refinement T=0.1, Sub-Query Decomposition T=0.5, Parallel Retrieval, Agent Enhancement, Synthesis T=0.2, Citation Insert). Validate that executed configuration contains no residual agricultural roles or domains and that citations are deterministic, restricted to sources with similarity ≥ 0.75. The rationale is that architectural intelligence (domain specialization, controlled sampling, multi-stage reasoning) compensates for parameter count, so the architecture itself is never modified.

## Related tools

- **AgriIR** (Configurable 6-stage RAG framework being retargeted via declarative configuration) — github.com/Shuvam-Banerji-Seal/AgriIR
- **FAISS** (Local vector search used in Stage-3 parallel retrieval over the domain knowledge base) — github.com/facebookresearch/faiss
- **Gemma3:1B** (Local Ollama model for Stage-1 query refinement after retargeting)
- **Llama3.2:3B** (Local Ollama model for Stage-5 synthesis after retargeting)
- **Ollama** (Runtime for pulling and serving the local models (ollama pull gemma3:1b && ollama pull llama3.2:3b)) — github.com/ollama/ollama

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR && ollama pull gemma3:1b && ollama pull llama3.2:3b && python3 agri_bot_searcher/src/enhanced_web_ui.py
```

## Evaluation signals

- Each saved answer in retargeted_rag_answers contains deterministic citations restricted to sources with similarity ≥ 0.75
- Retrieved web sources come only from the retargeted domain allow-list (no residual extension.org, fao.org, usda.gov entries)
- No agricultural-domain agent roles (crop_specialist, disease_expert, economics_analyst, climate_researcher) remain in the executed configuration
- Edits are confined to config.yml and config/enhanced_rag_config.yaml with no code changes or fine-tuning performed
- Pipeline stages execute with the declared temperatures (Stage-1 T=0.1, Stage-2 T=0.5, Stage-5 T=0.2) and produce 3–5 sub-queries per query

## Limitations

- Adaptation is demonstrated for high-level domain classes (agriculture, health, climate) where domain artifacts — prompts, agents, scoring heuristics — can be expressed declaratively; domains needing new retrieval logic may require code changes
- The source document contains no discussion-section text, so limitations, failure modes, and reproducibility gaps beyond this could not be extracted
- Retrieval depends on the target-domain source allow-list being well-chosen; poor allow-list selection directly limits web retrieval coverage

## Evidence

- [results] domain artifacts such as prompts, agents, and scoring heuristics are injected via configuration, enabling rapid vertical retargeting without fine-tuning or data collection: [evidence span withheld — anchor status: not_found]
- [intro] adaptable through reconfiguration alone: "This design-driven approach enables practitioners to adapt the framework to new domains, i.e. agriculture, health, and climate, through reconfiguration alone."
- [methods] citation insert threshold: "Citation | Insert sources where similarity ≥ 0.75"
- [intro] modular declarative stages: "decomposes the end-to-end process into modular, declarative stages i.e., query refinement, retrieval, synthesis, and evaluation"
- [intro] reproducibility via public codebase: [evidence span withheld — anchor status: not_found]
