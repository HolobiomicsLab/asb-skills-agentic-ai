---
name: temperature-controlled-answer-synthesis
description: Use when a retrieval-augmented pipeline has already collected parallel
  evidence corpora (local vector search D and web results W) and needs a deterministic,
  low-temperature final answer synthesis step in a resource-constrained, domain-specific
  setting.
when_to_use_negative:
- The goal is to insert source citations into an already-synthesized answer — that
  is Stage-6 (Citation Insert with similarity >= 0.75), not synthesis
- The task is query understanding (refinement at temperature 0.1 or decomposition
  into 3-5 sub-queries at temperature 0.5) rather than final answer generation
- A high-temperature, creative generation task is needed — the 0.2 temperature setting
  is designed for faithful, evidence-grounded answers, not divergence
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3482
tools:
- name: AgriIR
  role: Configurable 6-stage RAG framework that hosts this Stage-5 synthesis step;
    model and temperature are set via configuration, not code
  repo: github.com/Shuvam-Banerji-Seal/AgriIR
- name: Llama3.2:3B
  role: Default synthesis model producing answer A at temperature 0.2
- name: Gemma3:27B
  role: High-quality synthesis model variant (composite score 0.820 ± 0.208, statistically
    equivalent to ChatGPT-4o)
- name: Gemma3:1B
  role: Upstream refinement/decomposition model whose outputs (refined query, sub-queries)
    feed synthesis
- name: FAISS
  role: Local vector search producing corpus D over the AgriIR dataset embedded with
    Qwen3-Embedding-8B
  repo: github.com/facebookresearch/faiss
- name: Ollama
  role: Runtime serving the synthesis LLM locally
  repo: ollama/ollama
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:bfcf45023b78ad525b4964a554e7d6afdf447133ea959da1a15f2f43304f2e58
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - AgriIR
  - DuckDuckGo
  - FAISS
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  - Ollama
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3482
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# temperature-controlled-answer-synthesis

## Summary

Stage-5 of the AgriIR pipeline: generate the final grounded answer A at temperature 0.2 from the agent-enhanced union of the local FAISS corpus D and web corpus W, using a configurable Ollama-hosted synthesis model (llama3.2:3b default; gemma3:27b for high quality), then write the answer plus both corpora to synthesized_answer without citation insertion.

## When to use

Use when a retrieval-augmented pipeline has already collected parallel evidence corpora (local vector search D and web results W) and needs a deterministic, low-temperature final answer synthesis step in a resource-constrained, domain-specific setting.

## When NOT to use

- The goal is to insert source citations into an already-synthesized answer — that is Stage-6 (Citation Insert with similarity >= 0.75), not synthesis
- The task is query understanding (refinement at temperature 0.1 or decomposition into 3-5 sub-queries at temperature 0.5) rather than final answer generation
- A high-temperature, creative generation task is needed — the 0.2 temperature setting is designed for faithful, evidence-grounded answers, not divergence

## Inputs

- Enhanced evidence corpus D from FAISS vector search over the AgriIR dataset (15,247 entries, embedded with Qwen3-Embedding-8B)
- Web evidence corpus W from DuckDuckGo search with LLM filtering
- Agent-enhanced query context (keywords from the domain-agent catalogue, max_agents: 6)
- Configuration selecting the synthesis model (llama3.2:3b default, gemma3:27b for high quality) via Ollama

## Outputs

- synthesized_answer file (JSON) with non-empty answer field A and both retrieved corpora D and W
- Final answer passed on to Stage-6 Citation Insert (similarity >= 0.75), which is not part of this skill

## How to apply

After Stages 1-4 (query refinement with Gemma3:1B at temperature 0.1, sub-query decomposition into 3-5 sub-queries at temperature 0.5, parallel FAISS/DuckDuckGo retrieval, and agent enhancement with up to 6 domain agents such as crop_specialist and disease_expert), feed the enhanced evidence corpora D and W into the synthesis model. Select the synthesis model from configuration rather than hard-coding it: llama3.2:3b is the default, while gemma3:27b is chosen for high-quality runs (AgriIR_Gemma-3-27B scored 0.820 ± 0.208, statistically equivalent to ChatGPT-4o). Set the sampling temperature to 0.2 to keep generation faithful to the retrieved evidence. Write the result to a synthesized_answer file containing the answer A plus both corpora D and W; validation requires the file to exist, parse as JSON, and contain a non-empty A, D, and W. Deliberately do NOT apply citation insertion at this stage — that is the separate Stage-6 step gated on similarity >= 0.75.

## Related tools

- **AgriIR** (Configurable 6-stage RAG framework that hosts this Stage-5 synthesis step; model and temperature are set via configuration, not code) — github.com/Shuvam-Banerji-Seal/AgriIR
- **Llama3.2:3B** (Default synthesis model producing answer A at temperature 0.2)
- **Gemma3:27B** (High-quality synthesis model variant (composite score 0.820 ± 0.208, statistically equivalent to ChatGPT-4o))
- **Gemma3:1B** (Upstream refinement/decomposition model whose outputs (refined query, sub-queries) feed synthesis)
- **FAISS** (Local vector search producing corpus D over the AgriIR dataset embedded with Qwen3-Embedding-8B) — github.com/facebookresearch/faiss
- **Ollama** (Runtime serving the synthesis LLM locally) — ollama/ollama

## Examples

```
ollama run llama3.2:3b --temperature 0.2  # Stage-5 synthesis in AgriIR (config-selectable model: llama3.2:3b or gemma3:27b)
```

## Evaluation signals

- The synthesized_answer output file exists and parses as valid JSON
- The answer field A is non-empty
- Both retrieved corpora D and W are present and non-empty in the output file
- Synthesis ran at temperature 0.2 with the configured model (llama3.2:3b or gemma3:27b), per the pipeline configuration
- No citation insertion was applied during this stage (Stage-6 citation insertion with similarity >= 0.75 is deferred)

## Limitations

- Citation insertion is intentionally out of scope for this stage; sources are only attached later when similarity >= 0.75
- The synthesis model is decoupled from retrieval, so output quality depends on the selected model — the default llama3.2:3b is small and lower quality than gemma3:27b
- The paper's discussion section is absent from the source material, so additional limitations could not be extracted

## Evidence

- [methods] Stage-5 Synthesis T=0.2: [evidence span withheld — anchor status: not_found]
