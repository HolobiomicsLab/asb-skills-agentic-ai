---
name: web-search-domain-allowlist-configuration
description: Use when adapting a multi-stage RAG pipeline to a second knowledge domain
  (e.g. health, climate) and you need retrieval restricted to trusted, domain-specific
  web sources via configuration only.
when_to_use_negative:
- When the target domain requires new retrieval logic, new pipeline stages, or fine-tuning
  — this skill covers declarative reconfiguration only, not architectural modification
- When no trusted domain-specific web sources can be enumerated for an allow-list
  (the guarantee that retrieved web sources come only from the allow-list depends
  on it)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3372
- http://edamontology.org/topic_0219
tools:
- name: AgriIR
  role: Configurable 6-stage RAG framework whose pipeline stages and domain artifacts
    are injected via configuration
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: FAISS
  role: Local vector search backend used in Stage-3 Parallel Retrieval alongside web
    search
  repo: https://github.com/facebookresearch/faiss
- name: Ollama
  role: Runtime for the local models (gemma3:1b for query refinement, llama3.2:3b
    for synthesis)
  repo: https://github.com/ollama/ollama
- name: Gemma3:1B
  role: Stage-1 query refinement model (T=0.1)
- name: Llama3.2:3B
  role: Stage-5 synthesis model (T=0.2)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:e8797f71c210a96839e394f87d498caddc457a41bad060062a979831b34460c9
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3372
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# web-search-domain-allowlist-configuration

## Summary

Retarget a declarative RAG framework (AgriIR) to a new knowledge domain purely by editing configuration: swap the agent role vocabulary and the web-search domain allow-list without touching code or fine-tuning models.

## When to use

Use when adapting a multi-stage RAG pipeline to a second knowledge domain (e.g. health, climate) and you need retrieval restricted to trusted, domain-specific web sources via configuration only.

## When NOT to use

- When the target domain requires new retrieval logic, new pipeline stages, or fine-tuning — this skill covers declarative reconfiguration only, not architectural modification
- When no trusted domain-specific web sources can be enumerated for an allow-list (the guarantee that retrieved web sources come only from the allow-list depends on it)

## Inputs

- AgriIR repository clone (github.com/Shuvam-Banerji-Seal/AgriIR)
- Declarative configuration files: config.yml and config/enhanced_rag_config.yaml from agri_bot_searcher/
- Target-domain role vocabulary (replacement agent roles, max_agents: 6)
- Target-domain web source allow-list (replacing search.agriculture_domains: extension.org, fao.org, usda.gov)
- Local Ollama models: gemma3:1b and llama3.2:3b

## Outputs

- Retargeted configuration files with no agricultural-domain roles or domains remaining
- JSON responses from POST /api/query executing the 6-stage pipeline
- Cited answers saved to retargeted_rag_answers with deterministic citations restricted to allow-listed sources with similarity >= 0.75

## How to apply

Clone the AgriIR repository and edit the declarative configuration files config.yml and config/enhanced_rag_config.yaml in agri_bot_searcher/ — no code changes or fine-tuning. Replace the agent role vocabulary (crop_specialist, disease_expert, economics_analyst, climate_researcher, max_agents: 6) with target-domain roles, and replace the search.agriculture_domains allow-list (extension.org, fao.org, usda.gov) with the target domain's trusted sources. Pull the local Ollama models (gemma3:1b for query refinement, llama3.2:3b for synthesis), start the full-RAG server (python3 agri_bot_searcher/src/enhanced_web_ui.py), and submit target-domain queries to POST /api/query. The 6-stage pipeline (query refinement T=0.1, sub-query decomposition T=0.5 with 3-5 sub-queries, parallel retrieval over FAISS and DuckDuckGo, agent enhancement, synthesis T=0.2, citation insert with similarity >= 0.75) executes identically; only the injected domain artifacts differ. This works because AgriIR decomposes the end-to-end process into modular, declarative stages controlled through configuration rather than hard-coded logic.

## Related tools

- **AgriIR** (Configurable 6-stage RAG framework whose pipeline stages and domain artifacts are injected via configuration) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **FAISS** (Local vector search backend used in Stage-3 Parallel Retrieval alongside web search) — https://github.com/facebookresearch/faiss
- **Ollama** (Runtime for the local models (gemma3:1b for query refinement, llama3.2:3b for synthesis)) — https://github.com/ollama/ollama
- **Gemma3:1B** (Stage-1 query refinement model (T=0.1))
- **Llama3.2:3B** (Stage-5 synthesis model (T=0.2))

## Examples

```
ollama pull gemma3:1b && ollama pull llama3.2:3b && python3 agri_bot_searcher/src/enhanced_web_ui.py
```

## Evaluation signals

- Each saved answer in retargeted_rag_answers contains deterministic citations restricted to sources with similarity >= 0.75
- All retrieved web sources come only from the retargeted domain allow-list
- No agricultural-domain roles (crop_specialist, disease_expert, economics_analyst, climate_researcher) or domains (extension.org, fao.org, usda.gov) remain in the executed configuration
- Responses execute the full 6-stage pipeline (query refinement T=0.1, sub-query decomposition T=0.5 with 3-5 sub-queries, parallel retrieval, agent enhancement, synthesis T=0.2, citation insert) without code modification
- Pipeline behavior is identical to the agricultural deployment apart from the substituted configuration artifacts

## Limitations

- Adaptation is limited to artifacts expressible in configuration (prompts, agents, scoring heuristics); architectural changes or model fine-tuning are out of scope
- No discussion section text present in the source document; no limitations or reproducibility gaps were extracted from the authors
- Retargeting quality depends on the target-domain corpus (FAISS vector DB) and the quality of the substitute allow-list sources

## Evidence

- [intro] small models via architecture: [evidence span withheld — anchor status: not_found]
