---
name: constrained-decoding-logits-masking
description: Use when an LLM must ground its outputs in an external knowledge base
  during inference — e.g., a QA pipeline where the model invokes a 'Fact' command
  and every emitted token must lie on a valid fact path in a tokenized prefix-tree
  index.
when_to_use_negative:
- Questions already fully covered by the LLM's parametric knowledge — the article
  reports ReFactX performs worse than LLM-only in accuracy on Mintaka (LLM-only 82.0%)
  and WebQSP (80.5%)
- When you need free-form generation rather than token sequences that must form an
  existing fact — the processor forbids any token outside valid fact paths
- When no prefix-tree index of verbalized, tokenized facts is available and building
  one is out of scope — the mechanism requires the tree to compute the admissible
  next-token set
edam_operation: http://edamontology.org/operation_2425
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3372
tools:
- name: refactx[torch]
  role: Python package providing the constrained-generation mechanism and prefix-tree
    index used by the LogitsProcessor
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-3B
  role: LLM run under greedy and beam-search (3 beams) decoding with the logits mask
    active during Fact-command generation
- name: PostgreSQL
  role: Disk-backed store for the large Wikidata prefix-tree index; adds only ~1%
    latency
- name: try_refactx.ipynb
  role: Notebook demonstrating the mechanism with an in-memory prefix tree from a
    31k-facts knowledge base
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:b57b21204f875d0a464825d8e167d0a5179a5e5eef49b2852cfad8b3f7f083ab
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - PostgreSQL
  - Qwen2.5-3B
  - refactx[torch]
  - try_refactx.ipynb
  edam_operation: http://edamontology.org/operation_2425
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# constrained-decoding-logits-masking

## Summary

A Huggingface LogitsProcessor-based decoding constraint (from ReFactX) that sets log-probabilities of all tokens not continuing a valid prefix-tree fact path to -infinity, so an LLM's autoregressive generation can only emit token sequences that form existing KB facts while the model itself still picks among allowed tokens.

## When to use

Use when an LLM must ground its outputs in an external knowledge base during inference — e.g., a QA pipeline where the model invokes a 'Fact' command and every emitted token must lie on a valid fact path in a tokenized prefix-tree index.

## When NOT to use

- Questions already fully covered by the LLM's parametric knowledge — the article reports ReFactX performs worse than LLM-only in accuracy on Mintaka (LLM-only 82.0%) and WebQSP (80.5%)
- When you need free-form generation rather than token sequences that must form an existing fact — the processor forbids any token outside valid fact paths
- When no prefix-tree index of verbalized, tokenized facts is available and building one is out of scope — the mechanism requires the tree to compute the admissible next-token set

## Inputs

- Tokenized knowledge-base facts indexed in an in-memory prefix tree (e.g., 31k-facts demo KB) or disk-backed prefix tree (up to 800M Wikidata facts, e.g., via PostgreSQL)
- A Huggingface-compatible causal LLM (e.g., Qwen2.5-3B, Qwen2.5-7B, Qwen2.5-70B)
- Inference prompts where the model is instructed via In-Context Learning to invoke the Fact command
- refactx[torch] package installed in a virtualenv or conda environment

## Outputs

- Generated token sequences guaranteed to form existing KB facts
- A finite-probability logits vector restricted to prefix-tree-consistent tokens at each decoding step
- Per-test verification records (e.g., constrained_decoding_verification.json with acceptance that 100% of post-Fact-command tokens lie on valid prefix-tree paths)

## How to apply

Verbalize Knowledge-Graph triples into textual facts, tokenize them, and index them in a prefix tree (e.g., from the 31k-facts KB shipped with try_refactx.ipynb, or the 800M-fact Wikidata tree). Implement a transformers LogitsProcessor that, at each decoding step, looks up the set of next tokens continuing a valid fact path in the prefix tree and sets the log-probability of every other vocabulary token to -infinity; the mask leaves finite probability only on tree-consistent tokens, but the final token choice remains with the LLM's probability estimates. Activate the processor at inference when In-Context Learning has instructed the model to invoke the Fact command, run generation (greedy or beam search, e.g., 3 beams with Qwen2.5-3B), then revert to unconstrained decoding once the fact is complete. Constrained generation adds only ~1.3% time overhead (4000 tokens: 303.89s vs 300.14s unconstrained), so throughput loss is not a blocker. Validate that every generated token sequence forms an existing fact — e.g., after the prefix '<Danny Boyle> <' the admissible continuation set includes 'date' but excludes 'born'.

## Related tools

- **refactx[torch]** (Python package providing the constrained-generation mechanism and prefix-tree index used by the LogitsProcessor) — https://github.com/rpo19/ReFactX
- **Qwen2.5-3B** (LLM run under greedy and beam-search (3 beams) decoding with the logits mask active during Fact-command generation)
- **PostgreSQL** (Disk-backed store for the large Wikidata prefix-tree index; adds only ~1% latency)
- **try_refactx.ipynb** (Notebook demonstrating the mechanism with an in-memory prefix tree from a 31k-facts knowledge base) — https://github.com/rpo19/ReFactX

## Examples

```
pip install refactx[torch]  # then: python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- 100% of tokens emitted after Fact-command activation lie on valid prefix-tree paths
- The masked logit vector leaves finite probability only on tree-consistent tokens (all others set to -infinity)
- Admissible-continuation spot check: after prefix '<Danny Boyle> <' the allowed set includes 'date' but excludes 'born'
- Throughput overhead stays near the reported ~1.3% (4000 tokens: 303.89s constrained vs 300.14s unconstrained)
- Every complete generated sequence corresponds to an existing fact in the KB tree

## Limitations

- Constrained generation only narrows the vocabulary to tokens forming an existing fact; the final token choice always remains with the LLM's probability estimates, so wrong facts can still be selected among valid ones
- Accuracy can drop below LLM-only baselines on datasets well covered by parametric knowledge (Mintaka, WebQSP)
- On challenging datasets precision may be low (Bank dataset: precision up to 51.3%, accuracy up to 42.8%)
- The disk-backed large index requires setup (PostgreSQL, .env configuration from env-sample.txt); the simple 31k-facts in-memory index is only a demo

## Evidence

- [results] logits processor masks forbidden tokens: [evidence span withheld — anchor status: not_found]
- [intro] final choice remains with the LLM: "Constrained generation only narrows the vocabulary to the tokens that form an existing fact in the KB, but always leaves the final choice to the LLM."
- [abstract] facts indexed in prefix tree: "Triples from a Knowledge Graph are verbalized in textual facts, tokenized, and indexed in a prefix tree for efficient access."
- [readme] installation command: "Create a virtualenv or conda environment, then pip install refactx[torch]"
- [methods] demo notebook: "For quickly trying ReFactX with an in-memory prefix tree (derived from a 31k-facts knowledge base) use the notebook"
- [results] time overhead: "the total time for generating 4000 tokens increases by only 1.3%, from 300.14 seconds to 303.89."
