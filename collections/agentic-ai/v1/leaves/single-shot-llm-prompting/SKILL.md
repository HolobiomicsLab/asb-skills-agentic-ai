---
name: single-shot-llm-prompting
description: Use when a multi-agent LLM pipeline needs free-text responses enriched
  with domain or industry context before downstream topic modeling, and each record
  can be processed independently in one LLM call.
when_to_use_negative:
- Inputs are structured fields only and require no free-text enrichment — the prompt
  extension targets free-text responses specifically
- The task requires multi-turn or stateful LLM reasoning across records — this skill
  is strictly single-shot, one call per response
- Downstream components expect the original raw response text rather than enriched
  responses
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MALTopic
  role: Multi-agent LLM topic modeling framework whose Agent 1 (enrichment agent)
    prompt template is extended for single-shot domain-context injection
  repo: https://github.com/yash91sharma/MALTopic
- name: openai-python
  role: Python SDK used to run the enrichment agent over all responses in single-shot
    mode, one API call per response
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:975e2a3f39673b05b5ca9ca12d0833967ab3a07591c2715c5e143afcc4081779
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-shot-llm-prompting

## Summary

Extend the MALTopic enrichment agent (Agent 1) by modifying its single-shot prompt template to inject domain- or industry-specific context into free-text survey responses while leaving structured respondent fields untouched, executing one OpenAI SDK call per response.

## When to use

Use when a multi-agent LLM pipeline needs free-text responses enriched with domain or industry context before downstream topic modeling, and each record can be processed independently in one LLM call.

## When NOT to use

- Inputs are structured fields only and require no free-text enrichment — the prompt extension targets free-text responses specifically
- The task requires multi-turn or stateful LLM reasoning across records — this skill is strictly single-shot, one call per response
- Downstream components expect the original raw response text rather than enriched responses

## Inputs

- Raw survey free-text responses
- Accompanying structured respondent data consumed by the enrichment agent
- Agent 1's single-shot prompt template from the MALTopic pipeline

## Outputs

- enriched_responses.csv in the same schema, with domain-context-enriched free text and unchanged structured fields

## How to apply

Load raw survey responses together with their accompanying structured respondent data, then open Agent 1's single-shot prompt template in the MALTopic pipeline and add an instruction that injects domain- or industry-specific context into every free-text response, leaving structured fields untouched. Run the enrichment agent over all responses via the OpenAI Python SDK in single-shot mode, issuing exactly one call per response rather than batching or conversational multi-turn calls, since the enrichment task is stateless and record-local. After the run, verify the N-in/N-out mapping: the same number of records must exit the agent as entered, with each record's structured fields identical to its input. Write the enriched responses to enriched_responses.csv in the same schema so the downstream topic modeling agent can consume them without format changes. The rationale is that prompt-level domain injection is the extension point for the enrichment agent's role beyond its original task of augmenting free text with structured data.

## Related tools

- **MALTopic** (Multi-agent LLM topic modeling framework whose Agent 1 (enrichment agent) prompt template is extended for single-shot domain-context injection) — https://github.com/yash91sharma/MALTopic
- **openai-python** (Python SDK used to run the enrichment agent over all responses in single-shot mode, one API call per response) — https://github.com/openai/openai-python

## Examples

```
from openai import OpenAI; client = OpenAI(); completion = client.chat.completions.create(model="gpt-5.5", messages=[{"role": "developer", "content": "Inject domain-specific context into the response; leave structured fields unchanged."}, {"role": "user", "content": response_text}])
```

## Evaluation signals

- N-in/N-out invariant: number of enriched records equals number of input records
- Per-record structured fields are byte-identical between input and enriched output
- Enriched output is written to enriched_responses.csv in the same schema as the input
- Each enriched free-text response contains injected domain/industry context while the original response content remains traceable
- Exactly one OpenAI SDK call was made per response (single-shot mode, no multi-turn state)

## Limitations

- Structured fields must remain untouched — any drift in them breaks the N-in/N-out schema contract
- Single-shot mode means each call is independent; cross-record context or conversational memory cannot be injected
- The approach is validated on a survey dataset in the MALTopic framework; transfer to other text corpora is unverified here

## Evidence

- [other] Single-shot enrichment prompt extension: [evidence span withheld — anchor status: not_found]
- [other] Sophisticated enrichment tasks: [evidence span withheld — anchor status: not_found]
