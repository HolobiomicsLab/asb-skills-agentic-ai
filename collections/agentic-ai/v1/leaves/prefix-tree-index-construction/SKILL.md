---
name: prefix-tree-index-construction
description: Use when triples from a large knowledge graph (e.g., a Wikidata truthy
  triples dump) must be verbalized, tokenized, and indexed in a prefix tree that exceeds
  memory and must be persisted for constrained generation lookups.
when_to_use_negative:
- The knowledge base is small enough to fit in memory — use the 31k-facts in-memory
  prefix tree via try_refactx.ipynb instead of the disk-backed PostgreSQL index
- The goal is retrieval-style question answering with an external retriever rather
  than constrained generation, since ReFactX is explicitly positioned as an alternative
  to pipeline-based retriever solutions
- The source data are not knowledge-graph triples (e.g., raw text corpora), since
  the pipeline starts by verbalizing KG triples into textual facts
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: PostgreSQL
  role: Persists the disk-backed prefix-tree nodes in batches, storing next-token
    sets and reachable-leaf counts, with a B-Tree index on the Prefix column for efficient
    prefix-match lookups at each decoding step
  repo: https://github.com/rpo19/ReFactX
- name: ReFactX
  role: Source package whose ingestion pipeline verbalizes triples, tokenizes facts,
    builds and persists the prefix tree; includes PrefixTree.md recipe and try_refactx.ipynb
    with a 31k-facts in-memory index for cross-checking
  repo: https://github.com/rpo19/ReFactX
- name: Qwen2.5-3B
  role: LLM used at inference time to traverse the index via constrained generation
    when measuring throughput (4000 tokens with key-value caching on one Nvidia Tesla
    T4)
  repo: https://github.com/rpo19/ReFactX
- name: Wikidata
  role: Source knowledge graph whose truthy triples are verbalized into 800 million
    textual facts and indexed in the prefix tree
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-09527-5_16
    title: 'ReFactX: Scalable Reasoning with Reliable Facts via Constrained Generation'
  source_content_hash: sha256:504dce6f2419d533888bf942bf724428afba7ec7eb08df9e0182d15096591f87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rpo19/ReFactX
  tools:
  - PostgreSQL
  - Qwen2.5-3B
  - ReFactX
  - Wikidata
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# prefix-tree-index-construction

## Summary

Construct and persist a disk-backed prefix-tree fact index (as in ReFactX) over very large knowledge bases such as Wikidata (~800M verbalized facts), so that LLM constrained generation can traverse only token sequences forming existing facts with ~1% added latency.

## When to use

Use when triples from a large knowledge graph (e.g., a Wikidata truthy triples dump) must be verbalized, tokenized, and indexed in a prefix tree that exceeds memory and must be persisted for constrained generation lookups.

## When NOT to use

- The knowledge base is small enough to fit in memory — use the 31k-facts in-memory prefix tree via try_refactx.ipynb instead of the disk-backed PostgreSQL index
- The goal is retrieval-style question answering with an external retriever rather than constrained generation, since ReFactX is explicitly positioned as an alternative to pipeline-based retriever solutions
- The source data are not knowledge-graph triples (e.g., raw text corpora), since the pipeline starts by verbalizing KG triples into textual facts

## Inputs

- Wikidata truthy triples dump (knowledge-graph triples)
- Verbalized natural-language textual fact corpus (up to 800 million textual facts)
- Tokenized fact sequences

## Outputs

- PostgreSQL fact-tree table with prefix-tree nodes (next-token sets, reachable-leaf counts)
- Pickle-packed subtrees beyond the cutoff length with single-row single-leaf compression
- B-Tree index on the Prefix column
- Validated disk-backed prefix-tree index supporting constrained generation

## How to apply

Load the knowledge-graph triples dump and verbalize each triple into a natural-language textual fact following the PrefixTree.md recipe. Tokenize the verbalized fact corpus (up to 800 million textual facts) and build the prefix tree in batches, since hardware memory limits do not allow the entire tree to be kept in memory; persist nodes in a PostgreSQL fact-tree table storing next-token sets and reachable-leaf counts to prevent duplicate fact generation. Pack subtrees with Pickle beyond the cutoff length and apply single-row single-leaf compression to reduce traversed rows. Create a B-Tree index on the Prefix column so prefix-match lookups at each decoding step stay efficient; the resulting disk-backed tree holds up to 800M facts and adds only ~1% latency. Validate by issuing next-token queries along sample fact prefixes and verifying only tokens forming existing facts are reachable, cross-checking against the 31k-facts in-memory prefix tree in try_refactx.ipynb.

## Related tools

- **PostgreSQL** (Persists the disk-backed prefix-tree nodes in batches, storing next-token sets and reachable-leaf counts, with a B-Tree index on the Prefix column for efficient prefix-match lookups at each decoding step) — https://github.com/rpo19/ReFactX
- **ReFactX** (Source package whose ingestion pipeline verbalizes triples, tokenizes facts, builds and persists the prefix tree; includes PrefixTree.md recipe and try_refactx.ipynb with a 31k-facts in-memory index for cross-checking) — https://github.com/rpo19/ReFactX
- **Qwen2.5-3B** (LLM used at inference time to traverse the index via constrained generation when measuring throughput (4000 tokens with key-value caching on one Nvidia Tesla T4)) — https://github.com/rpo19/ReFactX
- **Wikidata** (Source knowledge graph whose truthy triples are verbalized into 800 million textual facts and indexed in the prefix tree)

## Examples

```
python eval.py --index INDEX --model MODEL --dataset DATASET
```

## Evaluation signals

- Issue next-token queries along sample fact prefixes and verify only tokens forming existing facts are reachable
- Cross-check behavior against the 31k-facts in-memory prefix tree in try_refactx.ipynb
- Confirm constrained generation adds only ~1% latency vs unconstrained generation (~303.89s vs 300.14s for 4000 tokens, a 1.3% overhead)
- Confirm no duplicate fact generation, enabled by storing next-token sets and reachable-leaf counts per node
- Verify prefix-match lookups remain efficient via the B-Tree index on the Prefix column

## Limitations

- The entire tree cannot be kept in memory at 800M facts, so ingestion must be performed in batches and nodes persisted to PostgreSQL
- Performance was demonstrated with PostgreSQL and the LLM (Qwen2.5-3B) running on the same machine with a single Nvidia Tesla T4; other topologies are not characterized
- A preprint version (arXiv:2508.16983) has not undergone peer review; the peer-reviewed version is the ISWC 2025 paper

## Evidence

- [methods] ReFactX's fact-tree index is constructed in-memory and then persisted in a database during the ingestion process (performed in batches because hardware memory limits do not allow the entire tree to be kept in memory).: [evidence span withheld — anchor status: not_found]
- [readme] Triples from Wikidata are verbalized in 800 million textual facts, tokenized, and indexed in a prefix tree for efficient access.: [evidence span withheld — anchor status: not_found]
