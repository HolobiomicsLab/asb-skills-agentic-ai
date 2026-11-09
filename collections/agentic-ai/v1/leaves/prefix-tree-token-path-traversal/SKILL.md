---
name: prefix-tree-token-path-traversal
description: Use when an LLM must be restricted to generating only token sequences
  that form existing facts from a knowledge base, e.g. QA over Wikidata-scale triples
  without a retriever.
when_to_use_negative:
- The query targets knowledge already well covered by the model's parametric knowledge
  — on Mintaka and WebQSP ReFactX performs worse than LLM-only, which reaches 82.0%
  and 80.5% accuracy respectively
- The task requires free-form generation rather than fact retrieval from the indexed
  KB
- No prefix-tree index exists for the target knowledge base and building one from
  the KB triples is not feasible
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: refactx[torch]
  role: Package providing the constrained-generation implementation and prefix-tree
    index used during decoding
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-3B
  role: LLM whose autoregressive decoding is constrained to valid fact paths during
    generation
- name: PostgreSQL
  role: Backend supporting the disk-backed prefix-tree index for large KBs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:38445c2a32c08c9fe7a6df0323d2f35dcbe8fb92c5e59dd58f76add8f6b975a3
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
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# prefix-tree-token-path-traversal

## Summary

Implements constrained decoding over a tokenized prefix-tree index of verbalized KB facts (e.g. up to 800M Wikidata-derived facts), where at each autoregressive step only tokens continuing a valid fact path receive finite probability, so the LLM can only emit existing KB facts. ReFactX uses this to replace external retrievers with generation-time knowledge access.

## When to use

Use when an LLM must be restricted to generating only token sequences that form existing facts from a knowledge base, e.g. QA over Wikidata-scale triples without a retriever.

## When NOT to use

- The query targets knowledge already well covered by the model's parametric knowledge — on Mintaka and WebQSP ReFactX performs worse than LLM-only, which reaches 82.0% and 80.5% accuracy respectively
- The task requires free-form generation rather than fact retrieval from the indexed KB
- No prefix-tree index exists for the target knowledge base and building one from the KB triples is not feasible

## Inputs

- Knowledge Graph triples verbalized as textual facts (e.g. from Wikidata), tokenized and indexed in a prefix tree
- In-memory prefix tree derived from the 31k-facts knowledge base in the try_refactx notebook
- Prompt triggering the Fact command via In-Context Learning
- LLM with Huggingface-compatible decoding loop (e.g. Qwen2.5-3B)

## Outputs

- Generated token sequences that are guaranteed to form existing facts in the KB
- Per-step masked logit vectors with -infinity on non-tree tokens
- Verification records (e.g. constrained_decoding_verification.)

## How to apply

Tokenize verbalized triples from the Knowledge Graph and index them in a prefix tree for efficient access (ReFactX supports a disk-backed tree holding up to 800M facts adding only ~1% latency, or the in-memory 31k-facts tree shipped with the try_refactx notebook). At inference, instruct the LLM via In-Context Learning to invoke the Fact command; once recognized, activate a Huggingface LogitsProcessor that looks up the set of next tokens continuing a valid fact path and sets the log-probabilities of all other vocabulary tokens to -infinity, leaving the final token choice to the LLM's probability estimates. Decode with Qwen2.5-3B under greedy or beam-search (3 beams) and revert to normal generation once the fact is complete. Verify by checking that every emitted token sequence lies on a valid prefix-tree path, e.g. after the prefix '<Danny Boyle> <' the admissible continuation set must include 'date' but exclude 'born'.

## Related tools

- **refactx[torch]** (Package providing the constrained-generation implementation and prefix-tree index used during decoding) — https://github.com/rpo19/ReFactX
- **Qwen2.5-3B** (LLM whose autoregressive decoding is constrained to valid fact paths during generation)
- **PostgreSQL** (Backend supporting the disk-backed prefix-tree index for large KBs)

## Examples

```
pip install refactx[torch]
```

## Evaluation signals

- 100% of tokens emitted after Fact-command activation lie on valid prefix-tree paths
- Masked logit vector leaves finite probability only on tree-consistent tokens (all others set to -infinity)
- Prefix test: after '<Danny Boyle> <' the admissible continuation set includes 'date' but excludes 'born'
- Constrained generation adds only ~1.3% time overhead (4000 tokens: 303.89s vs 300.14s unconstrained)
- Every generated fact string matches an existing fact in the KB

## Limitations

- Constrained generation only narrows the vocabulary to tokens forming an existing fact; the final token choice always remains with the LLM, so it does not guarantee fact relevance to the question
- Latency overhead, while small (~1.3% for 4000 tokens), depends on the index backend (disk-backed tree adds ~1% latency)
- Quality depends on KB coverage — the Bank dataset proved especially challenging, with precision up to 51.3% and accuracy up to 42.8%

## Evidence
