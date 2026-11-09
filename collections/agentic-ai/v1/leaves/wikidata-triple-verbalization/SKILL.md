---
name: wikidata-triple-verbalization
description: Use when Wikidata truthy RDF triples must be converted into natural-language
  textual facts and tokenized for building a prefix-tree fact index for constrained-generation
  KE-QA.
when_to_use_negative:
- The knowledge base is small (~31k facts) — use the in-memory prefix tree in try_refactx.ipynb
  instead of the disk-backed ingestion pipeline
- The input is not KG triples that can be verbalized with the PrefixTree.md recipe
  (e.g. free text or already-natural-language corpora)
- A retriever-based QA pipeline is acceptable and no constrained-generation fact index
  is required
tools:
- name: PostgreSQL
  role: Persists the fact-tree nodes in batches, storing next-token sets and reachable-leaf
    counts, with a B-Tree index on the Prefix column for efficient prefix-match lookups
  repo: https://github.com/rpo19/ReFactX
- name: ReFactX
  role: Provides the PrefixTree.md recipe for verbalizing Wikidata triples and the
    ingestion code for building the prefix-tree index
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:9cee6b18209e3d5b3abe1edcaf9d92d1cdd95f460d4085de2ec34e2cdb5650a6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - PostgreSQL
  - ReFactX
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# wikidata-triple-verbalization

## Summary

Verbalizes Wikidata truthy triples into natural-language textual facts (up to 800M) and tokenizes them so they can be indexed in a disk-backed PostgreSQL prefix tree for ReFactX constrained generation.

## When to use

Use when Wikidata truthy RDF triples must be converted into natural-language textual facts and tokenized for building a prefix-tree fact index for constrained-generation KE-QA.

## When NOT to use

- The knowledge base is small (~31k facts) — use the in-memory prefix tree in try_refactx.ipynb instead of the disk-backed ingestion pipeline
- The input is not KG triples that can be verbalized with the PrefixTree.md recipe (e.g. free text or already-natural-language corpora)
- A retriever-based QA pipeline is acceptable and no constrained-generation fact index is required

## Inputs

- Wikidata truthy triples dump (RDF triples from a Knowledge Graph)
- Verbalization recipe documented in PrefixTree.md
- Tokenizer configuration matching the target LLM (e.g. Qwen2.5 models)

## Outputs

- Corpus of up to 800 million verbalized natural-language textual facts
- Tokenized fact sequences feeding prefix-tree construction
- PostgreSQL fact-tree table with Prefix column, next-token sets, reachable-leaf counts, Pickle-packed subtrees, and a B-Tree index on Prefix

## How to apply

Load the Wikidata truthy triples dump and verbalize each triple into a natural-language textual fact following the PrefixTree.md recipe shipped in the rpo19/ReFactX repository. Tokenize the verbalized fact corpus (up to 800 million textual facts). Build the prefix tree in batches — hardware memory limits do not allow the entire tree to be kept in memory — persisting nodes in a PostgreSQL fact-tree table that stores next-token sets and reachable-leaf counts to prevent duplicate fact generation. Pack subtrees with Pickle beyond the cutoff length and apply single-row single-leaf compression to reduce traversed rows. Finally, create a B-Tree index on the Prefix column so prefix-match lookups at each decoding step stay efficient; the disk-backed index holds up to 800M facts while adding only ~1% generation latency.

## Related tools

- **PostgreSQL** (Persists the fact-tree nodes in batches, storing next-token sets and reachable-leaf counts, with a B-Tree index on the Prefix column for efficient prefix-match lookups) — https://github.com/rpo19/ReFactX
- **ReFactX** (Provides the PrefixTree.md recipe for verbalizing Wikidata triples and the ingestion code for building the prefix-tree index) — https://github.com/rpo19/ReFactX

## Examples

```
pip install refactx[torch]
```

## Evaluation signals

- Issue next-token queries along sample fact prefixes and verify only tokens forming existing facts are reachable
- Terminal nodes expose a single reachable leaf, confirming no duplicate fact generation
- Cross-check index behavior against the 31k-facts in-memory prefix tree in try_refactx.ipynb
- Prefix-match lookups at each decoding step stay efficient (overall generation latency overhead ~1%, 303.89s vs 300.14s for 4000 tokens)
- Ingestion completes in batches without exceeding hardware memory limits

## Limitations

- Hardware memory limits prevent keeping the entire 800M-fact tree in memory, so ingestion must be performed in batches
- Subtree packing with Pickle beyond a cutoff length and single-row single-leaf compression trade storage simplicity for reduced traversed rows
- Verbalized facts are only as good as the PrefixTree.md recipe; the skill does not address downstream answer accuracy on datasets like Mintaka or WebQSP where LLM-only baselines performed better

## Evidence

- [readme] 800M facts verbalized from Wikidata: [evidence span withheld — anchor status: not_found]
