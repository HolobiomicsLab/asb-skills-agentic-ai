---
name: llm-prompt-based-survey-response-enrichment
description: Use when you have a survey dataset with paired free-text answers and
  structured respondent fields, and you want each response augmented with that context
  before topic modeling (MALTopic-style enrichment).
when_to_use_negative:
- The input responses have no paired structured fields (e.g., job title or work experience)
  to weave in — there is nothing for the enrichment agent to leverage
- Responses are already enriched or the goal is topic extraction/refinement rather
  than enrichment — use MALTopic's topic-modeling or deduplication agent steps instead
- N is very large and per-response single-shot LLM calls are cost-prohibitive, since
  the design is one GPT-4o-mini call per response
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: MALTopic
  role: Framework whose enrichment agent performs the prompt-based response enrichment;
    provides the example enrichment-prompt template
  repo: yash91sharma/MALTopic
- name: openai-python
  role: Python SDK used to invoke GPT-4o-mini for the single-shot enrichment call
    per response
  repo: openai/openai-python
- name: GPT-4o-mini
  role: LLM that generates each enriched response by weaving structured context into
    the free-text answer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:ced31d386b7558df11dd1ac92d7c35f392cc0292c51367fe2bccb4cb923c339c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - GPT-4o-mini
  - MALTopic
  - openai-python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# llm-prompt-based-survey-response-enrichment

## Summary

Independently enriches each free-text survey response with the respondent's structured fields (e.g., job title, work experience) using a single-shot LLM call, so N input responses yield N enriched responses. This is the enrichment-agent step of the MALTopic multi-agent topic-modeling framework, giving downstream topic modeling more contextual signal.

## When to use

Use when you have a survey dataset with paired free-text answers and structured respondent fields, and you want each response augmented with that context before topic modeling (MALTopic-style enrichment).

## When NOT to use

- The input responses have no paired structured fields (e.g., job title or work experience) to weave in — there is nothing for the enrichment agent to leverage
- Responses are already enriched or the goal is topic extraction/refinement rather than enrichment — use MALTopic's topic-modeling or deduplication agent steps instead
- N is very large and per-response single-shot LLM calls are cost-prohibitive, since the design is one GPT-4o-mini call per response

## Inputs

- Survey dataset of N free-text responses with associated structured fields (job title, work experience)
- Enrichment-prompt template from the MALTopic repo (yash91sharma/MALTopic)

## Outputs

- enriched_responses.csv with exactly one enriched row per input response, each retaining the original answer content plus embedded structured context

## How to apply

Load the N free-text survey responses together with their structured fields (job title, work experience) from the survey dataset. Load the example enrichment-prompt template from the MALTopic GitHub repo (yash91sharma/MALTopic) that specifies how structured context should be woven into a response. For each response, construct an enrichment prompt by combining the free-text answer with that respondent's job title and work experience per the template, then call GPT-4o-mini via the openai-python SDK in single-shot mode — one call per response, no iterative refinement. Write all N enriched responses to enriched_responses.csv in a row-per-response structure aligned with the input records. The rationale, per the paper, is that 'an enrichment agent leverages structured data to enhance textual responses' so topics gain human-readable contextual relevance.

## Related tools

- **MALTopic** (Framework whose enrichment agent performs the prompt-based response enrichment; provides the example enrichment-prompt template) — yash91sharma/MALTopic
- **openai-python** (Python SDK used to invoke GPT-4o-mini for the single-shot enrichment call per response) — openai/openai-python
- **GPT-4o-mini** (LLM that generates each enriched response by weaving structured context into the free-text answer)

## Examples

```
completion = client.chat.completions.create(model="gpt-4o-mini", messages=[{"role": "user", "content": enrichment_prompt}])
```

## Evaluation signals

- enriched_responses.csv exists and contains exactly one enriched row per input response (row count and alignment with input records)
- Each enriched response retains the original free-text answer content
- Each enriched response explicitly incorporates the structured job-title and work-experience context
- Enrichment was single-shot: one GPT-4o-mini call per response, with no iterative refinement passes
- Downstream, MALTopic topic coherence, diversity, and interpretability should improve versus LDA and BERTopic baselines

## Limitations

- One LLM call per response (single-shot mode) means cost and latency scale linearly with N; no iterative refinement is performed to correct weak enrichments
- Enrichment quality depends on the structured fields actually being present and informative; in this research only job title and work experience were used
- The prompt template is the one deposited in the MALTopic repo; results depend on that specific template's instructions for weaving structured context

## Evidence

- [readme] openai-python client access: [evidence span withheld — anchor status: external_doc]
