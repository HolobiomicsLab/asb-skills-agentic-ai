---
name: structured-output-constrained-generation
description: Use when you have raw free-text LLM responses to MCQs and need to convert
  each response into a normalized selected-option label (A–E or a sentinel) for automated
  accuracy scoring.
when_to_use_negative:
- Responses are already structured option labels or a scorable answer key — no extraction
  is needed
- The target model's output format differs so substantially that GPT-3.5-derived settings
  (temperature, max-token, prompt) may not generalize — the paper warns this configuration
  was developed only for OpenAI models
- Free-form clinical reasoning quality, hallucination detection, or documentation
  tasks are the goal — MCQ option extraction only captures correctness, causing information
  loss
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3376
tools:
- name: GPT-3.5
  role: Called via its API on each LLM's textual response to extract the selected
    option as structured output
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes the API interaction with GPT-3.5 for the extraction calls
- name: OpenAI Python package
  role: Performs communication with the OpenAI GPT-3.5 model for option extraction
- name: Python 3.11
  role: Runtime environment for the extraction pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:b942254e78b8c9322749a14fab1ceb31551377fd3c0eb5ce342dd1eeb9334252
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  tools:
  - GPT-3.5
  - LangChain
  - LangChain (0.1.17 and 0.2.4)
  - OpenAI Python package
  - Python
  - Python 3.11
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3376
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# structured-output-constrained-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A semi-automated evaluation pipeline step that uses GPT-3.5 via the OpenAI Python package and LangChain to map each LLM's free-text MCQ response onto a single scorable option label (A–E or a sentinel NOP, EOP, 2OP, error), enabling automated scoring of benchmark answers.

## When to use

Use when you have raw free-text LLM responses to MCQs and need to convert each response into a normalized selected-option label (A–E or a sentinel) for automated accuracy scoring.

## When NOT to use

- Responses are already structured option labels or a scorable answer key — no extraction is needed
- The target model's output format differs so substantially that GPT-3.5-derived settings (temperature, max-token, prompt) may not generalize — the paper warns this configuration was developed only for OpenAI models
- Free-form clinical reasoning quality, hallucination detection, or documentation tasks are the goal — MCQ option extraction only captures correctness, causing information loss

## Inputs

- Raw LLM textual responses to ACG-SA gastroenterology MCQs (from the Sdamirsa/LLM-VLM-in-Gastroenterology repository)
- GPT-3.5 API credentials/config via the OpenAI Python package
- Structured-output extraction schema mapping a response to one label (A–E or sentinel NOP, EOP, 2OP, error)

## Outputs

- extracted_options.csv with one normalized option label (A–E or NOP/EOP/2OP/error sentinel) per LLM response
- Manually reviewed 100-answer validation sample comparing extracted labels to raw responses

## How to apply

Load the raw LLM textual responses for the ACG-SA gastroenterology questions from the Sdamirsa/LLM-VLM-in-Gastroenterology repository using Python 3.11, then configure the GPT-3.5 API via the OpenAI Python package with LangChain (0.1.17/0.2.4) for standardized interaction. Set structured-output extraction with temperature 0 and max_token 50 so each call deterministically maps one raw response to a single label (A–E) or a sentinel label (NOP, EOP, 2OP, error). Run the extraction call over every response, emitting one normalized option label per LLM response. Before accepting the output file (extracted_options.csv), manually review a 100-answer sample of extracted labels against the raw responses and confirm extraction accuracy reaches the paper-reported ~98.8%. Parameter settings (function, prompt, temperature, max-token) were optimized in Experiment 0 using GPT-3.5 on a dataset subset and then reused for other LLMs' responses.

## Related tools

- **GPT-3.5** (Called via its API on each LLM's textual response to extract the selected option as structured output)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes the API interaction with GPT-3.5 for the extraction calls)
- **OpenAI Python package** (Performs communication with the OpenAI GPT-3.5 model for option extraction)
- **Python 3.11** (Runtime environment for the extraction pipeline)

## Examples

```
python extract_options.py --input raw_responses/ --model gpt-3.5-turbo --temperature 0 --max-token 50 --output extracted_options.csv
```

## Evaluation signals

- Manual review of a 100-answer sample of extracted labels against raw responses shows ~98.8% extraction accuracy
- Every LLM response maps to exactly one label: a single option A–E or one sentinel (NOP, EOP, 2OP, error)
- Output file extracted_options.csv contains one row per raw LLM response with a non-empty normalized label
- Extraction calls are deterministic (temperature 0) — repeated runs yield identical labels per response
- Sentinel labels (NOP, EOP, 2OP, error) are reserved for unparseable, no-option, extra-option, or error responses, not valid A–E answers

## Limitations

- Prompt/settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs' responses; per-model prompt configuration may be more effective, and generalizability to other LLMs was only limitedly validated
- Evaluating responses as simply correct/incorrect after extraction causes information loss — even a correct option selection may contain hallucinations
- Extraction and scoring on MCQ accuracy does not capture real-world dynamic interactions, documentation/extraction roles, or hallucinations

## Evidence

- [discussion] Semi-automated response evaluation pipeline using GPT-3.5 extraction with human review: [evidence span withheld — anchor status: not_found]
