---
name: llm-sampling-parameter-configuration
description: Use when an LLM-driven scientific pipeline must produce reproducible,
  deterministic-ish outputs across reruns, e.g. topic modeling or text enrichment
  agents whose generated topics are later compared to reference results.
when_to_use_negative:
- Exploratory brainstorming where output diversity is desirable and rerun reproducibility
  is not required
- Tasks using non-OpenAI model backends where the openai-python client's seed/temperature/top_p
  semantics do not apply
- Cases where the analysis has no need to compare generated outputs against fixed
  reference outputs (no reproducibility requirement)
edam_operation: http://edamontology.org/operation_3766
tools:
- name: openai
  role: OpenAI Python client used to set model, seed, temperature, and top_p parameters
    for reproducible LLM generation in the MALTopic agents
  repo: https://github.com/openai/openai-python
- name: MALTopic
  role: Multi-agent topic modeling framework whose enrichment, topic modeling, and
    deduplication agents are invoked under the fixed sampling configuration
  repo: https://github.com/yash91sharma/MALTopic
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/aiiot65859.2025.11105319
    title: 'MALTopic: Multi-Agent LLM Topic Modeling Framework'
  source_content_hash: sha256:688d8f4bf3b5ec058645ee2fb1c40781ae3f95088e5ad6996b2bf33d10492612
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/openai/openai-python
  tools:
  - MALTopic
  - openai
  edam_operation: http://edamontology.org/operation_3766
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# llm-sampling-parameter-configuration

## Summary

Configure OpenAI LLM generation parameters (model, seed, temperature, top_p) to make multi-agent LLM topic modeling runs reproducible, as done in the MALTopic pipeline with GPT-4o-mini-2024-07-18.

## When to use

Use when an LLM-driven scientific pipeline must produce reproducible, deterministic-ish outputs across reruns, e.g. topic modeling or text enrichment agents whose generated topics are later compared to reference results.

## When NOT to use

- Exploratory brainstorming where output diversity is desirable and rerun reproducibility is not required
- Tasks using non-OpenAI model backends where the openai-python client's seed/temperature/top_p semantics do not apply
- Cases where the analysis has no need to compare generated outputs against fixed reference outputs (no reproducibility requirement)

## Inputs

- OpenAI API key (OPENAI_API_KEY environment variable)
- Pipeline agent prompts (enrichment, topic modeling, deduplication) from the MALTopic framework
- Free-text survey responses to be processed under the fixed sampling configuration

## Outputs

- Client configuration object (OpenAI client with model='gpt-4o-mini-2024-07-18', seed=1234, temperature=0.2, top_p=0.9)
- Reproducible LLM generations used as enriched texts, candidate topics, and deduplicated topics

## How to apply

Instantiate the OpenAI Python client and pin the model to GPT-4o-mini-2024-07-18 for all agent calls in the pipeline. Fix seed 1234 to control sampling determinism, set temperature 0.2 to suppress creative variability in generated topics, and set top_p 0.9 to limit the nucleus-sampling tail. Apply the same configuration uniformly to every agent stage (enrichment, topic modeling, deduplication) so that generated topics can be faithfully compared against reference topics such as those in Tables I and IV of the MALTopic paper. Store the API key in an environment variable (OPENAI_API_KEY) rather than source control, as recommended by the openai-python README.

## Related tools

- **openai** (OpenAI Python client used to set model, seed, temperature, and top_p parameters for reproducible LLM generation in the MALTopic agents) — https://github.com/openai/openai-python
- **MALTopic** (Multi-agent topic modeling framework whose enrichment, topic modeling, and deduplication agents are invoked under the fixed sampling configuration) — https://github.com/yash91sharma/MALTopic

## Examples

```
from openai import OpenAI; client = OpenAI(); completion = client.chat.completions.create(model='gpt-4o-mini-2024-07-18', seed=1234, temperature=0.2, top_p=0.9, messages=[{'role':'user','content': prompt}])
```

## Evaluation signals

- All agent calls report model='gpt-4o-mini-2024-07-18', seed=1234, temperature=0.2, top_p=0.9 in the run configuration/logs
- Rerunning the pipeline with the same inputs yields identical or near-identical generated topics
- Generated topics remain directly comparable to the reference topics in Tables I and IV and the yash91sharma/MALTopic repository
- OPENAI_API_KEY is read from the environment and never hard-coded in source control

## Limitations

- Reproducibility via seed is best-effort: LLM provider-side nondeterminism can still cause small variations across reruns
- Low temperature (0.2) and narrow top_p (0.9) reduce topic diversity and may bias generated topics toward high-probability phrasings
- Parameters were tuned for the AI-tools survey dataset (N=202) and may not transfer to other corpora

## Evidence

- [methods] GPT-4o-mini generation parameters: [evidence span withheld — anchor status: not_found]
- [results] reproducibility rationale: [evidence span withheld — anchor status: not_found]
- [readme] README API key guidance: [evidence span withheld — anchor status: external_doc]
