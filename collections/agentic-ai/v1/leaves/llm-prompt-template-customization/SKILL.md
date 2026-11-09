---
name: llm-prompt-template-customization
description: Use when free-text survey responses must be augmented with domain- or
  industry-specific context before downstream topic modeling, while structured respondent
  data must be preserved unchanged.
when_to_use_negative:
- The input has no free-text component to enrich (e.g. responses are already fully
  structured fields)
- Downstream consumers require the original unmodified free-text responses, or schema
  changes are unacceptable
- The task is topic extraction or deduplication rather than response enrichment —
  those belong to Agent 2 (topic modeling) and Agent 3 (deduplication) in MALTopic
edam_operation: http://edamontology.org/operation_0306
tools:
- name: MALTopic
  role: Provides Agent 1's single-shot prompt template that is extended with the domain-context
    instruction before the enrichment run
  repo: https://github.com/yash91sharma/MALTopic
- name: openai-python
  role: Python SDK used to execute the customized prompt in single-shot mode, one
    API call per survey response
  repo: https://github.com/openai/openai-python
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:dc5adba4d5e07761351dad802ab5848e46ec94bdb8d1e226f4752a8f712c3eee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# llm-prompt-template-customization

## Summary

Customizes an LLM agent's prompt template — e.g. MALTopic's Agent 1 enrichment agent — so that free-text survey responses are augmented with domain- or industry-specific context injected via the OpenAI Python SDK, while structured respondent fields pass through untouched.

## When to use

Use when free-text survey responses must be augmented with domain- or industry-specific context before downstream topic modeling, while structured respondent data must be preserved unchanged.

## When NOT to use

- The input has no free-text component to enrich (e.g. responses are already fully structured fields)
- Downstream consumers require the original unmodified free-text responses, or schema changes are unacceptable
- The task is topic extraction or deduplication rather than response enrichment — those belong to Agent 2 (topic modeling) and Agent 3 (deduplication) in MALTopic

## Inputs

- Raw free-text survey responses (CSV) with accompanying structured respondent data
- Agent 1's single-shot prompt template from the MALTopic pipeline
- Domain- or industry-specific context to inject into the prompt instruction

## Outputs

- enriched_responses.csv with the same schema, containing context-enriched free-text responses and unchanged structured fields
- N-in/N-out verified record set for the downstream topic modeling agent

## How to apply

Open Agent 1's single-shot prompt template in the MALTopic pipeline and extend it with an instruction that injects domain- or industry-specific context into every free-text response, leaving the structured fields untouched. Run the enrichment agent over all responses via the openai-python SDK in single-shot mode, one API call per response. Before execution, load the raw survey responses together with the accompanying structured respondent data that the enrichment agent consumes. After execution, verify the N-in/N-out mapping holds: the same number of records enter and exit, and each record's structured fields are identical to the input. Finally, write the enriched responses to enriched_responses.csv in the same schema so the downstream topic modeling and deduplication agents can consume them without schema changes.

## Related tools

- **MALTopic** (Provides Agent 1's single-shot prompt template that is extended with the domain-context instruction before the enrichment run) — https://github.com/yash91sharma/MALTopic
- **openai-python** (Python SDK used to execute the customized prompt in single-shot mode, one API call per survey response) — https://github.com/openai/openai-python

## Examples

```
from openai import OpenAI; client = OpenAI(); resp = client.chat.completions.create(model="gpt-4o", messages=[{"role": "user", "content": enriched_prompt}])
```

## Evaluation signals

- N-in/N-out invariant: the number of records written to enriched_responses.csv equals the number of raw input responses
- Structured respondent fields in each output record are byte-identical to the corresponding input record
- Output CSV schema matches the input schema exactly, with only free-text response content enriched
- Spot-check that enriched responses actually contain domain/industry context introduced by the prompt instruction
- Every response was processed exactly once (one single-shot API call per record, no dropped or duplicated records)

## Limitations

- The prompt customization only targets free-text responses; it cannot alter or repair structured fields, and any enrichment quality is bounded by the domain context supplied in the template
- Single-shot mode means one LLM call per response, which scales cost and latency linearly with response count and is subject to OpenAI API availability and authentication (e.g. OPENAI_API_KEY must be configured and not stored in source control)

## Evidence

- [other] Finding: agents can take on more sophisticated enrichment tasks: [evidence span withheld — anchor status: not_found]
- [readme] OpenAI Python library used to execute the customized prompt: [evidence span withheld — anchor status: external_doc]
