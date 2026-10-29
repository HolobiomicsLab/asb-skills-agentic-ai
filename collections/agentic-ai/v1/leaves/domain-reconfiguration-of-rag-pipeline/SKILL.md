---
name: domain-reconfiguration-of-rag-pipeline
description: Use when a deployed AgriIR-style RAG pipeline must be moved to a new
  knowledge domain (e.g., health, climate) and the goal is rapid vertical retargeting
  via configuration edits alone, with no code changes, no fine-tuning, and no new
  training data collection.
when_to_use_negative:
- When the target domain requires different retrieval architecture, ranking logic,
  or model capability beyond what the fixed 6-stage AgriIR pipeline exposes — reconfiguration
  alone cannot change the architecture
- When high-quality target-domain fine-tuned models or domain training data are already
  available and fine-tuning is a viable option (this skill deliberately avoids fine-tuning
  and data collection)
- When the deployment requires GPU-heavy, high-resource RAG infrastructure — AgriIR
  is designed for low-resource, small-model settings (1B–3B parameters via Ollama)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3489
tools:
- name: AgriIR
  role: The configurable 6-stage RAG framework being retargeted via declarative configuration
    files
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: FAISS
  role: Local vector database used in Stage-3 parallel retrieval after domain retargeting
  repo: https://github.com/facebookresearch/faiss
- name: Gemma3:1B
  role: Ollama-served small model driving Stage-1 query refinement (T=0.1)
  repo: https://github.com/ollama/ollama
- name: Llama3.2:3B
  role: Ollama-served small model driving Stage-5 synthesis (T=0.2)
  repo: https://github.com/ollama/ollama
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:acd350bd24b33c86997b2329dad0d54c344af4540964e2edd1bbe9d7c0c0f665
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
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# domain-reconfiguration-of-rag-pipeline

## Summary

Retarget the AgriIR 6-stage RAG pipeline (query refinement → sub-query decomposition → parallel retrieval → agent enhancement → synthesis → citation insert) to a new knowledge domain purely through declarative reconfiguration of config.yml and enhanced_rag_config.yaml — swapping agent role vocabularies and domain source allow-lists — without modifying architecture or fine-tuning models.

## When to use

Use when a deployed AgriIR-style RAG pipeline must be moved to a new knowledge domain (e.g., health, climate) and the goal is rapid vertical retargeting via configuration edits alone, with no code changes, no fine-tuning, and no new training data collection.

## When NOT to use

- When the target domain requires different retrieval architecture, ranking logic, or model capability beyond what the fixed 6-stage AgriIR pipeline exposes — reconfiguration alone cannot change the architecture
- When high-quality target-domain fine-tuned models or domain training data are already available and fine-tuning is a viable option (this skill deliberately avoids fine-tuning and data collection)
- When the deployment requires GPU-heavy, high-resource RAG infrastructure — AgriIR is designed for low-resource, small-model settings (1B–3B parameters via Ollama)

## Inputs

- AgriIR repository (github.com/Shuvam-Banerji-Seal/AgriIR) with agri_bot_searcher/config.yml and agri_bot_searcher/config/enhanced_rag_config.yaml
- Target-domain agent role vocabulary replacing the agricultural roles (crop_specialist, disease_expert, economics_analyst, climate_researcher)
- Target-domain source allow-list replacing search.agriculture_domains (extension.org, fao.org, usda.gov)
- Ollama-served local models: gemma3:1b (query refinement) and llama3.2:3b (synthesis)
- Target-domain queries submitted to POST /api/query

## Outputs

- JSON responses from the /api/query endpoint with the 6-stage pipeline applied
- Cited answers saved to retargeted_rag_answers with deterministic citations restricted to similarity ≥ 0.75 sources
- Executed configuration showing only target-domain roles and allow-listed domains

## How to apply

Clone github.com/Shuvam-Banerji-Seal/AgriIR and edit only the configuration files (config.yml, config/enhanced_rag_config.yaml in agri_bot_searcher/): replace the agricultural agent role vocabulary (crop_specialist, disease_expert, economics_analyst, climate_researcher, max_agents: 6) with target-domain roles, and replace the search.agriculture_domains allow-list (extension.org, fao.org, usda.gov) with target-domain sources — no code edits or model fine-tuning. Pull the local models via Ollama (gemma3:1b for query refinement, llama3.2:3b for synthesis), start the full-RAG server (python3 agri_bot_searcher/src/enhanced_web_ui.py), and submit target-domain queries to POST /api/query to trigger the 6-stage pipeline (Stage-1 Query Refinement T=0.1; Stage-2 Sub-Query Decomposition T=0.5, 3-5 sub-queries; Stage-3 Parallel Retrieval over FAISS and DuckDuckGo; Stage-4 Agent Enhancement with domain keyword injection; Stage-5 Synthesis T=0.2; Stage-6 Citation Insert). Rationale: AgriIR's declarative stage decomposition means domain artifacts such as prompts, agents, and scoring heuristics are injected via configuration, so architectural design rather than model scale or retraining carries the domain specialization. Validate by checking every saved answer carries deterministic citations restricted to sources with similarity ≥ 0.75, web sources come only from the retargeted allow-list, and no agricultural roles or domains remain in the executed configuration.

## Related tools

- **AgriIR** (The configurable 6-stage RAG framework being retargeted via declarative configuration files) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **FAISS** (Local vector database used in Stage-3 parallel retrieval after domain retargeting) — https://github.com/facebookresearch/faiss
- **Gemma3:1B** (Ollama-served small model driving Stage-1 query refinement (T=0.1)) — https://github.com/ollama/ollama
- **Llama3.2:3B** (Ollama-served small model driving Stage-5 synthesis (T=0.2)) — https://github.com/ollama/ollama

## Examples

```
ollama pull gemma3:1b && ollama pull llama3.2:3b && python3 agri_bot_searcher/src/enhanced_web_ui.py  # then POST target-domain queries to /api/query
```

## Evaluation signals

- Every saved answer in retargeted_rag_answers contains deterministic citations restricted to sources with similarity ≥ 0.75
- All retrieved web sources come only from the retargeted-domain allow-list (no extension.org, fao.org, usda.gov hits)
- No agricultural-domain roles (crop_specialist, disease_expert, economics_analyst, climate_researcher) remain in the executed configuration
- No agricultural domains appear in search.* allow-list entries of the executed configuration
- Pipeline runs the full 6 stages with declared temperature settings (T=0.1 refinement, T=0.5 decomposition, T=0.2 synthesis) and 3-5 sub-queries per query

## Limitations

- Adaptation relies on domain artifacts (prompts, agents, scoring heuristics) being injectable via configuration; domains whose needs exceed the fixed stage structure or agent-count budget (max_agents: 6) cannot be accommodated by reconfiguration alone
- Validation in the source card is by structural checks (citations, allow-list containment, config residue), not by target-domain human evaluation — the paper's benchmark (191 annotated agricultural policy queries, 30 human annotators) covers agriculture only
- No discussion/limitations section text was available in the source document; no further reproducibility gaps could be extracted

## Evidence

- [other] domain artifacts such as prompts, agents, and scoring heuristics are injected via configuration, enabling rapid vertical retargeting without fine-tuning or data collection: [evidence span withheld — anchor status: not_found]
