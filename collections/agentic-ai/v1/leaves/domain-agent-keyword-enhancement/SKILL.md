---
name: domain-agent-keyword-enhancement
description: Use when retrieved evidence in a RAG pipeline lacks domain-specific terminology
  before synthesis — e.g., enriching AgriIR corpora D (FAISS) and W (DuckDuckGo) with
  up to 6 specialist-agent keywords ahead of Stage-5 synthesis.
when_to_use_negative:
- 'Citations must be attached to evidence at this point: source insertion is a separate
  Stage-6 operation (similarity >= 0.75) and this stage deliberately adds none'
- The query falls outside the curated agent roles (crop, disease, economics, climate)
  or in a domain whose catalogue has not been reconfigured — injection risks adding
  off-topic keywords
- Reproducing standalone-LLM baselines (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B
  style comparisons without retrieval) — agent enhancement would confound the baseline
  configuration
edam_operation: http://edamontology.org/operation_2422
tools:
- name: AgriIR
  role: 'Framework providing the declarative Stage-4 ''Agent Enhancement'' step and
    the domain-agent catalogue (max_agents: 6) whose keywords are injected into corpora
    D and W'
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: FAISS
  role: Vector search that produces the local corpus D which this stage enhances with
    domain-agent keywords
  repo: https://github.com/facebookresearch/faiss
- name: Ollama
  role: Runs the local LLMs surrounding this stage (Gemma3:1B refinement/decomposition;
    llama3.2:3b or gemma3:27b synthesis of the enhanced evidence)
  repo: https://github.com/ollama/ollama
- name: DuckDuckGo
  role: Web search (with LLM filtering) producing corpus W that is keyword-enhanced
    alongside D before synthesis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:83cd1a45399ebd261c3aedee385e1835ed50016dc8dd4e3f0c90d4d7746e6bd1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Shuvam-Banerji-Seal/AgriIR
  tools:
  - AgriIR
  - DuckDuckGo
  - FAISS
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  - Ollama
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# domain-agent-keyword-enhancement

## Summary

Stage-4 'Agent Enhancement' of the AgriIR 6-stage RAG pipeline: keywords from a declarative catalogue of up to six domain-specialist agents (crop_specialist, disease_expert, economics_analyst, climate_researcher) are injected into the FAISS-derived local corpus D and the LLM-filtered web corpus W so small synthesis models answer with domain-appropriate terminology.

## When to use

Use when retrieved evidence in a RAG pipeline lacks domain-specific terminology before synthesis — e.g., enriching AgriIR corpora D (FAISS) and W (DuckDuckGo) with up to 6 specialist-agent keywords ahead of Stage-5 synthesis.

## When NOT to use

- Citations must be attached to evidence at this point: source insertion is a separate Stage-6 operation (similarity >= 0.75) and this stage deliberately adds none
- The query falls outside the curated agent roles (crop, disease, economics, climate) or in a domain whose catalogue has not been reconfigured — injection risks adding off-topic keywords
- Reproducing standalone-LLM baselines (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B style comparisons without retrieval) — agent enhancement would confound the baseline configuration

## Inputs

- Corpus D: FAISS vector-search results over the AgriIR dataset (15,247 agricultural knowledge entries, embedded with Qwen3-Embedding-8B)
- Corpus W: DuckDuckGo web-search results filtered by an LLM
- Domain-agent catalogue configuration (declarative agent roles and keywords, max_agents: 6)
- Refined query and 3-5 decomposed sub-queries from Stages 1-2

## Outputs

- Agent-enhanced evidence: corpora D and W augmented with injected domain-agent keywords, passed to Stage-5 synthesis (T=0.2)
- Contributes to the synthesized_answer output file (JSON with non-empty answer field A plus retrieved corpora D and W)

## How to apply

Execute Agent Enhancement as Stage-4, immediately after Stage-3 parallel retrieval has produced corpus D (FAISS vector search over the 15,247-entry AgriIR dataset embedded with Qwen3-Embedding-8B) and corpus W (DuckDuckGo web search with LLM filtering). Select up to max_agents: 6 roles from the declarative domain-agent catalogue — e.g. crop_specialist, disease_expert, economics_analyst, climate_researcher — matched to the refined query and its 3-5 sub-queries (decomposed at temperature 0.5). Inject each selected agent's catalogue keywords into the evidence streams feeding Stage-5 synthesis (temperature 0.2; llama3.2:3b default, gemma3:27b for high quality). The decision point is role selection from the catalogue, not code: adapting the stage to a new domain such as health or climate is done by reconfiguration alone. Deliberately attach no citations at this step — source insertion is deferred to Stage-6, which only adds sources where similarity >= 0.75.

## Related tools

- **AgriIR** (Framework providing the declarative Stage-4 'Agent Enhancement' step and the domain-agent catalogue (max_agents: 6) whose keywords are injected into corpora D and W) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **FAISS** (Vector search that produces the local corpus D which this stage enhances with domain-agent keywords) — https://github.com/facebookresearch/faiss
- **Ollama** (Runs the local LLMs surrounding this stage (Gemma3:1B refinement/decomposition; llama3.2:3b or gemma3:27b synthesis of the enhanced evidence)) — https://github.com/ollama/ollama
- **DuckDuckGo** (Web search (with LLM filtering) producing corpus W that is keyword-enhanced alongside D before synthesis)

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR
```

## Evaluation signals

- Downstream Stage-5 output file exists, parses as JSON, and contains a non-empty answer field A plus both retrieved corpora D and W (the pipeline's defined validation)
- Number of applied agents never exceeds max_agents: 6 from the framework configuration
- Every injected keyword traces to a role declared in the domain-agent catalogue (crop_specialist, disease_expert, economics_analyst, climate_researcher)
- No citation metadata is present at this stage — citations (similarity >= 0.75) appear only after Stage-6 Citation Insert
- Adapting the agent set to a new domain requires only config-file changes; a diff of the adaptation should show configuration edits only, matching the 'reconfiguration alone' design claim

## Limitations

- The source article's discussion section is absent ('No changelog found.'), so stage-specific failure modes and limitations are not documented in the source
- Keyword coverage is bounded by the curated domain-agent catalogue (max_agents: 6); queries outside the agricultural roles may be poorly or wrongly enhanced
- Evidence leaves this stage unattributed; grounding quality depends on the downstream Stage-6 citation threshold (similarity >= 0.75)

## Evidence

- [intro] Domain specialization as architectural rationale: [evidence span withheld — anchor status: not_found]
- [methods] Stage-4 keyword injection from the domain-agent catalogue: [evidence span withheld — anchor status: not_found]
