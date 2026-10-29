---
name: rag-pipeline-benchmark-execution
description: Use when a small local LLM must be benchmarked against large commercial
  models on domain-specific Q&A, requiring per-query outputs, citations, and human-annotated
  composite scores.
when_to_use_negative:
- The task requires general-domain (non-domain-specific) retrieval where no curated
  domain knowledge base exists to build the FAISS index from
- The goal is to train or fine-tune models rather than evaluate and compare existing
  RAG configurations
- There is no capacity for human annotation, since answer/citation/composite scores
  in this protocol come from human annotators (30 evaluators)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3071
- http://edamontology.org/topic_3366
tools:
- name: AgriIR
  role: 6-stage configurable RAG pipeline executed over the benchmark queries
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: Ollama
  role: Serves the local models (gemma3:1b, llama3.2:3b, gemma3:27b) used in the framework
    variants
  repo: https://github.com/ollama/ollama
- name: FAISS
  role: Local vector search over the 15,247-entry knowledge base during the parallel
    retrieval stage
  repo: https://github.com/facebookresearch/faiss
- name: ChatGPT-4o
  role: Standalone large-model baseline queried with the same 191 queries
- name: Gemini 2.5 Flash
  role: Standalone large-model baseline queried with the same 191 queries
- name: GPT-OSS-120B
  role: Standalone large-model baseline queried with the same 191 queries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:103121635552bd8180e7e21b8b267a3f6f4296562e791e89774e4e818f6810b9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Shuvam-Banerji-Seal/AgriIR
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3071
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# rag-pipeline-benchmark-execution

## Summary

Executes a 6-stage RAG pipeline (query refinement → sub-query decomposition → parallel FAISS + web retrieval → agent enhancement → synthesis → citation insertion) across multiple model variants and standalone baselines over an annotated benchmark, then aggregates human-annotated answer, citation, and composite scores per system condition.

## When to use

Use when a small local LLM must be benchmarked against large commercial models on domain-specific Q&A, requiring per-query outputs, citations, and human-annotated composite scores.

## When NOT to use

- The task requires general-domain (non-domain-specific) retrieval where no curated domain knowledge base exists to build the FAISS index from
- The goal is to train or fine-tune models rather than evaluate and compare existing RAG configurations
- There is no capacity for human annotation, since answer/citation/composite scores in this protocol come from human annotators (30 evaluators)

## Inputs

- 191 agricultural policy benchmark queries
- AgriIR knowledge base: 15,247 entries, JSONL, ~1.9GB (huggingface.co/datasets/ShuvBan/AgriIR_dataset)
- FAISS vector database built from the knowledge base for with-database variants
- Local Ollama models: llama3.2:3b, gemma3:1b, gemma3:27b
- Human annotation scores from 30 annotators per system condition

## Outputs

- Per-query generated answers with inserted citations for each of the 4 AgriIR variants and 3 standalone baselines
- Per-condition aggregate metrics: answer quality, citation quality, perfect-citation rate, composite score (mean ± SD) with pairwise p-values
- benchmark_scores.csv comparison table

## How to apply

Install AgriIR via install_agriir.sh, pulling gemma3:1b and llama3.2:3b through Ollama, and download the 15,247-entry AgriIR knowledge base (JSONL, ~1.9GB) from HuggingFace (ShuvBan/AgriIR_dataset) to build a FAISS vector database for the with-database variants. Run the four AgriIR variants (Llama3.2:3B with and without database, Gemma3:1B without database, Gemma3:27B with database) over the 191 agricultural policy queries using the 6-stage pipeline with controlled temperatures: query refinement T=0.1, sub-query decomposition T=0.5, synthesis T=0.2, and citation insertion only for sources with similarity ≥ 0.75. Query the three standalone baselines (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B) with the same 191 queries for a like-for-like comparison. Aggregate human-annotated scores (30 annotators) for answer quality, citation quality, perfect-citation rate, and composite score per condition, and test equivalence/significance (e.g. AgriIR_Gemma-3-27B 0.820±0.208 vs ChatGPT-4o 0.840±0.233, p=0.493). Write the per-condition score table to benchmark_scores.csv.

## Related tools

- **AgriIR** (6-stage configurable RAG pipeline executed over the benchmark queries) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **Ollama** (Serves the local models (gemma3:1b, llama3.2:3b, gemma3:27b) used in the framework variants) — https://github.com/ollama/ollama
- **FAISS** (Local vector search over the 15,247-entry knowledge base during the parallel retrieval stage) — https://github.com/facebookresearch/faiss
- **ChatGPT-4o** (Standalone large-model baseline queried with the same 191 queries)
- **Gemini 2.5 Flash** (Standalone large-model baseline queried with the same 191 queries)
- **GPT-OSS-120B** (Standalone large-model baseline queried with the same 191 queries)

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR && bash install_agriir.sh
```

## Evaluation signals

- Composite score for the top AgriIR variant falls near 0.820 ± 0.208 and is statistically equivalent to ChatGPT-4o (0.840 ± 0.233, p = 0.493)
- AgriIR variants achieve 73-84% perfect citations while baseline models show no perfect-citation capability
- AgriIR_Gemma-3-27B significantly outperforms GPT-OSS-120B (0.705 ± 0.246, p < 0.001) and Gemini Flash (0.779 ± 0.250)
- Citations are inserted only for retrieved sources with similarity ≥ 0.75; stage temperatures match T=0.1 (refinement), T=0.5 (decomposition), T=0.2 (synthesis)
- benchmark_scores.csv contains one row per system condition with answer quality, citation quality, perfect-citation rate, and composite score

## Limitations

- The composite-score comparison depends on 30 human annotators; without equivalent annotation effort, the benchmark metrics cannot be reproduced
- With-database variants require building a FAISS index over the ~1.9GB, 15,247-entry JSONL knowledge base; the no-database variants will not match those scores
- No discussion/limitations section was available in the source document, so additional reproducibility gaps could not be extracted

## Evidence
