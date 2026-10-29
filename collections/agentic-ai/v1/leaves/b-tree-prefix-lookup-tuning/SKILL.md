---
name: b-tree-prefix-lookup-tuning
description: Use when a constrained-generation LLM pipeline needs a scalable disk-backed
  fact index over a very large KB (e.g., Wikidata), and in-memory prefix trees are
  infeasible due to hardware memory limits.
when_to_use_negative:
- The knowledge base is small enough to fit an in-memory prefix tree — the article
  uses a 31k-facts in-memory tree via try_refactx.ipynb instead of the disk-backed
  PostgreSQL index.
- The workflow uses a retriever or auxiliary retrieval model rather than constrained
  generation over a pre-built fact index — ReFactX is explicitly positioned as an
  alternative to pipeline-based solutions.
- The task is inference-time constrained generation over an already-built index rather
  than index construction and lookup tuning.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PostgreSQL
  role: Persist the batch-built prefix-tree nodes (fact-tree table with next-token
    sets and reachable-leaf counts) and host the B-Tree index on the Prefix column
    for efficient prefix-match lookups
  repo: https://github.com/rpo19/ReFactX
- name: Pickle
  role: Pack subtrees beyond the cutoff length as compressed blobs to reduce rows
    traversed during traversal
  repo: https://github.com/rpo19/ReFactX
- name: ReFactX
  role: Provides the ingestion pipeline, PrefixTree.md recipe, env-sample.txt configuration,
    and try_refactx.ipynb in-memory reference tree for validation
  repo: https://github.com/rpo19/ReFactX
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:9729240eb9e67746547b2b386da6c257a0b6bafe2fd9983e267d062908789f64
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - Pickle
  - PostgreSQL
  - ReFactX
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# B-Tree prefix-lookup tuning for disk-backed prefix-tree fact indexes

## Summary

Constructs and persists a PostgreSQL-backed prefix-tree index over verbalized, tokenized knowledge-base facts (ReFactX: up to 800M Wikidata facts), using batched ingestion, Pickle subtree packing, single-leaf compression, and a B-Tree index on the Prefix column so per-decoding-step prefix-match lookups add only ~1% latency.

## When to use

Use when a constrained-generation LLM pipeline needs a scalable disk-backed fact index over a very large KB (e.g., Wikidata), and in-memory prefix trees are infeasible due to hardware memory limits.

## When NOT to use

- The knowledge base is small enough to fit an in-memory prefix tree — the article uses a 31k-facts in-memory tree via try_refactx.ipynb instead of the disk-backed PostgreSQL index.
- The workflow uses a retriever or auxiliary retrieval model rather than constrained generation over a pre-built fact index — ReFactX is explicitly positioned as an alternative to pipeline-based solutions.
- The task is inference-time constrained generation over an already-built index rather than index construction and lookup tuning.

## Inputs

- Wikidata truthy triples dump (knowledge-graph triples)
- Verbalized and tokenized textual fact corpus (up to 800 million facts)
- PostgreSQL instance reachable via .env connection settings (from env-sample.txt)

## Outputs

- PostgreSQL fact-tree table of prefix-tree nodes with next-token sets and reachable-leaf counts
- Pickle-packed subtree blobs for nodes beyond the cutoff length
- B-Tree index on the Prefix column supporting per-decoding-step prefix-match lookups

## How to apply

Load the knowledge-graph truthy triples dump (Wikidata) and verbalize each triple into a natural-language textual fact following the PrefixTree.md recipe, then tokenize the corpus of up to 800 million textual facts. Build the prefix tree in batches — hardware memory limits do not allow the entire tree to be kept in memory — and persist nodes in a PostgreSQL fact-tree table storing next-token sets and reachable-leaf counts to prevent duplicate fact generation. Beyond a cutoff length, pack subtrees with Pickle and apply single-row single-leaf compression to reduce the number of rows traversed at each decoding step. Create a B-Tree index on the Prefix column so prefix-match lookups at each constrained-generation step remain efficient. Validate by issuing next-token queries along sample fact prefixes and verifying only tokens forming existing facts are reachable, cross-checking against the 31k-facts in-memory prefix tree in try_refactx.ipynb; configure the PostgreSQL connection via the .env file copied from env-sample.txt.

## Related tools

- **PostgreSQL** (Persist the batch-built prefix-tree nodes (fact-tree table with next-token sets and reachable-leaf counts) and host the B-Tree index on the Prefix column for efficient prefix-match lookups) — https://github.com/rpo19/ReFactX
- **Pickle** (Pack subtrees beyond the cutoff length as compressed blobs to reduce rows traversed during traversal) — https://github.com/rpo19/ReFactX
- **ReFactX** (Provides the ingestion pipeline, PrefixTree.md recipe, env-sample.txt configuration, and try_refactx.ipynb in-memory reference tree for validation) — https://github.com/rpo19/ReFactX

## Examples

```
cp env-sample.txt .env && python throughput.py --model MODEL --index INDEX --max-tokens 4001 --output out.json
```

## Evaluation signals

- Issue next-token queries along sample fact prefixes against the persisted tree and verify only tokens forming existing facts are reachable.
- Cross-check disk-backed index behavior against the 31k-facts in-memory prefix tree in try_refactx.ipynb.
- Confirm a B-Tree index exists on the Prefix column and that per-step prefix-match lookups remain efficient at 800M-fact scale.
- Confirm throughput overhead of constrained generation stays ~1%: 4000-token generation should take ~303.89s vs ~300.14s unconstrained (1.3% measured on one Nvidia Tesla T4 with Qwen2.5-3B).
- Verify no duplicate facts can be generated: reachable-leaf counts are stored per node for this purpose.

## Limitations

- The entire prefix tree cannot be kept in memory; ingestion must be performed in batches because of hardware memory limits.
- Constrained generation was measured to add ~1.3% latency (303.89s vs 300.14s for 4000 tokens) on a single Nvidia Tesla T4; other hardware may differ.
- ReFactX performs worse than LLM-only answering on datasets well covered by parametric knowledge (Mintaka, WebQSP, where LLM-only reaches 82.0% and 80.5% accuracy).

## Evidence

- [intro] Disk-backed index scale and latency: "a disk-backed prefix tree holds up to 800M facts and adds only ∼1 % latency."
- [other] Batched build and persistence in PostgreSQL: [evidence span withheld — anchor status: not_found]
- [other] Pickle packing and compression steps: [evidence span withheld — anchor status: not_found]
- [other] B-Tree index on Prefix column: [evidence span withheld — anchor status: not_found]
- [methods] Index construction recipe reference: "Refer to [PrefixTree.md](PrefixTree.md) for creating the Wikidata prefix tree we used in our work."
- [readme] Facts indexed in a prefix tree: [evidence span withheld — anchor status: not_found]
- [methods] PostgreSQL configuration via .env: "prepare the `.env` file: `cp env-sample.txt .env`, then edit `.env`"
