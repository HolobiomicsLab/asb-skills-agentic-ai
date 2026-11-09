---
name: huggingface-transformers-logits-processor-implementation
description: Use when an LLM must generate outputs restricted to token sequences that
  exist in a knowledge base (e.g., verbalized Wikidata facts indexed in a prefix tree)
  and decoding must be constrained at each autoregressive step.
when_to_use_negative:
- The task does not require outputs restricted to existing KB facts — unconstrained
  generation adds no benefit and the constraint only narrows the vocabulary to fact-forming
  tokens
- No pre-built prefix-tree index exists for the target knowledge base; the processor
  has no allowed next-token set to look up
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: refactx[torch]
  role: Package providing the prefix-tree index and constrained-generation machinery
    installed via pip
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-3B
  role: Causal LM whose decoding is constrained by the LogitsProcessor in greedy and
    beam-search (3 beams) experiments
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:2a2b042921e02d6c5615474019d67be7d6d7b67906c399ca6d959631510f764b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - Qwen2.5-3B
  - refactx[torch]
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# huggingface-transformers-logits-processor-implementation

## Summary

Implements a Huggingface LogitsProcessor that enforces constrained fact-generation decoding by setting the log-probabilities of all vocabulary tokens outside the prefix-tree-allowed next-token set to −∞, so the LLM can only emit token sequences forming existing KB facts.

## When to use

Use when an LLM must generate outputs restricted to token sequences that exist in a knowledge base (e.g., verbalized Wikidata facts indexed in a prefix tree) and decoding must be constrained at each autoregressive step.

## When NOT to use

- The task does not require outputs restricted to existing KB facts — unconstrained generation adds no benefit and the constraint only narrows the vocabulary to fact-forming tokens
- No pre-built prefix-tree index exists for the target knowledge base; the processor has no allowed next-token set to look up

## Inputs

- Prefix-tree index over tokenized, verbalized KG triples (in-memory tree from a 31k-facts KB for the try_refactx notebook; disk-backed Wikidata tree holding up to 800M facts)
- Prompt containing the Fact command invoked via In-Context Learning
- Huggingface causal LM checkpoint (e.g., Qwen2.5-3B)

## Outputs

- Token sequences guaranteed to lie on valid prefix-tree paths, i.e., existing KB facts
- Per-step masked logit vector with finite probability only on tree-consistent tokens
- constrained_decoding_verification.json recording per-test results

## How to apply

Install refactx[torch] in a virtualenv or conda environment and load the in-memory prefix tree derived from the 31k-facts knowledge base shipped with the try_refactx notebook (the full Wikidata index verbalizes 800M facts). Implement a Huggingface LogitsProcessor that, at each decoding step, queries the prefix tree for the set of next tokens continuing a valid fact path and sets the log-probabilities of all other vocabulary tokens to -infinity; constrained generation is activated only once the LLM's Fact command (invoked via In-Context Learning) is recognized, and decoding reverts to normal afterwards. Run generation with a model such as Qwen2.5-3B under greedy and beam-search decoding (3 beams) on prompts that trigger the Fact command. Decide point: the final token choice always remains with the LLM's probability estimates among tree-allowed tokens — the processor narrows the vocabulary, it does not pick the token. The overhead is minimal (about 1.3%: 4000 tokens take 303.89s constrained vs 300.14s unconstrained).

## Related tools

- **refactx[torch]** (Package providing the prefix-tree index and constrained-generation machinery installed via pip) — https://github.com/rpo19/ReFactX
- **Qwen2.5-3B** (Causal LM whose decoding is constrained by the LogitsProcessor in greedy and beam-search (3 beams) experiments)

## Examples

```
pip install refactx[torch]
```

## Evaluation signals

- 100% of tokens emitted after Fact-command activation lie on valid prefix-tree paths
- The masked logit vector leaves finite probability only on tree-consistent tokens (all forbidden tokens are -infinity)
- Admissible-continuation check: after a prefix such as '<Danny Boyle> <' the allowed set includes 'date' but excludes 'born'
- Per-test results recorded in constrained_decoding_verification.json
- Constrained generation time overhead stays near the reported ~1.3% (303.89s vs 300.14s for 4000 tokens)

## Limitations

- The disk-backed prefix tree adds only ~1% latency but requires building the index over tokenized, verbalized triples (e.g., 800M Wikidata facts)
- Constrained generation narrows the vocabulary but leaves the final token choice to the LLM's probability estimates, so tree-valid facts can still be unhelpful for the question
- Performance is dataset-dependent: ReFactX performs worse than LLM-only on Mintaka and WebQSP, and on the Bank dataset reaches only up to 51.3% precision and 42.8% accuracy

## Evidence

- [methods] logits processor masks forbidden tokens: [evidence span withheld — anchor status: not_found]
