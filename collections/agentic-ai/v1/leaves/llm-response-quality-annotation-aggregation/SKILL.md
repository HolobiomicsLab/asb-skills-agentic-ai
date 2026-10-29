---
name: llm-response-quality-annotation-aggregation
description: Use when comparing question-answering or RAG systems on responses that
  need human-annotated answer and citation quality scores aggregated per system condition,
  with statistical equivalence testing (e.g. p=0.493) against baselines.
when_to_use_negative:
- Responses are ungrounded by design and citation quality cannot be assessed (the
  paper notes perfect-citation capability is absent in baseline models, so citation
  metrics only make sense when systems emit citations)
- The analysis goal is automatic metric computation (e.g. BLEU/ROUGE) rather than
  human annotation — this skill requires a human annotator panel
- The query set has no fixed, shared benchmark across systems, making per-condition
  aggregation incomparable
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: AgriIR
  role: Framework whose four variants' responses are annotated and scored against
    standalone baselines
  repo: https://github.com/Shuvam-Banerji-Seal/AgriIR
- name: ChatGPT-4o
  role: Standalone baseline whose composite score (0.840±0.233) anchors equivalence
    testing
- name: Gemini 2.5 Flash
  role: Standalone baseline scored on the same 191 queries
- name: GPT-OSS-120B
  role: Standalone baseline, significantly outperformed (p<0.001) by AgriIR_Gemma-3-27B
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-21324-2_37
    title: 'AgriIR: A Scalable Framework for Domain-Specific Knowledge Retrieval'
  source_content_hash: sha256:59a4f6ddb48ab2df10fbceeb037d16228be313f81573e75ec3ce9447d01e9ffc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Shuvam-Banerji-Seal/AgriIR
  tools:
  - AgriIR
  - ChatGPT-4o
  - GPT-OSS-120B
  - Gemini 2.5 Flash
  - Gemma3:1B
  - Gemma3:27B
  - Llama3.2:3B
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# LLM response quality annotation aggregation

## Summary

Human-annotation protocol for scoring LLM/QA-system responses on answer quality, citation quality, perfect-citation rate, and a composite performance score, then aggregating per system condition over an annotated benchmark (e.g. 191 agricultural policy queries rated by 30 annotators).

## When to use

Use when comparing question-answering or RAG systems on responses that need human-annotated answer and citation quality scores aggregated per system condition, with statistical equivalence testing (e.g. p=0.493) against baselines.

## When NOT to use

- Responses are ungrounded by design and citation quality cannot be assessed (the paper notes perfect-citation capability is absent in baseline models, so citation metrics only make sense when systems emit citations)
- The analysis goal is automatic metric computation (e.g. BLEU/ROUGE) rather than human annotation — this skill requires a human annotator panel
- The query set has no fixed, shared benchmark across systems, making per-condition aggregation incomparable

## Inputs

- Responses from each system condition over the 191-query agricultural policy benchmark
- Human annotator scores (30 annotators) for answer quality, citation quality, and perfect-citation rate per response
- System condition labels (e.g. AgriIR_Gemma-3-27B, ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B)

## Outputs

- Per-condition aggregate table of answer quality, citation quality, perfect-citation rate, and composite score with mean ± std (e.g. benchmark_scores.csv)
- Pairwise statistical comparison results (p-values, significance/equivalence decisions)

## How to apply

Run each candidate system (e.g. the four AgriIR variants using Llama3.2:3B, Gemma3:1B, and Gemma3:27B) and each standalone baseline (ChatGPT-4o, Gemini 2.5 Flash, GPT-OSS-120B) over the same fixed query set, here the 191 agricultural policy queries. Have a fixed panel of human annotators (30 in the paper) score each response for answer quality, citation quality, and whether citations are perfect. Aggregate per system condition to compute means and standard deviations plus the composite performance score that integrates answer and citation quality, e.g. AgriIR_Gemma-3-27B composite 0.820±0.208 vs ChatGPT-4o 0.840±0.233. Run statistical comparisons between system pairs to declare equivalence or significance (p=0.493 for equivalence; p<0.001 where AgriIR significantly outperforms GPT-OSS-120B at 0.705±0.246). Write the per-condition score table to an output file such as benchmark_scores.csv for reuse and diff-checking.

## Related tools

- **AgriIR** (Framework whose four variants' responses are annotated and scored against standalone baselines) — https://github.com/Shuvam-Banerji-Seal/AgriIR
- **ChatGPT-4o** (Standalone baseline whose composite score (0.840±0.233) anchors equivalence testing)
- **Gemini 2.5 Flash** (Standalone baseline scored on the same 191 queries)
- **GPT-OSS-120B** (Standalone baseline, significantly outperformed (p<0.001) by AgriIR_Gemma-3-27B)

## Examples

```
python scripts/aggregate_scores.py --queries 191 --annotators 30 --conditions AgriIR_Gemma-3-27B,AgriIR_Llama3.2-3B,AgriIR_Gemma3-1B,AgriIR_Llama3.2-3B-nodb,ChatGPT-4o,Gemini-2.5-Flash,GPT-OSS-120B --output benchmark_scores.csv
```

## Evaluation signals

- Composite score for the top AgriIR variant lands near 0.820±0.208 and is statistically equivalent to ChatGPT-4o (0.840±0.233, p=0.493)
- AgriIR conditions show 73-84% perfect citations, a capability absent in baseline models
- All system conditions are evaluated over the same 191 annotated queries with 30 human evaluators
- Per-condition table contains mean ± std for answer quality, citation quality, perfect-citation rate, and composite score for all seven system conditions
- Output table (benchmark_scores.csv) reproduces reported values on diff against published numbers

## Limitations

- Baseline models lack citation insertion, so citation-quality metrics structurally favor retrieval-augmented conditions
- Composite score depends on human annotation (30 annotators), introducing annotator-subjectivity not discussed in a limitations section (no discussion text present in source document)
- Results are specific to Indian agriculture and the 191-query benchmark; generalization to other domains is not demonstrated by this protocol alone

## Evidence

- [results] standalone baselines listed: [evidence span withheld — anchor status: not_found]
