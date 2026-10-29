---
name: constrained-generation-vocabulary-preparation
description: Use when a constrained-generation pipeline needs a persistent, scalable
  vocabulary index over a very large KB (e.g., up to 800M verbalized Wikidata facts)
  too big for in-memory prefix trees.
when_to_use_negative:
- The knowledge base is small enough to fit in memory (~31k facts, as in try_refactx.ipynb)
  — use the in-memory prefix tree instead of the database-backed pipeline.
- The task requires free-form generation or LLM-only parametric answering rather than
  restricting generation to token sequences forming existing KB facts.
- The KB triples cannot be verbalized into tokenizable textual facts following the
  PrefixTree.md recipe.
tools:
- name: PostgreSQL
  role: 'Disk-backed persistence of the fact-tree: stores prefix-tree nodes, next-token
    sets, reachable-leaf counts, and the B-Tree index on the Prefix column'
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-3B
  role: Tokenizer whose vocabulary determines the token sequences indexed in the prefix
    tree for constrained generation
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:afb583e4f7b0547f5f4c935d99b5aded52ec6de42dd27a0e7155e763b7b1c9ac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - PostgreSQL
  - Qwen2.5-3B
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# constrained-generation-vocabulary-preparation

## Summary

Builds the disk-backed prefix-tree index that restricts an LLM's decoding vocabulary to token sequences forming existing knowledge-base facts, as done in ReFactX over ~800 million verbalized Wikidata facts. Persistence in PostgreSQL with prefix lookups makes constrained generation scale beyond what fits in RAM.

## When to use

Use when a constrained-generation pipeline needs a persistent, scalable vocabulary index over a very large KB (e.g., up to 800M verbalized Wikidata facts) too big for in-memory prefix trees.

## When NOT to use

- The knowledge base is small enough to fit in memory (~31k facts, as in try_refactx.ipynb) — use the in-memory prefix tree instead of the database-backed pipeline.
- The task requires free-form generation or LLM-only parametric answering rather than restricting generation to token sequences forming existing KB facts.
- The KB triples cannot be verbalized into tokenizable textual facts following the PrefixTree.md recipe.

## Inputs

- Wikidata truthy triples dump (~800 million facts)
- Verbalization recipe (PrefixTree.md in the rpo19/ReFactX repository)
- Tokenized verbalized-fact corpus (model tokenizer, e.g., Qwen2.5)
- PostgreSQL .env configuration file (from env-sample.txt)

## Outputs

- PostgreSQL fact-tree table storing prefix-tree nodes with next-token sets and reachable-leaf counts
- Pickle-packed subtree blobs beyond the cutoff length with single-row single-leaf compression
- B-Tree index on the Prefix column for efficient prefix-match lookups

## How to apply

Verbalize each truthy Wikidata triple into a natural-language textual fact following the repository's PrefixTree.md recipe, then tokenize the full verbalized fact corpus. Build the prefix tree in batches — hardware memory limits do not allow the entire 800M-fact tree to be kept in memory — persisting nodes into a PostgreSQL fact-tree table that stores next-token sets and reachable-leaf counts (the leaf counts prevent duplicate fact generation at decoding time). Pack subtrees with Pickle beyond the cutoff length and apply single-row single-leaf compression to reduce the number of traversed rows. Finally create a B-Tree index on the Prefix column so prefix-match lookups at each decoding step stay efficient. Validate by issuing next-token queries along sample fact prefixes and confirming only tokens forming existing facts are reachable, cross-checking against the 31k-facts in-memory prefix tree in try_refactx.ipynb.

## Related tools

- **PostgreSQL** (Disk-backed persistence of the fact-tree: stores prefix-tree nodes, next-token sets, reachable-leaf counts, and the B-Tree index on the Prefix column) — https://github.com/rpo19/ReFactX
- **Qwen2.5-3B** (Tokenizer whose vocabulary determines the token sequences indexed in the prefix tree for constrained generation) — https://github.com/rpo19/ReFactX

## Examples

```
pip install refactx[torch] && cp env-sample.txt .env && python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Next-token queries along sample fact prefixes return only tokens that form existing facts (no dead or invented continuations).
- Behavior of the disk-backed tree matches the 31k-facts in-memory prefix tree in try_refactx.ipynb on shared prefixes.
- Reachable-leaf counts are consistent with the stored facts, so constrained decoding never generates duplicate facts.
- Prefix-match lookups at each decoding step remain efficient, adding only ~1% latency: 303.89s vs 300.14s unconstrained for 4000 tokens (1.3% overhead).
- The ingestion pipeline completes in batches without exhausting hardware memory despite the 800M-fact corpus.

## Limitations

- Hardware memory limits prevent keeping the entire 800M-fact tree in memory, so ingestion must be performed in batches against PostgreSQL.
- Disk-backed access adds overhead relative to in-memory trees; the ~1% latency figure was measured with Qwen2.5-3B and PostgreSQL on the same machine, one Nvidia Tesla T4, so it is hardware-dependent.
- The in-memory quick-start path (try_refactx.ipynb) is limited to a 31k-facts knowledge base and requires skipping the .env setup.

## Evidence
