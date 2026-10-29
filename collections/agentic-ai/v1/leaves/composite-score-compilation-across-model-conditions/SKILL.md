---
name: composite-score-compilation-across-model-conditions
description: Use when comparing multiple RAG or LLM system conditions on a fixed human-annotated
  query benchmark and you need a single composite score (integrating answer and citation
  quality) compiled per condition with variance and significance tests.
when_to_use_negative:
- The evaluation is not grounded in human annotations (e.g., purely automatic metrics
  with no annotator scores to aggregate)
- System conditions were not run over the same fixed query set, so per-condition scores
  are not comparable
edam_operation: http://edamontology.org/operation_2945
tools:
- name: AgriIR
  role: RAG framework executed in four model variants to produce system outputs scored
    on the benchmark
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: FAISS
  role: Local vector search backend used by the with-database AgriIR variants during
    the retrieval stage
  repo: https://github.com/facebookresearch/faiss
- name: ChatGPT-4o
  role: Top-ranked standalone baseline whose composite score (0.840±0.233) anchors
    the statistical comparison
- name: Gemini 2.5 Flash
  role: Standalone baseline significantly outperformed by AgriIR_Gemma-3-27B (0.779±0.250)
- name: GPT-OSS-120B
  role: Standalone baseline significantly outperformed by AgriIR_Gemma-3-27B (0.705±0.246,
    p<0.001)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:1db2176be053c793b2e356a0ef54cb03ba55c001cba53edc4793eee7ce9023ab
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# composite-score-compilation-across-model-conditions

## Summary

Aggregates human-annotated answer quality, citation quality, perfect-citation rate, and composite scores across the AgriIR variants and standalone baselines (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B) over 191 agricultural policy queries, producing a per-condition benchmark score table.

## When to use

Use when comparing multiple RAG or LLM system conditions on a fixed human-annotated query benchmark and you need a single composite score (integrating answer and citation quality) compiled per condition with variance and significance tests.

## When NOT to use

- The evaluation is not grounded in human annotations (e.g., purely automatic metrics with no annotator scores to aggregate)
- System conditions were not run over the same fixed query set, so per-condition scores are not comparable

## Inputs

- 191 agricultural policy queries (annotated benchmark)
- Human-annotated scores from 30 annotators (answer quality, citation quality, perfect-citation rate)
- System outputs from 4 AgriIR variants and 3 standalone baselines (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B)

## Outputs

- benchmark_scores.csv — per-condition table of answer quality, citation quality, perfect-citation rate, and composite score (mean ± std)
- Pairwise significance results (p-values) between AgriIR variants and baselines

## How to apply

Run all system conditions over the same 191-query annotated benchmark: the four AgriIR variants (Llama3.2:3B with/without database, Gemma3:1B without database, Gemma3:27B with database, executed through the 6-stage pipeline with query refinement T=0.1, sub-query decomposition T=0.5, synthesis T=0.2, and citation insertion at similarity ≥ 0.75) plus the three standalone baselines. Aggregate the 30-annotator human scores for answer quality, citation quality, perfect-citation rate, and composite score per condition, reporting mean ± standard deviation (e.g., AgriIR_Gemma-3-27B 0.820±0.208 vs ChatGPT-4o 0.840±0.233, p=0.493). Compute pairwise statistical tests between the best AgriIR variant and each baseline to distinguish equivalence (p=0.493 vs ChatGPT-4o) from significant differences (p<0.001 vs GPT-OSS-120B). Write the per-condition score table to benchmark_scores.csv for downstream comparison.

## Related tools

- **AgriIR** (RAG framework executed in four model variants to produce system outputs scored on the benchmark) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **FAISS** (Local vector search backend used by the with-database AgriIR variants during the retrieval stage) — https://github.com/facebookresearch/faiss
- **ChatGPT-4o** (Top-ranked standalone baseline whose composite score (0.840±0.233) anchors the statistical comparison)
- **Gemini 2.5 Flash** (Standalone baseline significantly outperformed by AgriIR_Gemma-3-27B (0.779±0.250))
- **GPT-OSS-120B** (Standalone baseline significantly outperformed by AgriIR_Gemma-3-27B (0.705±0.246, p<0.001))

## Examples

```
bash install_agriir.sh && ollama pull gemma3:1b && ollama pull llama3.2:3b
```

## Evaluation signals

- AgriIR_Gemma-3-27B composite reproduces to 0.820±0.208 and is statistically equivalent to ChatGPT-4o (0.840±0.233, p=0.493)
- AgriIR variants show 73–84% perfect citations, a capability absent in the baseline models
- All seven conditions (4 AgriIR variants + 3 baselines) are scored over the identical 191 queries from the same 30 annotators
- benchmark_scores.csv contains a row per system condition with answer quality, citation quality, perfect-citation rate, and composite score

## Limitations

- The paper's discussion section is absent from the source document, so no limitations, reproducibility gaps, or missing artifacts could be extracted from the authors
- Standalone baselines lack a citation-insertion capability, so citation-quality comparisons are asymmetric across conditions

## Evidence
