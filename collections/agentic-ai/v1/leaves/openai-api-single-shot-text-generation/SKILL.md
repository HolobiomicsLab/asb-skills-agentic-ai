---
name: openai-api-single-shot-text-generation
description: Use when you have N free-text survey responses each paired with structured
  fields (e.g., job title, work experience) and need one enriched response per input
  record via a single LLM call per row.
when_to_use_negative:
- The survey data lacks structured fields (job title, work experience) that the enrichment
  prompt is meant to weave into the free text
- The task requires iterative refinement or multi-turn dialogue with the model — this
  skill is strictly single-shot, one call per response
- Responses are already enriched with structured context and only need topic extraction
  (e.g., the MALTopic topic modeling agent stage)
tools:
- name: openai-python
  role: Python SDK used to invoke GPT-4o-mini in single-shot chat-completions mode,
    one call per free-text response
  repo: https://github.com/openai/openai-python
- name: GPT-4o-mini
  role: LLM that generates each enriched response by embedding structured job-title/work-experience
    context into the free text
- name: MALTopic
  role: Framework whose enrichment-prompt template defines how structured context
    is combined with the free-text response
  repo: https://github.com/yash91sharma/MALTopic
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:6152c807d3cf023dd52ed6fa536ce6fbc8583c77e171e51bd57f7a41cd96395e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/openai/openai-python
  tools:
  - GPT-4o-mini
  - MALTopic
  - openai-python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# openai-api-single-shot-text-generation

## Summary

Generates enriched free-text survey responses by calling GPT-4o-mini through the openai-python SDK in single-shot mode — one API call per response, no iterative refinement — to weave structured respondent context into free-text answers for the MALTopic enrichment agent.

## When to use

Use when you have N free-text survey responses each paired with structured fields (e.g., job title, work experience) and need one enriched response per input record via a single LLM call per row.

## When NOT to use

- The survey data lacks structured fields (job title, work experience) that the enrichment prompt is meant to weave into the free text
- The task requires iterative refinement or multi-turn dialogue with the model — this skill is strictly single-shot, one call per response
- Responses are already enriched with structured context and only need topic extraction (e.g., the MALTopic topic modeling agent stage)

## Inputs

- N free-text survey responses (raw free-text answers)
- Structured survey fields per respondent: job title and work experience
- Enrichment-prompt template from the yash91sharma/MALTopic repo

## Outputs

- enriched_responses.csv with exactly one enriched row per input response, each embedding the job-title/work-experience context

## How to apply

Load the N free-text responses together with their structured fields from the survey dataset, and load the example enrichment-prompt template from the yash91sharma/MALTopic GitHub repo that specifies how structured context should be woven into a response. For each response, construct the enrichment prompt by combining the free-text answer with the respondent's job title and work experience per the template. Instantiate an OpenAI client via the openai-python library and issue exactly one chat-completions call per response with model 'gpt-4o-mini' — no multi-turn conversation or iterative refinement — because the enrichment agent is designed as a single, independent per-row transformation. Collect all outputs and write them to enriched_responses.csv in a row-per-response structure aligned with the input records, so downstream MALTopic agents (topic modeling, deduplication) consume a 1:1 mapped dataset.

## Related tools

- **openai-python** (Python SDK used to invoke GPT-4o-mini in single-shot chat-completions mode, one call per free-text response) — https://github.com/openai/openai-python
- **GPT-4o-mini** (LLM that generates each enriched response by embedding structured job-title/work-experience context into the free text)
- **MALTopic** (Framework whose enrichment-prompt template defines how structured context is combined with the free-text response) — https://github.com/yash91sharma/MALTopic

## Examples

```
from openai import OpenAI; client = OpenAI(); completion = client.chat.completions.create(model="gpt-4o-mini", messages=[{"role":"user","content":enrichment_prompt}])
```

## Evaluation signals

- enriched_responses.csv exists and contains exactly one enriched row per input response (row count matches N)
- Each enriched response retains the original free-text answer content while explicitly incorporating the job-title/work-experience context
- Each enriched response was produced by exactly one GPT-4o-mini API call (no iterative refinement loop)
- Output CSV keeps a row-per-response structure aligned with the input records so downstream MALTopic agents can consume it
- Prompt construction follows the example enrichment-prompt template deposited in the yash91sharma/MALTopic repo

## Limitations

- Single-shot mode gives the model no opportunity to correct omissions — if structured context is dropped from an enriched response, the row must be regenerated manually
- Quality of enrichment depends on the enrichment-prompt template from the MALTopic repo; malformed prompts may fail to weave in structured data
- An input of N responses produces exactly N enriched responses — there is no aggregation or merging across respondents

## Evidence

- [readme] openai-python library purpose: [evidence span withheld — anchor status: external_doc]
- [readme] Chat Completions API as text-generation interface: [evidence span withheld — anchor status: not_found]
