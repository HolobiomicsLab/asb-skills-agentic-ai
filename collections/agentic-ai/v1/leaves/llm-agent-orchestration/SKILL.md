---
name: llm-agent-orchestration
description: Use when processing survey text responses that need enriched context,
  latent theme extraction, and deduplication of overlapping topics via a chain of
  cooperating LLM agents rather than a single-pass model.
when_to_use_negative:
- The response set is small enough that the topic modeling agent can process it in
  a single pass with no duplicated or overlapping topics
- The task requires a numeric, threshold-based deduplication guarantee rather than
  qualitative LLM judgment of semantic overlap
- The input is not survey text (the MALTopic pipeline was evaluated on a survey dataset,
  not arbitrary corpora)
edam_operation: http://edamontology.org/operation_0306
tools:
- name: MALTopic
  role: The multi-agent LLM framework whose enrichment, topic modeling, and deduplication
    agents are orchestrated in sequence
  repo: https://github.com/yash91sharma/MALTopic
- name: openai-python
  role: OpenAI Python API library used to invoke the LLM behind each agent (Responses
    API, sync/async clients)
  repo: https://github.com/openai/openai-python
- name: BERTopic
  role: Baseline topic modeling approach against which MALTopic's agent-produced topics
    are compared for coherence, diversity, and interpretability
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:e052f07bc5df9782fea73046528d8edf75e6315cd1268a2c3a8cd09de69332d7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# llm-agent-orchestration

## Summary

Orchestrates the MALTopic multi-agent LLM pipeline—enrichment agent, topic modeling agent, and deduplication agent—as sequential LLM-backed stages, using the OpenAI Python client to drive the agents and BERTopic-style topic extraction to produce refined, human-readable survey topics.

## When to use

Use when processing survey text responses that need enriched context, latent theme extraction, and deduplication of overlapping topics via a chain of cooperating LLM agents rather than a single-pass model.

## When NOT to use

- The response set is small enough that the topic modeling agent can process it in a single pass with no duplicated or overlapping topics
- The task requires a numeric, threshold-based deduplication guarantee rather than qualitative LLM judgment of semantic overlap
- The input is not survey text (the MALTopic pipeline was evaluated on a survey dataset, not arbitrary corpora)

## Inputs

- Raw textual survey responses
- Structured data records for response enrichment
- Topic list emitted by the upstream Topic Modeling Agent (Agent 2)

## Outputs

- Enriched textual responses
- Latent topic list from the topic modeling agent
- Refined deduplicated topic list (deduplicated_topics) with distinct, human-readable topic labels

## How to apply

Chain three agents in order: (1) an enrichment agent that leverages structured data to enhance the raw textual survey responses, (2) a topic modeling agent that extracts latent themes from the enriched responses, and (3) a deduplication agent that refines the topic list. The deduplication agent is required when the number of survey responses exceeds the topic modeling agent's capacity for single-pass processing: it loads the emitted topic list, compares topic labels/keywords pairwise with an LLM to identify duplicate and semantically overlapping themes, merges each duplicate cluster into one representative topic, and writes a refined list (e.g. deduplicated_topics). Drive each agent with the OpenAI Python SDK (synchronous or asynchronous client); no numeric deduplication threshold is stated, so acceptance is qualitative.

## Related tools

- **MALTopic** (The multi-agent LLM framework whose enrichment, topic modeling, and deduplication agents are orchestrated in sequence) — https://github.com/yash91sharma/MALTopic
- **openai-python** (OpenAI Python API library used to invoke the LLM behind each agent (Responses API, sync/async clients)) — https://github.com/openai/openai-python
- **BERTopic** (Baseline topic modeling approach against which MALTopic's agent-produced topics are compared for coherence, diversity, and interpretability)

## Examples

```
from openai import OpenAI; client = OpenAI(); response = client.responses.create(model="gpt-5.5", instructions="Deduplicate overlapping survey topics", input=topic_list); deduplicated_topics = response.output_text
```

## Evaluation signals

- Every entry in the refined topic list is distinct — no duplicate or semantically overlapping topics remain
- Topic coherence, diversity, and interpretability exceed LDA and BERTopic baselines on the survey dataset
- Topics are human-readable with enhanced contextual relevance attributable to the structured-data enrichment step
- Each agent stage completes in order (enrichment → topic modeling → deduplication) with its output feeding the next agent
- The OpenAI client calls complete successfully (sync or async), with the deduplication agent consuming the exact topic list emitted by Agent 2

## Limitations

- Agent 2 cannot process large response volumes in a single pass, forcing the deduplication agent; scaling limits are dataset-dependent
- No numeric deduplication threshold is stated, so duplicate/overlap decisions rest on qualitative LLM judgment
- Reported improvements (coherence, diversity, interpretability) were demonstrated on a single survey dataset

## Evidence
