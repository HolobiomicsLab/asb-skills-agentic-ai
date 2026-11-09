---
name: structured-topic-schema-generation
description: Use when enriched survey text responses must be partitioned into a fixed
  number M of exhaustive, non-overlapping topics, each yielding the four MALTopic
  fields (Topic Name, Description, Respondent Profile, Representative Words) for downstream
  deduplication.
when_to_use_negative:
- Input responses are raw and not yet enhanced by the enrichment agent, so the structured-data
  context MALTopic relies on is missing
- The analysis requires soft/overlapping topic memberships (e.g. probabilistic assignments
  as in LDA or BERTopic) rather than a hard, non-overlapping partition
- The corpus exceeds the model's input token limit and cannot be split into compliant
  batches
edam_operation: null
tools:
- name: MALTopic
  role: Multi-agent framework whose topic modeling agent (Agent 2) defines the batch
    partitioning, the four-field topic schema, and the exhaustiveness/non-overlap
    constraints
  repo: yash91sharma/MALTopic
- name: openai-python
  role: OpenAI Python client used to construct and send the LLM prompt that performs
    the batch topic extraction
  repo: openai/openai-python
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:f9135f6da8b9c4c3e320b5f1b8333647bdecf1e8d0ee4b5d83a847abeacd698c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - MALTopic
  - openai-python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# structured-topic-schema-generation

## Summary

Batch topic-extraction step of the MALTopic pipeline: an OpenAI LLM partitions a batch of N enriched survey responses into M exhaustive, non-overlapping, distinct latent themes, each serialized as a structured record with Topic Name, Description, Respondent Profile, and Representative Words.

## When to use

Use when enriched survey text responses must be partitioned into a fixed number M of exhaustive, non-overlapping topics, each yielding the four MALTopic fields (Topic Name, Description, Respondent Profile, Representative Words) for downstream deduplication.

## When NOT to use

- Input responses are raw and not yet enhanced by the enrichment agent, so the structured-data context MALTopic relies on is missing
- The analysis requires soft/overlapping topic memberships (e.g. probabilistic assignments as in LDA or BERTopic) rather than a hard, non-overlapping partition
- The corpus exceeds the model's input token limit and cannot be split into compliant batches

## Inputs

- Batch of N enriched survey text responses produced by the MALTopic enrichment agent
- Target topic count M
- Model input token limit used to size the batch

## Outputs

- topics.: valid JSON containing M topic records, each with Topic Name, Description, Respondent Profile, and Representative Words
- Topic membership assignment covering all N responses, exhaustive and mutually exclusive

## How to apply

Load the batch of N enriched textual responses produced by the upstream enrichment agent, splitting batches so they fit the model's input token limit. Construct an OpenAI LLM prompt via the openai-python client (chat.completions.create) instructing the model to partition the batch into exactly M latent themes under two constraints: exhaustiveness (every response is assigned) and non-overlap (no response appears in two topics). Request the four required fields per topic — Topic Name, Description, Respondent Profile, Representative Words — then parse the model output into one structured record per topic and validate schema conformance. Check the partition properties: the union of topics covers all enriched responses and topic memberships are mutually exclusive. Serialize the validated topic list to topics. for downstream refinement by the deduplication agent.

## Related tools

- **MALTopic** (Multi-agent framework whose topic modeling agent (Agent 2) defines the batch partitioning, the four-field topic schema, and the exhaustiveness/non-overlap constraints) — yash91sharma/MALTopic
- **openai-python** (OpenAI Python client used to construct and send the LLM prompt that performs the batch topic extraction) — openai/openai-python

## Examples

```
from openai import OpenAI; client = OpenAI(); completion = client.chat.completions.create(model="gpt-5.5", messages=[{"role":"user","content": topic_extraction_prompt}])
```

## Evaluation signals

- topics. exists and parses as valid JSON
- The file contains exactly M topic records, each conforming to the schema with all four required fields (Topic Name, Description, Respondent Profile, Representative Words)
- Union of topic memberships covers every enriched response in the batch (exhaustiveness)
- No response appears in more than one topic (mutual exclusivity / non-overlap)
- Topic set is distinct, i.e. no duplicate or degenerate topics before deduplication refinement

## Limitations

- Batch size N is constrained by the LLM's input token limit, so very large response sets must be split across multiple batches
- Output is model-generated and must be parsed and schema-validated; the LLM may violate the exhaustiveness or non-overlap constraints and require re-prompting
- Topics produced here still require refinement by the downstream deduplication agent before final use

## Evidence

- [readme] openai-python library purpose: [evidence span withheld — anchor status: external_doc]
