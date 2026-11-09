---
name: knowledge-base-fact-grounded-answer-evaluation
description: Use when a QA pipeline generates answers from KB facts via constrained
  generation and you must quantify precision/accuracy against LLM-only baselines and
  validate fact-groundedness of emitted segments.
when_to_use_negative:
- The questions are fully covered by the LLM's parametric knowledge (as on Mintaka
  and WebQSP, where LLM-only reached 82.0% and 80.5% accuracy and ReFactX underperformed
  on accuracy)
- You need a single-metric comparison without a judge model — precision here requires
  LLM-as-a-Judge, not string-match scoring
- The KB is not verbalized and tokenized into a prefix tree, so constrained generation
  cannot guarantee fact-grounded segments
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3345
tools:
- name: Qwen2.5-3B
  role: Generation model whose constrained fact-generation loop is evaluated on the
    QA benchmarks
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-72B
  role: LLM-as-a-Judge scoring answers for precision and accuracy
  repo: https://github.com/rpo19/ReFactX
- name: Llama3.3-70B
  role: Alternative judge and generation model (P=77.8%, A=70.6%)
  repo: https://github.com/rpo19/ReFactX
- name: PostgreSQL
  role: Disk-backed prefix-tree index serving facts during constrained generation
    during evaluation runs
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:4dd284ee717b5696456e7766ab9570362882c2e0f4202f13a4be937b883ff2b1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - Llama3.3-70B
  - PostgreSQL
  - Qwen2.5-3B
  - Qwen2.5-72B
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# knowledge-base-fact-grounded-answer-evaluation

## Summary

Evaluates QA systems that ground answers in knowledge-base facts (e.g., ReFactX with constrained generation over a Wikidata prefix-tree index) using LLM-as-a-Judge precision plus exact accuracy on multi-dataset benchmarks, verifying that constrained segments decode to existing KB facts.

## When to use

Use when a QA pipeline generates answers from KB facts via constrained generation and you must quantify precision/accuracy against LLM-only baselines and validate fact-groundedness of emitted segments.

## When NOT to use

- The questions are fully covered by the LLM's parametric knowledge (as on Mintaka and WebQSP, where LLM-only reached 82.0% and 80.5% accuracy and ReFactX underperformed on accuracy)
- You need a single-metric comparison without a judge model — precision here requires LLM-as-a-Judge, not string-match scoring
- The KB is not verbalized and tokenized into a prefix tree, so constrained generation cannot guarantee fact-grounded segments

## Inputs

- Benchmark QA datasets: Mintaka, WebQSP, 2WikiMH, Bank
- Pre-built prefix-tree fact index (in-memory 31k-facts KB or disk-backed Wikidata index of up to 800M verbalized facts)
- Instruction-tuned LLM (Qwen2.5-3B/7B/70B, Phi-4, Llama3.3-70B)
- LLM judge model (Qwen2.5-72B or Llama3.3-70B)

## Outputs

- Per-dataset precision and accuracy scores (LLM-as-a-Judge)
- Per-question verdicts including 'I don't know' abstentions and token-budget terminations
- Constrained segments verified to decode to existing prefix-tree facts

## How to apply

Run the QA control loop end-to-end (e.g., with Qwen2.5-3B or Qwen2.5-7B over the pre-built prefix-tree index) and collect answers on the benchmark datasets (Mintaka, WebQSP, 2WikiMH, Bank). Score answers with an LLM-as-a-Judge configuration (e.g., Qwen2.5-72B or Llama3.3-70B as judge), reporting precision and accuracy per dataset — ReFactX reaches 92.5% accuracy with Qwen2.5-70B, 85.2% precision with Qwen2.5-72B judge, and improves 2WikiMH accuracy/precision by more than 20% over LLM-only models. Verify the invariant that every constrained segment decodes to an existing fact in the prefix tree, and that runs terminate with a fact-grounded answer, an explicit "I don't know", or the maximum-token budget (e.g., 4000 tokens). Compare precision against LLM-only parametric answering: ReFactX should always win on precision, even where accuracy drops (Mintaka, WebQSP). Reproduce with eval.py by swapping INDEX, MODEL, and DATASET imports.

## Related tools

- **Qwen2.5-3B** (Generation model whose constrained fact-generation loop is evaluated on the QA benchmarks) — https://github.com/rpo19/ReFactX
- **Qwen2.5-72B** (LLM-as-a-Judge scoring answers for precision and accuracy) — https://github.com/rpo19/ReFactX
- **Llama3.3-70B** (Alternative judge and generation model (P=77.8%, A=70.6%)) — https://github.com/rpo19/ReFactX
- **PostgreSQL** (Disk-backed prefix-tree index serving facts during constrained generation during evaluation runs) — https://github.com/rpo19/ReFactX

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Every constrained segment decodes to an existing KB fact in the prefix tree
- Each run terminates with a fact-grounded answer, an explicit "I don't know", or the maximum-token budget (4000 tokens)
- ReFactX precision exceeds LLM-only precision on every dataset under LLM-as-a-Judge scoring
- Accuracy on 2WikiMH improves by more than 20% relative to LLM-only parametric answering
- Constrained-generation throughput overhead stays near the reported 1.3% (303.89s vs 300.14s for 4000 tokens)

## Limitations

- ReFactX performs worse than LLM-only on Mintaka and WebQSP in accuracy because those datasets are largely covered by parametric knowledge
- The Bank dataset is especially challenging: precision up to 51.3% and accuracy up to 42.8%
- Constrained generation narrows the vocabulary but always leaves the final token choice to the LLM's probability estimates

## Evidence
