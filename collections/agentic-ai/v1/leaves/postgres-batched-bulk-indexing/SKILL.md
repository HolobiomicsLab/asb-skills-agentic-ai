---
name: postgres-batched-bulk-indexing
description: Use when a tokenized textual-fact corpus (hundreds of millions of verbalized
  triples) must be indexed into a persistent prefix tree so next-token queries during
  constrained generation avoid duplicate facts and stay efficient at scale.
when_to_use_negative:
- The knowledge base is small enough to fit in memory — use the in-memory prefix tree
  from the 31k-facts knowledge base in try_refactx.ipynb instead of the disk-backed
  PostgreSQL ingestion pipeline
- The goal is unconstrained generation or retrieval-augmented QA with a retriever
  — ReFactX is explicitly positioned as an alternative to pipeline-based solutions
  with external retrievers
- Input triples are not verbalizable into natural-language textual facts, since the
  index is over tokenized verbalized facts, not raw triples
tools:
- name: PostgreSQL
  role: Persistent fact-tree database storing prefix-tree nodes (next-token sets,
    reachable-leaf counts) with a B-Tree index on the Prefix column for prefix-match
    lookups
  repo: https://github.com/rpo19/ReFactX
- name: ReFactX
  role: Reference implementation providing the ingestion pipeline, PrefixTree.md recipe
    for building the Wikidata prefix tree, and the 31k-facts in-memory notebook for
    cross-validation
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:e3ad0cb533d44b1f348c7cb6a71c20efb0cb608f765876a9a0127fa24b28a112
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
yes_rate: 0.75
n_claims: 8
---

# postgres-batched-bulk-indexing

## Summary

Build and persist a disk-backed prefix-tree fact index in PostgreSQL so that constrained-generation over very large knowledge bases (up to 800M Wikidata textual facts) stays scalable: the tree is built in-memory batch by batch because hardware memory limits preclude holding it whole, with subtrees packed via Pickle and a B-Tree index on the Prefix column for fast prefix-match lookups.

## When to use

Use when a tokenized textual-fact corpus (hundreds of millions of verbalized triples) must be indexed into a persistent prefix tree so next-token queries during constrained generation avoid duplicate facts and stay efficient at scale.

## When NOT to use

- The knowledge base is small enough to fit in memory — use the in-memory prefix tree from the 31k-facts knowledge base in try_refactx.ipynb instead of the disk-backed PostgreSQL ingestion pipeline
- The goal is unconstrained generation or retrieval-augmented QA with a retriever — ReFactX is explicitly positioned as an alternative to pipeline-based solutions with external retrievers
- Input triples are not verbalizable into natural-language textual facts, since the index is over tokenized verbalized facts, not raw triples

## Inputs

- Wikidata truthy triples dump (Knowledge Graph triples)
- Verbalized natural-language textual facts derived from the triples (up to 800 million)
- Tokenized verbalized fact corpus (tokenizer-specific token sequences)

## Outputs

- PostgreSQL fact-tree table persisting prefix-tree nodes with next-token sets and reachable-leaf counts
- Pickle-packed subtrees beyond the cutoff length
- B-Tree index on the Prefix column of the fact-tree table
- Validated index supporting next-token queries that allow only tokens forming existing facts

## How to apply

Load the Wikidata truthy triples dump and verbalize each triple into a natural-language textual fact following the PrefixTree.md recipe, then tokenize the verbalized corpus of up to 800 million textual facts. Because hardware memory limits do not allow the entire tree to be kept in memory, build the prefix tree in batches and persist nodes into a PostgreSQL fact-tree table during ingestion, storing next-token sets and reachable-leaf counts to prevent duplicate fact generation. Beyond a cutoff length, pack subtrees with Pickle and apply single-row single-leaf compression to reduce the number of rows traversed. Finally, create a B-Tree index on the Prefix column so prefix-match lookups at each decoding step stay efficient. Validate by issuing next-token queries along sample fact prefixes, checking that only tokens forming existing facts are reachable, and cross-checking behavior against the 31k-facts in-memory prefix tree in try_refactx.ipynb.

## Related tools

- **PostgreSQL** (Persistent fact-tree database storing prefix-tree nodes (next-token sets, reachable-leaf counts) with a B-Tree index on the Prefix column for prefix-match lookups) — https://github.com/rpo19/ReFactX
- **ReFactX** (Reference implementation providing the ingestion pipeline, PrefixTree.md recipe for building the Wikidata prefix tree, and the 31k-facts in-memory notebook for cross-validation) — https://github.com/rpo19/ReFactX

## Evaluation signals

- Next-token queries along sample fact prefixes return only tokens that form existing facts in the knowledge base
- Constrained-generation behavior on the disk-backed PostgreSQL index matches that of the 31k-facts in-memory prefix tree in try_refactx.ipynb
- Index adds only ~1% latency to generation (measured: 4000 tokens take 303.89s constrained vs 300.14s unconstrained, a 1.3% overhead)
- No duplicate fact generation occurs, confirmed by reachable-leaf counts stored per node
- Prefix-match lookups at each decoding step remain efficient via the B-Tree index on the Prefix column

## Limitations

- The entire prefix tree cannot be kept in memory — hardware memory limits force batched ingestion and disk persistence
- On Mintaka and WebQSP, ReFactX performs worse than LLM-only models in accuracy (82.0% and 80.5% for LLM-only), so the index does not help when parametric knowledge already covers the questions
- The Bank dataset proved especially challenging, with ReFactX reaching only up to 51.3% precision and 42.8% accuracy
- Pickle-packed subtrees and single-row single-leaf compression trade row-granularity for reduced traversal, constraining per-node queries beyond the cutoff length

## Evidence

- [readme] Triples from Wikidata are verbalized in 800 million textual facts, tokenized, and indexed in a prefix tree for efficient access.: [evidence span withheld — anchor status: not_found]
- [other] Build the prefix tree in batches and persist nodes in a PostgreSQL fact-tree table, storing next-token sets and reachable-leaf counts to prevent duplicate fact generation.: [evidence span withheld — anchor status: not_found]
- [other] Pack subtrees with Pickle beyond the cutoff length and apply single-row single-leaf compression to reduce traversed rows.: [evidence span withheld — anchor status: not_found]
- [other] Create a B-Tree index on the Prefix column so prefix-match lookups at each decoding step stay efficient.: [evidence span withheld — anchor status: not_found]
