---
name: fact-command-token-sequence-detection
description: Use when building or reproducing a QA/KE-QA loop where an LLM must fetch
  external knowledge on demand by emitting an inline command (e.g. 'Fact') that triggers
  constrained decoding over a tokenized fact index.
when_to_use_negative:
- The task is already covered by the model's parametric knowledge — on Mintaka and
  WebQSP, ReFactX performed worse than LLM-only generation (82.0% and 80.5% accuracy)
- You need a retriever-based pipeline rather than generation-time knowledge acquisition
  — ReFactX is explicitly positioned as an alternative to retriever/auxiliary-model
  solutions
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3489
tools:
- name: Qwen2.5-3B
  role: Instructed LLM whose output stream is scanned for the Fact-command token sequence;
    emits facts under constrained decoding
  repo: https://github.com/rpo19/ReFactX
- name: PostgreSQL
  role: Disk-backed prefix-tree index consulted during constrained generation; runs
    on the same machine with key-value caching enabled
  repo: https://github.com/rpo19/ReFactX
- name: refactx[torch]
  role: Package providing the prefix-tree index and constrained-generation decoding
    loop
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:40dd4afe7cf8f2e4847a6833d3a092f38a0770556bd616e48dad2aaebe631a86
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
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# fact-command-token-sequence-detection

## Summary

Implements ReFactX's inference-time control loop: scan the LLM's stream for the token sequence of the Fact command (invoked via In-Context Learning) and switch decoding between normal generation and constrained generation restricted to valid paths in a prefix-tree fact index.

## When to use

Use when building or reproducing a QA/KE-QA loop where an LLM must fetch external knowledge on demand by emitting an inline command (e.g. 'Fact') that triggers constrained decoding over a tokenized fact index.

## When NOT to use

- The task is already covered by the model's parametric knowledge — on Mintaka and WebQSP, ReFactX performed worse than LLM-only generation (82.0% and 80.5% accuracy)
- You need a retriever-based pipeline rather than generation-time knowledge acquisition — ReFactX is explicitly positioned as an alternative to retriever/auxiliary-model solutions

## Inputs

- Instructed LLM (e.g. Qwen2.5-3B) with a system prompt containing the Fact-command ICL instructions
- Tokenized fact index: in-memory prefix tree from a 31k-facts knowledge base, or disk-backed PostgreSQL prefix tree holding up to 800M verbalized Wikidata triples
- refactx[torch] package installed in a virtualenv/conda environment
- .env configuration file (copied from env-sample.txt) when using the PostgreSQL-backed index

## Outputs

- Generated answer grounded in KB facts emitted via constrained generation
- Explicit 'I don't know' response when no supporting fact is found
- Constrained segments that decode to existing KB facts in the prefix tree

## How to apply

Compose a system prompt that instructs the model via In-Context Learning to reason step by step and to invoke the Fact command when external facts are required. During generation, scan the output token stream for the exact token sequence corresponding to the Fact command; upon detection, activate constrained generation that permits only token sequences forming existing facts along valid paths in the pre-built prefix-tree index (in-memory for the 31k-facts KB, or disk-backed PostgreSQL for Wikidata-scale KBs). After each complete fact is emitted, revert decoding to normal (unconstrained) mode so multi-step reasoning can continue. Terminate on a final answer, an explicit 'I don't know' when no supporting fact is found, or the max-token budget (e.g. 4001 tokens in the throughput experiment). Constrained generation only narrows the vocabulary — the final token choice always remains with the LLM's probability estimates, and the overhead is small (~1.3%, 303.89s vs 300.14s for 4000 tokens).

## Related tools

- **Qwen2.5-3B** (Instructed LLM whose output stream is scanned for the Fact-command token sequence; emits facts under constrained decoding) — https://github.com/rpo19/ReFactX
- **PostgreSQL** (Disk-backed prefix-tree index consulted during constrained generation; runs on the same machine with key-value caching enabled) — https://github.com/rpo19/ReFactX
- **refactx[torch]** (Package providing the prefix-tree index and constrained-generation decoding loop) — https://github.com/rpo19/ReFactX

## Examples

```
python throughput.py --model MODEL --index INDEX --max-tokens 4001 --output out.json
```

## Evaluation signals

- Every constrained segment must decode to an existing KB fact along a valid path in the prefix tree
- The demo run terminates with either a fact-grounded answer or an explicit 'I don't know' response
- Constrained-generation time overhead stays near the reported ~1.3% (303.89s vs 300.14s for 4000 tokens on a Tesla T4 with KV caching)
- Precision should exceed LLM-only baselines in LLM-as-a-Judge evaluation on external-knowledge-dependent datasets (e.g. >20% improvement on 2WikiMH)
- Fact-command detection must trigger constrained decoding at the correct boundary and revert to normal decoding after each complete fact

## Limitations

- On datasets largely covered by parametric knowledge (Mintaka, WebQSP), accuracy drops below LLM-only generation (82.0% and 80.5% respectively)
- Constrained generation narrows the vocabulary but does not guarantee correctness — final token choice remains with the LLM's probabilities
- The Bank dataset proved especially challenging, with precision up to only 51.3% and accuracy up to 42.8%

## Evidence
