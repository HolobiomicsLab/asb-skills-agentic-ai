---
name: citation-quality-scoring
description: Use when comparing RAG or QA systems on grounded answer quality and you
  must score, per query, whether answers are supported by verifiable citations (e.g.,
  inserted at similarity ≥ 0.75) across multiple system conditions.
when_to_use_negative:
- You only need raw retrieval metrics (e.g., FAISS similarity recall) without human
  judgments of grounded answer quality
- Your system conditions generate no citations (e.g., closed-book standalone answering)
  — perfect-citation rate is undefined and the citation-quality axis is meaningless
- The query set is not fixed/shared across systems, so cross-condition score comparison
  is invalid
edam_operation: http://edamontology.org/operation_0306
tools:
- name: AgriIR
  role: 6-stage RAG pipeline whose Stage-6 Citation Insert supplies the citations
    (similarity ≥ 0.75) that are scored for citation quality
  repo: github.com/Shuvam-Banerji-Seal/AgriIR
- name: FAISS
  role: Local vector search feeding retrieved sources whose similarity threshold gates
    citation insertion
  repo: github.com/facebookresearch/faiss
- name: ChatGPT-4o
  role: Standalone baseline scored with the same annotation protocol for comparison
    (composite 0.840 ± 0.233)
- name: GPT-OSS-120B
  role: Standalone baseline scored under the same protocol (composite 0.705 ± 0.246,
    p < 0.001 vs AgriIR_Gemma-3-27B)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:c7efc6dfa98e5870085883be9494bdf7dfd9d40bae8ff2c03bad4da0aa42f127
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - AgriIR
  - ChatGPT-4o
  - FAISS
  - GPT-OSS-120B
  - Gemini 2.5 Flash
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# citation-quality-scoring

## Summary

A human-annotation scoring procedure from the AgriIR paper that evaluates both answer quality and citation quality (including a perfect-citation rate) for RAG systems over 191 agricultural policy queries, aggregating them into a composite score per system condition.

## When to use

Use when comparing RAG or QA systems on grounded answer quality and you must score, per query, whether answers are supported by verifiable citations (e.g., inserted at similarity ≥ 0.75) across multiple system conditions.

## When NOT to use

- You only need raw retrieval metrics (e.g., FAISS similarity recall) without human judgments of grounded answer quality
- Your system conditions generate no citations (e.g., closed-book standalone answering) — perfect-citation rate is undefined and the citation-quality axis is meaningless
- The query set is not fixed/shared across systems, so cross-condition score comparison is invalid

## Inputs

- Responses of each system condition to the same 191 annotated agricultural policy queries
- Human annotation scores from 30 annotators per response (answer quality, citation quality, perfect-citation flag)
- Per-response citation lists produced by Stage-6 Citation Insert (sources with similarity ≥ 0.75)

## Outputs

- Per-condition score table (answer quality, citation quality, perfect-citation rate, composite score with mean ± SD), e.g. written to benchmark_scores.csv
- Statistical comparison results between conditions (e.g., p-values such as p = 0.493, p < 0.001)

## How to apply

Run each system condition (e.g., four AgriIR variants — Llama3.2:3B with/without database, Gemma3:1B, Gemma3:27B with database — plus standalone baselines ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B) over the same 191 agricultural policy queries. Have human annotators score each response for answer quality, citation quality, and whether citations are 'perfect' (correct source, correct claim). Aggregate the per-condition means and standard deviations (e.g., AgriIR_Gemma-3-27B composite 0.820 ± 0.208 vs ChatGPT-4o 0.840 ± 0.233, p = 0.493) and test differences statistically (p < 0.001 marks significant outperformance). Rationale: citation quality distinguishes domain-specific RAG from standalone LLMs — AgriIR variants achieved 73–84% perfect citations, a capability the baselines lack — so answer-only scoring misses the key architectural benefit.

## Related tools

- **AgriIR** (6-stage RAG pipeline whose Stage-6 Citation Insert supplies the citations (similarity ≥ 0.75) that are scored for citation quality) — github.com/Shuvam-Banerji-Seal/AgriIR
- **FAISS** (Local vector search feeding retrieved sources whose similarity threshold gates citation insertion) — github.com/facebookresearch/faiss
- **ChatGPT-4o** (Standalone baseline scored with the same annotation protocol for comparison (composite 0.840 ± 0.233))
- **GPT-OSS-120B** (Standalone baseline scored under the same protocol (composite 0.705 ± 0.246, p < 0.001 vs AgriIR_Gemma-3-27B))

## Examples

```
git clone https://github.com/Shuvam-Banerji-Seal/AgriIR.git && cd AgriIR && bash install_agriir.sh
```

## Evaluation signals

- Every condition is scored on the same 191 queries with 30 human annotators, so per-condition means and SDs are directly comparable
- Composite score falls in [0,1]; reported reference values are AgriIR_Gemma-3-27B 0.820 ± 0.208 and ChatGPT-4o 0.840 ± 0.233
- Perfect-citation rate for AgriIR variants should land in the reported 73–84% band; baselines should score near zero on this axis
- Statistical equivalence or difference between conditions is confirmed with reported p-values (e.g., p = 0.493 equivalence; p < 0.001 significance)
- The final per-condition table is written to benchmark_scores.csv with columns for answer quality, citation quality, perfect-citation rate, and composite score

## Limitations

- The paper's discussion section is absent from the source document, so known limitations of the scoring protocol could not be extracted
- Citation-quality advantage was only demonstrated for agricultural policy queries in an Indian-agriculture testbed; transfer to other domains is unverified
- Scoring depends on human annotators (30 evaluators), introducing annotator subjectivity and cost compared to automatic metrics

## Evidence
