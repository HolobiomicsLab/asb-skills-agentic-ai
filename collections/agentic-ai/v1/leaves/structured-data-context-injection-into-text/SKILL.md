---
name: structured-data-context-injection-into-text
description: Use when a corpus of free-text survey answers is paired with structured
  respondent fields (e.g. job title, work experience) and you need each answer independently
  enhanced with that context before topic modeling.
when_to_use_negative:
- Responses have no paired structured fields (no job title, work experience, etc.)
  to inject as context
- Iterative multi-turn refinement of each response is required — the enrichment step
  is single-shot by design
- The text units are not one-per-respondent records, so row-per-response alignment
  with structured fields cannot be maintained
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0008
tools:
- name: openai-python
  role: SDK used to issue one single-shot GPT-4o-mini chat completion per survey response
    to generate the enriched text
  repo: openai/openai-python
- name: GPT-4o-mini
  role: LLM that rewrites each free-text answer with the structured job-title/work-experience
    context embedded
- name: MALTopic
  role: Framework supplying the enrichment-prompt template; the enrichment step is
    Agent 1 of its multi-agent pipeline
  repo: yash91sharma/MALTopic
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:b91d79c84e16ec8fe5784b26cc584f681eb7aad4d5e7e4c38364392582cdc257
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/openai/openai-python
  tools:
  - GPT-4o-mini
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0008
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# structured-data-context-injection-into-text

## Summary

Per-record prompt-based enrichment in which structured fields (here: job title and work experience from a survey dataset) are woven into each corresponding free-text response by a single-shot GPT-4o-mini call, yielding one enriched response per input and giving downstream topic modeling agents respondent context.

## When to use

Use when a corpus of free-text survey answers is paired with structured respondent fields (e.g. job title, work experience) and you need each answer independently enhanced with that context before topic modeling.

## When NOT to use

- Responses have no paired structured fields (no job title, work experience, etc.) to inject as context
- Iterative multi-turn refinement of each response is required — the enrichment step is single-shot by design
- The text units are not one-per-respondent records, so row-per-response alignment with structured fields cannot be maintained

## Inputs

- Survey dataset with N free-text responses plus paired structured fields (job title, work experience)
- Enrichment-prompt template from yash91sharma/MALTopic specifying how structured context is woven into a response
- OpenAI API key (OPENAI_API_KEY) for GPT-4o-mini access

## Outputs

- enriched_responses.csv with one enriched row per input response, each retaining the original answer content with job-title/work-experience context embedded

## How to apply

Load the N free-text survey responses together with their structured fields (job title, work experience) and combine each answer with its respondent's fields according to an enrichment-prompt template from the MALTopic repo (yash91sharma/MALTopic). For every response, issue exactly one single-shot call to GPT-4o-mini via the openai-python SDK — no iterative refinement loop — so an input of N responses produces exactly N enriched responses. Write the results to enriched_responses.csv keeping a row-per-response structure aligned with the input records, as the enrichment agent's role is to 'leverage structured data to enhance textual responses'. The rationale is that embedding structured context per respondent yields more human-readable, contextually relevant topics in the subsequent MALTopic modeling stages.

## Related tools

- **openai-python** (SDK used to issue one single-shot GPT-4o-mini chat completion per survey response to generate the enriched text) — openai/openai-python
- **GPT-4o-mini** (LLM that rewrites each free-text answer with the structured job-title/work-experience context embedded)
- **MALTopic** (Framework supplying the enrichment-prompt template; the enrichment step is Agent 1 of its multi-agent pipeline) — yash91sharma/MALTopic

## Examples

```
from openai import OpenAI; client = OpenAI(); completion = client.chat.completions.create(model="gpt-4o-mini", messages=[{"role":"user","content": enrichment_prompt}]); print(completion.choices[0].message.content)
```

## Evaluation signals

- enriched_responses.csv exists and contains exactly one enriched row per input response (N in = N out)
- Each enriched response retains the original answer content rather than replacing it
- Each enriched response explicitly incorporates the respondent's structured job-title/work-experience context
- Exactly one GPT-4o-mini call per response was made (single-shot, no iterative refinement)
- Row alignment between enriched_responses.csv and the input records is preserved

## Limitations

- Enrichment is single-shot per response with no iterative refinement, so poorly worded enrichments cannot be corrected in-loop
- Quality of enrichment depends entirely on the presence and informativeness of the structured fields (job title, work experience); records lacking these cannot be meaningfully enriched
- Requires an OpenAI API key and incurs one LLM call per response, so cost scales linearly with survey size

## Evidence

- [readme] openai-python SDK purpose: [evidence span withheld — anchor status: external_doc]
