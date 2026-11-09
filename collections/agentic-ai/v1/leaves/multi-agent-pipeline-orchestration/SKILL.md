---
name: multi-agent-pipeline-orchestration
description: Use when a survey-text topic modeling task needs to be decomposed into
  sequential cooperating LLM agents — enrichment, theme extraction, and deduplication
  — rather than a single-pass topic model like LDA or BERTopic.
when_to_use_negative:
- The task is a standard topic modeling benchmark where LDA or BERTopic suffices and
  LLM cost/latency is not justified
- The corpus is not survey-style short textual responses paired with structured data
  that the enrichment agent can exploit
- A single LLM call can handle the whole corpus without batching — the multi-agent
  split adds orchestration overhead without benefit
edam_topics: []
tools:
- name: MALTopic
  role: Framework whose three-agent pipeline (enrichment → topic modeling → deduplication)
    this skill orchestrates
  repo: yash91sharma/MALTopic
- name: openai-python
  role: Python client used to invoke the OpenAI LLM calls for the topic modeling agent's
    batch extraction step
  repo: openai/openai-python
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:794d724b9e9fc42fab84417ecba256f4f88a0882dd25f69152084372643ee199
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
yes_rate: 0.6
n_claims: 10
---

# multi-agent-pipeline-orchestration

## Summary

Orchestrates the MALTopic multi-agent LLM pipeline, chaining an enrichment agent (structured-data augmentation of survey responses), a topic modeling agent (LLM-based latent theme extraction), and a deduplication agent (result refinement) into a validated sequential workflow.

## When to use

Use when a survey-text topic modeling task needs to be decomposed into sequential cooperating LLM agents — enrichment, theme extraction, and deduplication — rather than a single-pass topic model like LDA or BERTopic.

## When NOT to use

- The task is a standard topic modeling benchmark where LDA or BERTopic suffices and LLM cost/latency is not justified
- The corpus is not survey-style short textual responses paired with structured data that the enrichment agent can exploit
- A single LLM call can handle the whole corpus without batching — the multi-agent split adds orchestration overhead without benefit

## Inputs

- Batch of enriched textual survey responses (output of the enrichment agent)
- Target topic count M for the partition
- Structured survey data used by the enrichment agent to augment responses

## Outputs

- topics JSON file containing M topic records, each with Topic Name, Description, Respondent Profile, and Representative Words
- Validated topic assignments forming an exhaustive, non-overlapping partition of the enriched responses
- Refined topic set after the deduplication agent stage

## How to apply

Run the three agents in the MALTopic order: first the enrichment agent leverages structured data to enhance the raw textual survey responses; second the topic modeling agent batch-processes the enriched responses — the LLM concurrently analyzes a batch of N responses constrained by the model's input token limit and yields M exhaustive, non-overlapping, distinct topics; third the deduplication agent refines the topic modeling results. Invoke the topic modeling agent via the openai-python client with a prompt requiring exactly M themes and, per topic, the four fields Topic Name, Description, Respondent Profile, and Representative Words. Parse the LLM output into structured per-topic records, validate schema conformance for all four fields, and check the partition properties (coverage of all responses, mutually exclusive memberships). Serialize the topic list to topics JSON before handing off to the deduplication agent, so each agent's contract is a verifiable file artifact.

## Related tools

- **MALTopic** (Framework whose three-agent pipeline (enrichment → topic modeling → deduplication) this skill orchestrates) — yash91sharma/MALTopic
- **openai-python** (Python client used to invoke the OpenAI LLM calls for the topic modeling agent's batch extraction step) — openai/openai-python

## Examples

```
from openai import OpenAI; client = OpenAI(); r = client.chat.completions.create(model="gpt-4o", messages=[{"role":"user","content":"Partition these N enriched responses into exactly M non-overlapping topics; return Topic Name, Description, Respondent Profile, Representative Words per topic"}])
```

## Evaluation signals

- topics JSON file exists and parses as valid JSON
- Contains exactly M topic records, each carrying all four required fields: Topic Name, Description, Respondent Profile, Representative Words
- Union of topic memberships covers all enriched responses (exhaustive partition)
- No enriched response appears in more than one topic (non-overlapping memberships)
- Comparative check that topic coherence, diversity, and interpretability improve over LDA and BERTopic baselines on the survey dataset

## Limitations

- Batch size N is capped by the LLM's input token limit, so very large survey corpora must be split across multiple concurrent batch calls
- Exhaustiveness and non-overlap are requested via the prompt and must be validated post hoc; the LLM may return partitions violating these constraints
- Reported improvements (coherence, diversity, interpretability) are demonstrated on a single survey dataset, not generalized across domains

## Evidence

- [readme] openai-python client: [evidence span withheld — anchor status: external_doc]
