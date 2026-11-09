---
name: survey-response-latent-theme-modeling
description: Use when you hold a batch of enriched textual survey responses (output
  of the upstream enrichment agent) and need M human-readable latent themes with coverage
  of every response and no overlap between topics.
when_to_use_negative:
- Inputs are raw, unenriched survey responses that have not passed the MALTopic enrichment
  agent — this step expects structured-data-enhanced text
- The task is comparative benchmarking of classical baselines such as LDA or BERTopic
  rather than LLM-based theme extraction within the MALTopic pipeline
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: MALTopic
  role: Multi-agent LLM framework whose topic modeling agent (Agent 2) executes the
    batch partition into M latent themes
  repo: https://github.com/yash91sharma/MALTopic
- name: openai-python
  role: Python client used to construct and send the OpenAI LLM prompt that performs
    the exhaustive, non-overlapping topic partition
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:f5bcd346ccd35e01ac6147edfc7225438e7d41f1d837a092b55302264658ab4b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Survey-response latent theme modeling

## Summary

Batch topic-extraction step of the MALTopic multi-agent LLM framework (Agent 2): an LLM concurrently partitions a batch of N enriched survey responses into M exhaustive, non-overlapping, distinct latent themes, emitting per-topic Topic Name, Description, Respondent Profile, and Representative Words.

## When to use

Use when you hold a batch of enriched textual survey responses (output of the upstream enrichment agent) and need M human-readable latent themes with coverage of every response and no overlap between topics.

## When NOT to use

- Inputs are raw, unenriched survey responses that have not passed the MALTopic enrichment agent — this step expects structured-data-enhanced text
- The task is comparative benchmarking of classical baselines such as LDA or BERTopic rather than LLM-based theme extraction within the MALTopic pipeline

## Inputs

- Batch of enriched textual survey responses from the upstream MALTopic enrichment agent (batch size N bounded by the LLM input token limit)
- Target number of topics M
- OpenAI API key (e.g. via OPENAI_API_KEY environment variable)

## Outputs

- topics JSON file containing M topic records, each with Topic Name, Description, Respondent Profile, and Representative Words
- Topic memberships that form an exhaustive, mutually exclusive partition of the enriched responses

## How to apply

Load the batch of enriched textual responses produced by the enrichment agent (the 'topic modeling agent extracts latent themes' stage of the MALTopic pipeline). Construct an OpenAI LLM prompt via the openai-python client instructing the model to partition the batch into exactly M latent themes under two constraints: exhaustiveness (every response assigned to a topic) and non-overlap (no response appears in two topics). The batch size N is constrained by the model's input token limit, so chunk responses accordingly. Request the four required fields per topic — Topic Name, Description, Respondent Profile, and Representative Words — parse the output into structured records, and validate schema conformance. Before serializing to topics JSON, check the partition properties: the union of topics covers all enriched responses and topic memberships are mutually exclusive; the result then feeds downstream refinement by the deduplication agent.

## Related tools

- **MALTopic** (Multi-agent LLM framework whose topic modeling agent (Agent 2) executes the batch partition into M latent themes) — https://github.com/yash91sharma/MALTopic
- **openai-python** (Python client used to construct and send the OpenAI LLM prompt that performs the exhaustive, non-overlapping topic partition) — https://github.com/openai/openai-python

## Examples

```
from openai import OpenAI; r = OpenAI().chat.completions.create(model="gpt-5.5", messages=[{"role":"user","content":"Partition these N enriched survey responses into M exhaustive, non-overlapping topics with Name, Description, Respondent Profile, Representative Words"}])
```

## Evaluation signals

- topics. exists and is valid JSON
- The file contains exactly M topic records, each carrying all four required fields (Topic Name, Description, Respondent Profile, Representative Words)
- Union of topic memberships covers every enriched response in the batch (exhaustiveness)
- No response ID appears in more than one topic (mutual exclusivity / non-overlap)
- Downstream acceptance: the serialized topic list is consumable by the MALTopic deduplication agent for refinement

## Limitations

- Batch size N is capped by the LLM model's input token limit, so large response sets must be split into multiple batches
- Topic quality depends on the upstream enrichment agent; MALTopic's coherence/diversity/interpretability gains are reported relative to LDA and BERTopic on a survey dataset
- The exhaustiveness and non-overlap partition properties are model outputs and must be explicitly validated rather than assumed

## Evidence

- [readme] openai-python provides LLM access used to issue the topic-partition prompt: [evidence span withheld — anchor status: external_doc]
- [readme] Chat Completions is the documented API surface for issuing prompts: [evidence span withheld — anchor status: not_found]
