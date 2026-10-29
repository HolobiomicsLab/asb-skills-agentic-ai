---
name: prefix-tree-fact-retrieval
description: Use when an LLM must answer QA questions from a large KB (e.g., Wikidata)
  without external retrievers, by constraining decoding to token sequences that form
  existing KB facts.
when_to_use_negative:
- The questions are already well covered by the LLM's parametric knowledge — ReFactX
  performs worse than LLM-only on Mintaka and WebQSP (LLM-only reaches 82.0% and 80.5%
  accuracy respectively).
- 'You only need a quick trial: use the in-memory prefix tree from the 31k-facts knowledge
  base (try_refactx notebook) instead of the full disk-backed Wikidata index.'
- A pipeline with an external retriever or auxiliary model is required by the target
  system — ReFactX is explicitly designed as an alternative to retriever-based solutions.
edam_operation: http://edamontology.org/operation_3283
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: PostgreSQL
  role: Serves the disk-backed Wikidata prefix-tree index on the same machine with
    key-value caching enabled
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-7B
  role: LLM evaluated with constrained generation over the prefix tree (achieves 73.0%
    accuracy with the large Wikidata KB)
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-72B
  role: Largest evaluated LLM; reaches judge precision 85.2% and accuracy 78.9% with
    ReFactX
  repo: https://github.com/rpo19/ReFactX
- name: Llama3.3-70B
  role: Evaluated LLM reaching P=77.8% and C=70.6% (LLM-as-a-Judge) with ReFactX
  repo: https://github.com/rpo19/ReFactX
- name: Phi-4
  role: Evaluated LLM that maintains consistent precision across all answer types
  repo: https://github.com/rpo19/ReFactX
- name: Nvidia Tesla T4
  role: GPU used to run generation with key-value caching during the PostgreSQL-backed
    experiments
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-3B
  role: Model used for the throughput/latency experiment over 4000 tokens with the
    PostgreSQL index on a Tesla T4
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:4a42729635880fb1c2e674b4a84a693cd7d12f0d8e80ef91098623bb445a3b8b
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
  edam_operation: http://edamontology.org/operation_3283
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# prefix-tree-fact-retrieval

## Summary

ReFactX-style knowledge access for LLMs: verbalized Wikidata triples (up to 800M facts) are tokenized into a disk-backed prefix-tree index, and the LLM acquires external knowledge purely via constrained generation over valid fact paths — no retriever or auxiliary model.

## When to use

Use when an LLM must answer QA questions from a large KB (e.g., Wikidata) without external retrievers, by constraining decoding to token sequences that form existing KB facts.

## When NOT to use

- The questions are already well covered by the LLM's parametric knowledge — ReFactX performs worse than LLM-only on Mintaka and WebQSP (LLM-only reaches 82.0% and 80.5% accuracy respectively).
- You only need a quick trial: use the in-memory prefix tree from the 31k-facts knowledge base (try_refactx notebook) instead of the full disk-backed Wikidata index.
- A pipeline with an external retriever or auxiliary model is required by the target system — ReFactX is explicitly designed as an alternative to retriever-based solutions.

## Inputs

- Knowledge Graph triples (e.g., Wikidata) verbalized into textual facts
- Tokenized fact corpus indexed in a prefix tree (disk-backed, ~800M facts, PostgreSQL-served)
- QA datasets: 2WikiMH, Mintaka, WebQSP, Bank
- LLM checkpoint (e.g., Qwen2.5-7B/72B, Llama3.3-70B, Phi-4) via ICL prompt with the Fact command

## Outputs

- LLM-generated facts constrained to valid prefix-tree paths
- Answer predictions scored by Exact Match and LLM-as-a-Judge accuracy and precision per model/dataset pair
- Throughput/latency measurements (e.g., constrained vs unconstrained generation time over 4000 tokens)

## How to apply

Verbalize Knowledge Graph triples into textual facts, tokenize them, and index them in a prefix tree (a disk-backed tree stores up to 800M Wikidata facts and adds only ~1% latency). At inference, instruct the LLM via In-Context Learning to invoke the Fact command; once recognized, activate constrained generation so the model emits only tokens along valid prefix-tree paths, then revert to normal decoding. Serve the index through PostgreSQL on the same machine with key-value caching (e.g., on one Nvidia Tesla T4) and run evaluation with the eval.py script, swapping INDEX, MODEL, and DATASET (2WikiMH, Mintaka, WebQSP, Bank). Rationale: constrained generation narrows the vocabulary to tokens forming existing KB facts while leaving the final token choice to the LLM's probabilities, adding only a 1.3% time overhead (4000 tokens: 300.14s unconstrained vs 303.89s constrained).

## Related tools

- **PostgreSQL** (Serves the disk-backed Wikidata prefix-tree index on the same machine with key-value caching enabled) — https://github.com/rpo19/ReFactX
- **Qwen2.5-7B** (LLM evaluated with constrained generation over the prefix tree (achieves 73.0% accuracy with the large Wikidata KB)) — https://github.com/rpo19/ReFactX
- **Qwen2.5-72B** (Largest evaluated LLM; reaches judge precision 85.2% and accuracy 78.9% with ReFactX) — https://github.com/rpo19/ReFactX
- **Llama3.3-70B** (Evaluated LLM reaching P=77.8% and C=70.6% (LLM-as-a-Judge) with ReFactX) — https://github.com/rpo19/ReFactX
- **Phi-4** (Evaluated LLM that maintains consistent precision across all answer types) — https://github.com/rpo19/ReFactX
- **Nvidia Tesla T4** (GPU used to run generation with key-value caching during the PostgreSQL-backed experiments) — https://github.com/rpo19/ReFactX
- **Qwen2.5-3B** (Model used for the throughput/latency experiment over 4000 tokens with the PostgreSQL index on a Tesla T4) — https://github.com/rpo19/ReFactX

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Constrained generation overhead stays ~1.3%: generating 4000 tokens takes about 303.89s vs 300.14s unconstrained (Qwen2.5-3B, T4, PostgreSQL with key-value caching).
- On 2WikiMH, ReFactX accuracy and precision exceed LLM-only by more than 20% (e.g., Qwen2.5-72B: 92.5% judge accuracy and 96.4% precision vs 45.0%/45.2% LLM-only).
- ReFactX precision must always beat LLM-only under LLM-as-a-Judge evaluation; judge figures for Qwen2.5-72B should be P=85.2%/A=78.9 and Llama3.3-70B P=77.8%/C=70.6.
- Bank dataset results fall in the reported ranges: precision up to 51.3% and accuracy up to 42.8%.
- Every generated fact decodes to a token sequence that forms an existing fact in the prefix-tree index (i.e., only valid prefix-tree paths).

## Limitations

- ReFactX performs worse than LLM-only on Mintaka and WebQSP, where parametric knowledge already reaches 82.0% and 80.5% accuracy.
- The Bank dataset proved especially challenging, with ReFactX reaching only precision up to 51.3% and accuracy up to 42.8%.
- Constrained generation only narrows the vocabulary to tokens forming an existing KB fact; the final token choice always remains with the LLM, so it does not guarantee factual correctness.
- The full-scale setup requires a disk-backed index (~800M facts) served via PostgreSQL; a lightweight in-memory alternative covers only 31k facts.

## Evidence
