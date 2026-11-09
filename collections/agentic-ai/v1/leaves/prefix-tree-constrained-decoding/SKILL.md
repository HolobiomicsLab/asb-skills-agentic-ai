---
name: prefix-tree-constrained-decoding
description: Use when an LLM must acquire external knowledge during generation without
  a retriever or auxiliary model, by decoding facts directly from a tokenized prefix-tree
  KB index via constrained generation.
when_to_use_negative:
- Questions are already well covered by the model's parametric knowledge — ReFactX
  performed worse than LLM-only on Mintaka and WebQSP, where LLM-only reached 82.0%
  and 80.5% accuracy
- A retriever-based or pipeline-based KE-QA solution is preferred or already available;
  ReFactX is positioned as an alternative that avoids retrievers and auxiliary models
- The target KB facts have not been verbalized, tokenized, and indexed in a prefix
  tree (the .env setup and index build are prerequisites)
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0218
- http://edamontology.org/topic_3474
tools:
- name: Qwen2.5-3B
  role: LLM whose generation loop detects the Fact-command token sequence and switches
    between normal and constrained decoding
  repo: https://github.com/rpo19/ReFactX
- name: PostgreSQL
  role: Disk-backed prefix-tree index holding the tokenized facts, running on the
    same machine with key-value caching enabled
  repo: https://github.com/rpo19/ReFactX
- name: refactx[torch]
  role: Python package that loads the tokenized fact index and runs the constrained-generation
    loop
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:61e51486bea41b1c77e1bd0ca5147dad0224cbb1873d04beae3f3eb8e57f3aba
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - PostgreSQL
  - Qwen2.5-3B
  - refactx[torch]
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0218
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# prefix-tree-constrained-decoding

## Summary

Implements ReFactX's constrained-decoding control loop: verbalized KG triples are tokenized into a prefix-tree fact index, and at inference the LLM's emitted Fact-command token sequence triggers constrained generation restricted to valid prefix-tree paths, reverting to normal decoding after each fact.

## When to use

Use when an LLM must acquire external knowledge during generation without a retriever or auxiliary model, by decoding facts directly from a tokenized prefix-tree KB index via constrained generation.

## When NOT to use

- Questions are already well covered by the model's parametric knowledge — ReFactX performed worse than LLM-only on Mintaka and WebQSP, where LLM-only reached 82.0% and 80.5% accuracy
- A retriever-based or pipeline-based KE-QA solution is preferred or already available; ReFactX is positioned as an alternative that avoids retrievers and auxiliary models
- The target KB facts have not been verbalized, tokenized, and indexed in a prefix tree (the .env setup and index build are prerequisites)

## Inputs

- Verbalized KG triples tokenized and indexed in a prefix tree (in-memory 31k-facts index or disk-backed PostgreSQL index of up to 800M facts)
- ICL system prompt instructing the LLM to invoke the Fact command
- QA dataset questions (e.g. Mintaka, WebQSP, 2WikiMH, Bank)

## Outputs

- Fact-grounded answers produced by decoding complete facts along valid prefix-tree paths interleaved with free reasoning
- Explicit 'I don't know' response when no supporting fact is found

## How to apply

Tokenize and index verbalized facts (e.g. Wikidata triples verbalized into 800M textual facts, or the 31k-facts demo KB) in a prefix tree, either in-memory via the refactx[torch] package or disk-backed in PostgreSQL. Compose an ICL system prompt instructing the LLM (e.g. Qwen2.5-3B) to reason step by step and invoke the Fact command when external knowledge is needed. While generating, scan output for the Fact-command token sequence; on detection, switch to constrained generation allowing only tokens along valid prefix-tree paths, and revert to normal decoding after each complete fact so multi-step reasoning continues. Terminate on a final answer, an 'I don't know' response, or the max-token budget (experiments use 4000 tokens). Rationale: constrained generation only narrows the vocabulary to tokens forming an existing KB fact, always leaving the final token choice to the LLM's probability estimates, and the disk-backed index adds only ~1% latency.

## Related tools

- **Qwen2.5-3B** (LLM whose generation loop detects the Fact-command token sequence and switches between normal and constrained decoding) — https://github.com/rpo19/ReFactX
- **PostgreSQL** (Disk-backed prefix-tree index holding the tokenized facts, running on the same machine with key-value caching enabled) — https://github.com/rpo19/ReFactX
- **refactx[torch]** (Python package that loads the tokenized fact index and runs the constrained-generation loop) — https://github.com/rpo19/ReFactX

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Every constrained segment must decode to an existing KB fact in the prefix tree
- Demo run must terminate with either a fact-grounded answer or an explicit 'I don't know' response
- Constrained generation time overhead over 4000 tokens should be ~1.3% (303.89s vs 300.14s unconstrained) on a single Nvidia Tesla T4
- Disk-backed prefix-tree index should add only ~1% latency versus in-memory access
- LLM-as-a-Judge evaluation should show ReFactX precision consistently above LLM-only baselines (e.g. 85.2% with Qwen2.5-72B judging)

## Limitations

- Accuracy drops below LLM-only baselines on datasets well covered by parametric knowledge (Mintaka, WebQSP: 82.0% and 80.5% for LLM-only)
- Hard KBs remain challenging: on the Bank dataset precision peaks at 51.3% and accuracy at 42.8%
- Constrained generation narrows the vocabulary but leaves token choice to the LLM's probability estimates, so emitted facts are guaranteed to exist but not guaranteed to be relevant

## Evidence
