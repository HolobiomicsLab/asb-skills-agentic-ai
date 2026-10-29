---
name: survey-free-text-theme-extraction
description: Use when a survey dataset contains free-text open-ended fields (e.g.,
  concerns/excitement responses, N≈202) and you need reproducible, human-readable
  topics to compare against LDA or BERTopic baselines.
when_to_use_negative:
- Input text is short structured labels or numeric fields rather than open-ended free-text
  responses
- No LLM API access is available and the pipeline must run fully offline (MALTopic
  requires GPT-4o-mini via the OpenAI API)
- The goal is closed-vocabulary or seeded topic sets rather than latent themes extracted
  from raw responses
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: MALTopic
  role: Multi-agent LLM topic modeling framework providing the enrichment, topic modeling,
    and deduplication agents
  repo: https://github.com/yash91sharma/MALTopic
- name: openai
  role: Python client used to call GPT-4o-mini-2024-07-18 for the pipeline's LLM agents
    with fixed seed, temperature, and top_p
  repo: https://github.com/openai/openai-python
- name: BERTopic
  role: Baseline topic modeling technique for comparative evaluation on the survey
    free-text fields
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:03fe6d19cedf716f274aa734b3c514d00c6cf9fde509d37f615b5b4fabb44e3a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yash91sharma/MALTopic
  tools:
  - BERTopic
  - MALTopic
  - openai
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# survey-free-text-theme-extraction

## Summary

Extract latent themes from survey free-text fields (e.g., concerns and excitement responses) using the MALTopic multi-agent LLM topic modeling pipeline, which enriches raw responses with structured survey fields before topic extraction and deduplication, and outperforms LDA and BERTopic on coherence, diversity, and inter-topic similarity.

## When to use

Use when a survey dataset contains free-text open-ended fields (e.g., concerns/excitement responses, N≈202) and you need reproducible, human-readable topics to compare against LDA or BERTopic baselines.

## When NOT to use

- Input text is short structured labels or numeric fields rather than open-ended free-text responses
- No LLM API access is available and the pipeline must run fully offline (MALTopic requires GPT-4o-mini via the OpenAI API)
- The goal is closed-vocabulary or seeded topic sets rather than latent themes extracted from raw responses

## Inputs

- Survey free-text responses (e.g., concerns and excitement fields, N=202) loaded via pandas
- Structured survey fields used as enrichment context by the enrichment agent
- Reference topics from Tables I and IV / yash91sharma/MALTopic repository for comparison

## Outputs

- Refined, deduplicated topic list per free-text field
- Side-by-side topic comparison file (topic_comparison.csv)
- Evaluation metrics: word coherence, word diversity, topic similarity, document coverage

## How to apply

Load the free-text survey responses (e.g., concerns and excitement fields, N=202) with pandas. Configure the MALTopic multi-agent pipeline to call GPT-4o-mini-2024-07-18 through the openai Python client, fixing seed=1234, temperature=0.2, and top_p=0.9 for reproducible generation. Run the enrichment agent to enhance textual responses using the structured survey fields, then the topic modeling agent to extract latent themes, and finally the deduplication agent to refine and consolidate the topics. Judge success by comparing refined topics against reference topics (Tables I and IV, yash91sharma/MALTopic repo) and by evaluating word coherence, word diversity, inter-topic similarity, and document coverage against LDA and BERTopic baselines; MALTopic should show increased coherence and diversity with reduced similarity.

## Related tools

- **MALTopic** (Multi-agent LLM topic modeling framework providing the enrichment, topic modeling, and deduplication agents) — https://github.com/yash91sharma/MALTopic
- **openai** (Python client used to call GPT-4o-mini-2024-07-18 for the pipeline's LLM agents with fixed seed, temperature, and top_p) — https://github.com/openai/openai-python
- **BERTopic** (Baseline topic modeling technique for comparative evaluation on the survey free-text fields)

## Examples

```
from openai import OpenAI; client = OpenAI(); completion = client.chat.completions.create(model="gpt-4o-mini-2024-07-18", messages=[{"role": "user", "content": enriched_response}], seed=1234, temperature=0.2, top_p=0.9)
```

## Evaluation signals

- Generated topics achieve higher word coherence and word diversity than both LDA and BERTopic baselines
- Inter-topic similarity is reduced relative to LDA and BERTopic
- Refined topics reproduce the reference topics reported in Tables I and IV / the yash91sharma/MALTopic repository
- topic_comparison.csv is produced with a side-by-side match between generated and reference topics
- Repeated runs with seed=1234, temperature=0.2, top_p=0.9 yield consistent topics

## Limitations

- Requires access to OpenAI models (GPT-4o-mini-2024-07-18) via API, adding cost and network dependence
- Reproducibility depends on fixing generation parameters; temperature/top_p drift changes outputs
- Evaluation was demonstrated on one survey dataset (N=202); transfer to other domains is unverified
- The API key must be kept out of source control (use OPENAI_API_KEY environment variable or .env, per the openai-python README)

## Evidence

- [readme] openai client installation: [evidence span withheld — anchor status: external_doc]
- [readme] API key handling guidance: [evidence span withheld — anchor status: external_doc]
