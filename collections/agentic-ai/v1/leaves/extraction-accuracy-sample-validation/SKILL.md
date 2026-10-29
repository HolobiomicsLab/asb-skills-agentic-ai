---
name: extraction-accuracy-sample-validation
description: Use when free-text LLM responses to MCQs have been mapped to scorable
  options by an automated (GPT-3.5 API) extraction step and you must quantify extraction
  accuracy before scoring the benchmark.
when_to_use_negative:
- The responses are already structured option labels rather than free text — no extraction
  step exists to validate
- You are scoring model correctness against the answer key; this skill validates the
  label-extraction step, not model accuracy itself
- Your extraction pipeline's prompts/settings were optimized for a different LLM than
  GPT-3.5, since the paper notes the configuration was limited to OpenAI models and
  its generalizability is uncertain
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3345
tools:
- name: GPT-3.5
  role: LLM whose API performs the automated option extraction that this validation
    step quality-checks
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes the GPT-3.5 extraction calls producing the labels under validation
- name: OpenAI Python package
  role: Communication channel for the GPT-3.5 API extraction calls
- name: Python 3.11
  role: Language for API interactions and the validation sampling workflow
- name: Streamlit (1.33)
  role: Human evaluation app used to support manual review of extracted answers
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:481e7e8ab557e2d4aa5e64a2100e1e39da45bb7bbcd7fb612829ded5b702c5d9
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
  - Streamlit (1.33)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3345
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# extraction-accuracy-sample-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A semi-automated response-evaluation validation step in which a human manually reviews a 100-answer sample of GPT-3.5-extracted MCQ option labels against the raw LLM textual responses to confirm the extraction pipeline reached ~98.8% accuracy before accepting the output file (extracted_options.csv).

## When to use

Use when free-text LLM responses to MCQs have been mapped to scorable options by an automated (GPT-3.5 API) extraction step and you must quantify extraction accuracy before scoring the benchmark.

## When NOT to use

- The responses are already structured option labels rather than free text — no extraction step exists to validate
- You are scoring model correctness against the answer key; this skill validates the label-extraction step, not model accuracy itself
- Your extraction pipeline's prompts/settings were optimized for a different LLM than GPT-3.5, since the paper notes the configuration was limited to OpenAI models and its generalizability is uncertain

## Inputs

- Raw free-text LLM responses to the 162 text-only and 138 image-inclusive 2022 ACG-SA gastroenterology questions
- extracted_options.csv containing one normalized option label (A-E or sentinel NOP/EOP/2OP/error) per LLM response

## Outputs

- Manual-review annotation of a 100-answer sample (extracted label vs. correct option per raw response)
- Sample-based extraction accuracy estimate (target ~98.8%) used as a go/no-go gate for accepting extracted_options.csv

## How to apply

After running the GPT-3.5 extraction call (via the OpenAI Python package with LangChain 0.1.17/0.2.4, temperature 0, max_token 50) over every raw response from the 2022 ACG-SA gastroenterology questions, draw a fixed sample of 100 extracted labels. A human reviewer compares each sampled label (A-E or a sentinel such as NOP, EOP, 2OP, or error) against the corresponding raw LLM textual response to judge whether the correct option was extracted. Compute the sample's extraction accuracy and accept the resulting extracted_options.csv only if it reaches the paper-reported ~98.8%; otherwise revise the extraction prompt/settings (which were optimized in Experiment 0 using a subset of the dataset and GPT-3.5) and re-run. The authors supported human review with a Streamlit (1.33) app for human evaluation.

## Related tools

- **GPT-3.5** (LLM whose API performs the automated option extraction that this validation step quality-checks)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes the GPT-3.5 extraction calls producing the labels under validation)
- **OpenAI Python package** (Communication channel for the GPT-3.5 API extraction calls)
- **Python 3.11** (Language for API interactions and the validation sampling workflow)
- **Streamlit (1.33)** (Human evaluation app used to support manual review of extracted answers)

## Examples

```
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0, max_tokens=50); label = extraction_chain.invoke(raw_llm_response)  # then manually review 100 sampled labels vs raw responses
```

## Evaluation signals

- A fixed 100-answer sample is reviewed; every sampled extracted label is traceable to one raw LLM response
- Sample extraction accuracy reaches the paper-reported ~98.8% before the output file is accepted
- Every extracted label is a single valid value: one of A-E or a sentinel (NOP, EOP, 2OP, error)
- Sentinel labels (e.g., 2OP for two options selected) are reviewed against raw text to confirm they correctly flag unscorable responses

## Limitations

- Extraction configuration was optimized only for OpenAI (GPT-3.5) models and reused for other LLMs; per-model prompt configuration may be more effective
- Generalizability of the GPT-3.5-derived configuration to other LLMs is uncertain; only limited validation was performed
- Manual validation covers only a 100-answer sample, so errors in unsampled extracted labels are not directly measured

## Evidence

- [discussion] Semi-automated response evaluation pipeline using GPT-3.5 extraction with human review: [evidence span withheld — anchor status: not_found]
