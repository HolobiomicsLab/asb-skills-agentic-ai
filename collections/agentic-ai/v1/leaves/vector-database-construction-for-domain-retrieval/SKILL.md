---
name: vector-database-construction-for-domain-retrieval
description: 'Use when a RAG pipeline needs local domain-specific vector search: you
  have a JSONL knowledge corpus (e.g. ShuvBan/AgriIR_dataset) and must index it for
  FAISS retrieval alongside web results.'
when_to_use_negative:
- You are running the no-database AgriIR variants (Llama3.2:3B without database, Gemma3:1B
  without database), which by design skip local vector retrieval
- Your corpus is not domain-specific agricultural knowledge and the AgriIR citation-insertion
  threshold (similarity >= 0.75) has not been validated on your domain
- You need commercial large-model baselines only (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B),
  which are queried standalone without any local vector database
tools:
- name: FAISS
  role: Local vector similarity search library used to index the 15,247-entry agricultural
    knowledge base for Stage-3 Parallel Retrieval
  repo: https://github.com/facebookresearch/faiss
- name: AgriIR
  role: Configurable RAG framework whose installation script and pipeline consume
    the built FAISS index for with-database variants
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: ShuvBan/AgriIR_dataset
  role: Source JSONL knowledge corpus (15,247 entries, ~1.9GB) to be embedded and
    indexed into the FAISS database
  repo: https://huggingface.co/datasets/ShuvBan/AgriIR_dataset
- name: Ollama
  role: Installs the local models (gemma3:1b, llama3.2:3b) required by the AgriIR
    environment in which the index is built and queried
  repo: https://ollama.ai
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:5a49634b67188d35f41b5488c31234dcf1f82737dd1866877f891d97e2be7d62
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/facebookresearch/faiss
  tools:
  - AgriIR
  - ChatGPT-4o
  - FAISS
  - GPT-OSS-120B
  - Gemini 2.5 Flash
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  - Ollama
  - ShuvBan/AgriIR_dataset
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# vector-database-construction-for-domain-retrieval

## Summary

Builds a FAISS local vector database from the AgriIR agricultural knowledge base (15,247 entries, JSONL, ~1.9GB on HuggingFace) so that AgriIR's 'with-database' variants can run parallel FAISS + web retrieval over agricultural policy queries.

## When to use

Use when a RAG pipeline needs local domain-specific vector search: you have a JSONL knowledge corpus (e.g. ShuvBan/AgriIR_dataset) and must index it for FAISS retrieval alongside web results.

## When NOT to use

- You are running the no-database AgriIR variants (Llama3.2:3B without database, Gemma3:1B without database), which by design skip local vector retrieval
- Your corpus is not domain-specific agricultural knowledge and the AgriIR citation-insertion threshold (similarity >= 0.75) has not been validated on your domain
- You need commercial large-model baselines only (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B), which are queried standalone without any local vector database

## Inputs

- AgriIR knowledge base: 15,247 agricultural knowledge entries in JSONL format (~1.9GB) from huggingface.co/datasets/ShuvBan/AgriIR_dataset
- AgriIR repository (github.com/Shuvam-Banerji-Seal/AgriIR) with install_agriir.sh for environment setup
- Ollama-installed local models (gemma3:1b, llama3.2:3b) used by the surrounding pipeline

## Outputs

- FAISS vector index over the 15,247-entry agricultural corpus, used by Stage-3 Parallel Retrieval
- Retrieved source passages at similarity >= 0.75, consumed by Stage-6 Citation Insert

## How to apply

Clone the AgriIR repository (github.com/Shuvam-Banerji-Seal/AgriIR) and install dependencies via install_agriir.sh, pulling the local models gemma3:1b and llama3.2:3b through Ollama. Download the AgriIR knowledge base from HuggingFace (ShuvBan/AgriIR_dataset), a 15,247-entry JSONL corpus of ~1.9GB covering agricultural knowledge. Build the FAISS vector database over this corpus; this index is required only for the 'with-database' AgriIR variants (Llama3.2:3B with database and Gemma3:27B with database), while the no-database variants run without it. At query time the index serves Stage-3 Parallel Retrieval, combining local FAISS vector search with DuckDuckGo web retrieval, and downstream citation insertion only inserts sources where similarity >= 0.75, so embedding/index quality directly gates citation yield.

## Related tools

- **FAISS** (Local vector similarity search library used to index the 15,247-entry agricultural knowledge base for Stage-3 Parallel Retrieval) — https://github.com/facebookresearch/faiss
- **AgriIR** (Configurable RAG framework whose installation script and pipeline consume the built FAISS index for with-database variants) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **ShuvBan/AgriIR_dataset** (Source JSONL knowledge corpus (15,247 entries, ~1.9GB) to be embedded and indexed into the FAISS database) — https://huggingface.co/datasets/ShuvBan/AgriIR_dataset
- **Ollama** (Installs the local models (gemma3:1b, llama3.2:3b) required by the AgriIR environment in which the index is built and queried) — https://ollama.ai

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR && bash install_agriir.sh
```

## Evaluation signals

- The index covers all 15,247 JSONL entries from the ~1.9GB AgriIR_dataset download with no load errors
- Stage-3 Parallel Retrieval returns both FAISS local hits and DuckDuckGo web hits for the 191 agricultural policy queries
- Citation insertion yields sources at similarity >= 0.75; with-database variants should reach the reported 73-84% perfect-citation range
- The with-database AgriIR_Gemma-3-27B variant reproduces a composite score near 0.820 +/- 0.208 on the 191-query benchmark, confirming index-driven retrieval quality
- No-database variants run successfully without the index, confirming the index is wired only into the with-database configurations

## Limitations

- The knowledge base is domain-specific (agriculture); the FAISS index built from it is not validated for other domains without reconfiguration of the framework
- No discussion/limitations section was present in the source document, so reproducibility gaps for the indexing step could not be extracted
- The index is only meaningful for the with-database variants; results for standalone baselines (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B) do not depend on it

## Evidence
