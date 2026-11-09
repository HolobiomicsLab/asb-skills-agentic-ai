---
name: exhaustive-nonoverlap-partition-validation
description: Use when an LLM topic-modeling step (e.g. MALTopic Agent 2) has emitted
  a topic set over a batch of enriched responses and you must verify exhaustiveness,
  non-overlap, and per-topic schema conformance before downstream deduplication.
when_to_use_negative:
- The input is raw, un-enriched survey responses that have not yet passed through
  the enrichment agent — this skill validates the topic modeling agent's output, not
  its input.
- The task is topic quality assessment (coherence, diversity vs. LDA/BERTopic baselines)
  rather than partition-property checking.
- The topic set is intended to allow multi-label or overlapping topic assignments
  — the MALTopic design requires mutually exclusive membership.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: MALTopic
  role: Multi-agent framework whose Agent 2 (topic modeling agent) produces the batch
    topic set that this skill validates
  repo: https://github.com/yash91sharma/MALTopic
- name: openai-python
  role: OpenAI Python client used to construct and invoke the LLM prompt that constrains
    the model to M exhaustive, non-overlapping topics
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:adbae73c4af239f07ccb5647ed2e4205b6500487f4820cecdd34831800b820c4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# Exhaustive non-overlap partition validation

## Summary

Validates that the MALTopic topic modeling agent's batch output forms a proper partition of the enriched survey responses: every response is assigned to exactly one of M topics, topics are mutually exclusive, and each topic record carries the four required fields (Topic Name, Description, Respondent Profile, Representative Words).

## When to use

Use when an LLM topic-modeling step (e.g. MALTopic Agent 2) has emitted a topic set over a batch of enriched responses and you must verify exhaustiveness, non-overlap, and per-topic schema conformance before downstream deduplication.

## When NOT to use

- The input is raw, un-enriched survey responses that have not yet passed through the enrichment agent — this skill validates the topic modeling agent's output, not its input.
- The task is topic quality assessment (coherence, diversity vs. LDA/BERTopic baselines) rather than partition-property checking.
- The topic set is intended to allow multi-label or overlapping topic assignments — the MALTopic design requires mutually exclusive membership.

## Inputs

- topics file (JSON) with M topic records, each containing Topic Name, Description, Respondent Profile, and Representative Words
- batch of N enriched textual survey responses from the upstream enrichment agent

## Outputs

- validation verdict that the topic set is exhaustive and non-overlapping
- schema-conformance report for the four per-topic fields
- validated topic list passed downstream to the deduplication agent

## How to apply

Load the serialized topics file produced by the topic modeling agent and first check schema conformance: the file must be valid JSON containing exactly M topic records, each with Topic Name, Description, Respondent Profile, and Representative Words. Then reconstruct the response-to-topic assignments and verify the partition properties: the union of topic memberships covers every enriched response in the batch (exhaustiveness, matching the prompt's constraint that every response be assigned), and no response appears in two topics (non-overlap). The upstream LLM prompt — built with the openai-python client — explicitly requests exactly M latent themes with these constraints, so validation is checking that the model obeyed its instructions. If either property fails, return the batch to the topic modeling agent for re-invocation rather than passing a defective partition to the deduplication agent.

## Related tools

- **MALTopic** (Multi-agent framework whose Agent 2 (topic modeling agent) produces the batch topic set that this skill validates) — https://github.com/yash91sharma/MALTopic
- **openai-python** (OpenAI Python client used to construct and invoke the LLM prompt that constrains the model to M exhaustive, non-overlapping topics) — https://github.com/openai/openai-python

## Examples

```
import ; topics = .load(open('topics.')); assert len(topics) == M and all(set(t) == {'Topic Name','Description','Respondent Profile','Representative Words'} for t in topics)
```

## Evaluation signals

- topics file exists and parses as valid JSON
- JSON contains exactly M topic records
- every topic record contains all four required fields: Topic Name, Description, Respondent Profile, Representative Words
- union of topic memberships covers all N enriched responses in the batch (exhaustiveness)
- no enriched response is assigned to more than one topic (mutual exclusivity)

## Limitations

- The LLM may violate the exhaustiveness or non-overlap constraints in its output despite being instructed to obey them, so the partition must be checked programmatically.
- Batch size N is bounded by the model's input token limit, so very large response sets must be split into multiple batches before partitioning.
- Validation confirms structural partition properties only; it does not assess semantic topic quality, which the paper evaluates separately via coherence, diversity, and interpretability against LDA and BERTopic.

## Evidence

- [full_text] Workflow step requiring the four per-topic fields and partition checks: [evidence span withheld — anchor status: not_found]
- [readme] openai-python client used for LLM invocation: [evidence span withheld — anchor status: external_doc]
