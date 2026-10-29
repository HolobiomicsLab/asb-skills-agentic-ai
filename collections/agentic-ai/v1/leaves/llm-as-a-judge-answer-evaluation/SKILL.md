---
name: llm-as-a-judge-answer-evaluation
description: Use when evaluating natural-language QA outputs whose surface forms vary
  (dates, names, numbers) and Exact Match alone would under- or over-count correctness,
  e.g. benchmarking constrained-generation vs. parametric-LLM answering.
when_to_use_negative:
- Answers are canonical identifiers or fixed strings where Exact Match alone is a
  sufficient, unambiguous metric
- You need reproducible, judge-free scoring — LLM-as-a-Judge introduces judge-model
  variance into the metric
- Evaluating judge-relevant reasoning traces rather than final answer strings; the
  article scores answers, not chains
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Qwen2.5-72B
  role: QA model whose answers are judge-scored; source of the headline judge metrics
    (P=85.2%, A=78.9; 92.5% judge accuracy on 2WikiMH)
  repo: https://github.com/rpo19/ReFactX
- name: Llama3.3-70B
  role: QA model baseline scored by the judge (P=77.8%, C=70.6)
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-7B
  role: Smaller QA model scored by the judge (73.0% accuracy reported)
  repo: https://github.com/rpo19/ReFactX
- name: Phi-4
  role: QA model evaluated; maintains consistent judge precision across all answer
    types
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:408673d0ec207d20e185d563a29dc74a969ba19f40c292a8285edc5c6b659745
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - Llama3.3-70B
  - Nvidia Tesla T4
  - Phi-4
  - PostgreSQL
  - Qwen2.5-3B
  - Qwen2.5-72B
  - Qwen2.5-7B
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# LLM-as-a-Judge answer evaluation

## Summary

Scores free-text QA answers by having an LLM judge decide whether a generated answer is correct, complementing Exact Match, and reports accuracy (A) and precision (P) per model/dataset pair. It was used to quantify ReFactX's gains over LLM-only parametric answering.

## When to use

Use when evaluating natural-language QA outputs whose surface forms vary (dates, names, numbers) and Exact Match alone would under- or over-count correctness, e.g. benchmarking constrained-generation vs. parametric-LLM answering.

## When NOT to use

- Answers are canonical identifiers or fixed strings where Exact Match alone is a sufficient, unambiguous metric
- You need reproducible, judge-free scoring — LLM-as-a-Judge introduces judge-model variance into the metric
- Evaluating judge-relevant reasoning traces rather than final answer strings; the article scores answers, not chains

## Inputs

- QA predictions per model/dataset pair (2WikiMH, Mintaka, WebQSP, Bank) from eval.py runs
- Dataset reference answers
- Optional: paired Exact Match scores for cross-checking the judge

## Outputs

- Per model/dataset accuracy (A) and precision (P) under LLM-as-a-Judge scoring
- Comparison table of ReFactX vs. LLM-only baselines (e.g. judge P=85.2%/A=78.9 for Qwen2.5-72B)

## How to apply

Run the QA system (here ReFactX with constrained generation over the Wikidata prefix-tree index, or the LLM-only unconstrained baseline) on each dataset (2WikiMH, Mintaka, WebQSP, Bank) using eval.py with a chosen model (e.g. Qwen2.5-72B, Llama3.3-70B). Score every answer twice: with Exact Match and with an LLM-as-a-Judge that reads the question, reference, and candidate and decides correctness. Aggregate per model/dataset into two judge metrics — accuracy (A) and precision (P) — e.g. Qwen2.5-72B reaches P=85.2% and A=78.9 with the judge, vs. P=77.8% and C=70.6 for Llama3.3-70B. Compare ReFactX vs. LLM-only pairs: ReFactX consistently wins on judge precision, and on 2WikiMH improves both accuracy and precision by more than 20% relative (Qwen2.5-72B: 92.5% judge accuracy, 96.4% precision vs. 45.0%/45.2% LLM-only).

## Related tools

- **Qwen2.5-72B** (QA model whose answers are judge-scored; source of the headline judge metrics (P=85.2%, A=78.9; 92.5% judge accuracy on 2WikiMH)) — https://github.com/rpo19/ReFactX
- **Llama3.3-70B** (QA model baseline scored by the judge (P=77.8%, C=70.6)) — https://github.com/rpo19/ReFactX
- **Qwen2.5-7B** (Smaller QA model scored by the judge (73.0% accuracy reported)) — https://github.com/rpo19/ReFactX
- **Phi-4** (QA model evaluated; maintains consistent judge precision across all answer types) — https://github.com/rpo19/ReFactX

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Judge metrics fall in the reported ranges: Qwen2.5-72B judge P=85.2% and A=78.9; Llama3.3-70B P=77.8% and C=70.6
- ReFactX judged precision exceeds LLM-only precision for every model/dataset pair (the paper's invariant: 'Re FactX always outperforms LLM-only models in precision in the LLM-as-a-Judge evaluation')
- On 2WikiMH the ReFactX-vs-LLM-only gap exceeds 20% in both accuracy and precision (e.g. 92.5%/96.4% vs. 45.0%/45.2%)
- Exact Match and judge scores are computed for the same runs and reported side by side; large judge-vs-EM divergence flags verbose but correct answers
- Bank dataset results stay within reported bounds (precision up to 51.3%, accuracy up to 42.8%)

## Limitations

- ReFactX performs worse than LLM-only in accuracy on Mintaka and WebQSP, where LLM-only parametric knowledge already reaches 82.0% and 80.5% accuracy
- The Bank dataset proved especially challenging, capping judge precision at 51.3% and accuracy at 42.8%
- Judge scores depend on the judge LLM's own reliability; the paper reports LLM-as-a-Judge alongside Exact Match rather than replacing it

## Evidence
