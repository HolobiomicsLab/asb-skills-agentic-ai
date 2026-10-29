---
name: llm-prompt-based-topic-extraction
description: Use when a batch of enriched textual survey responses must be partitioned
  into a fixed number M of latent themes, each documented with name, description,
  respondent profile, and representative words.
when_to_use_negative:
- The input is raw, un-enriched responses — MALTopic's pipeline expects the enrichment
  agent to have leveraged structured data to enhance textual responses first
- The task is benchmarking against classical topic models (LDA, BERTopic) rather than
  executing the MALTopic agent pipeline
- The batch exceeds the OpenAI model's input token limit and cannot be split into
  compliant batches
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: MALTopic
  role: Multi-agent framework whose topic modeling agent (Agent 2) executes the batch
    extraction step before deduplication agent refinement
  repo: https://github.com/yash91sharma/MALTopic
- name: openai-python
  role: OpenAI Python client used to construct the LLM prompt and invoke the model
    over the response batch
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:023154d4bd30c4c0dc3beea47c06b655aa1240a193473b09be2a539a39c331db
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# llm-prompt-based-topic-extraction

## Summary

Batch topic extraction from enriched survey responses using an OpenAI LLM prompted to return M exhaustive, non-overlapping, distinct topics, as done by the MALTopic topic modeling agent (Agent 2).

## When to use

Use when a batch of enriched textual survey responses must be partitioned into a fixed number M of latent themes, each documented with name, description, respondent profile, and representative words.

## When NOT to use

- The input is raw, un-enriched responses — MALTopic's pipeline expects the enrichment agent to have leveraged structured data to enhance textual responses first
- The task is benchmarking against classical topic models (LDA, BERTopic) rather than executing the MALTopic agent pipeline
- The batch exceeds the OpenAI model's input token limit and cannot be split into compliant batches

## Inputs

- Batch of enriched textual survey responses from the MALTopic enrichment agent
- Target number of topics M (constrained by model input token limit)

## Outputs

- topics. containing M topic records, each with Topic Name, Description, Respondent Profile, and Representative Words
- Exhaustive, mutually exclusive topic assignments over all enriched responses

## How to apply

Load the batch of enriched textual responses produced by the upstream MALTopic enrichment agent, sized so the batch fits within the OpenAI model's input token limit. Construct an OpenAI LLM prompt via the openai-python client that instructs the model to partition the batch into exactly M latent themes under two constraints: exhaustiveness (every response is assigned to a topic) and non-overlap (no response appears in two topics). Invoke the topic-modeling agent over the batch, requesting per topic the four required fields — Topic Name, Description, Respondent Profile, and Representative Words. Parse the model output into one structured record per topic, validate schema conformance for all four fields, then check the partition properties: the union of topics covers all enriched responses and memberships are mutually exclusive. Serialize the topic list to topics. for downstream refinement by the MALTopic deduplication agent.

## Related tools

- **MALTopic** (Multi-agent framework whose topic modeling agent (Agent 2) executes the batch extraction step before deduplication agent refinement) — https://github.com/yash91sharma/MALTopic
- **openai-python** (OpenAI Python client used to construct the LLM prompt and invoke the model over the response batch) — https://github.com/openai/openai-python

## Examples

```
from openai import OpenAI; client = OpenAI(); completion = client.chat.completions.create(model="gpt-5.5", messages=[{"role": "user", "content": batch_prompt}])
```

## Evaluation signals

- topics. exists and is valid JSON
- topics. contains exactly M topic records, each with the four required fields (Topic Name, Description, Respondent Profile, Representative Words)
- Partition exhaustiveness: the union of topic assignments covers every enriched response in the batch
- Partition non-overlap: no enriched response is assigned to two topics
- Topics are distinct (no duplicates or overlapping themes) per the MALTopic pipeline design

## Limitations

- Batch size N is bounded by the OpenAI model's input token limit, so very large response sets must be processed in multiple batches
- The partition constraints (exhaustiveness, non-overlap, distinctness) must be validated post hoc because LLM output is not guaranteed to satisfy them
- Output topics still require downstream refinement by the deduplication agent before final results

## Evidence
