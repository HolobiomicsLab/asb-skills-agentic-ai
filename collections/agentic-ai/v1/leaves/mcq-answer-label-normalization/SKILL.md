---
name: mcq-answer-label-normalization
description: Use when raw free-text LLM answers to MCQs must be mapped to scorable
  option labels (A–E), or when building a benchmarking pipeline that scores heterogeneous
  LLM/VLM textual responses.
when_to_use_negative:
- Responses are already structured single-option labels (e.g., clean 'A'–'E' strings)
  requiring no free-text parsing
- The evaluation task does not involve fixed-option MCQ answers (e.g., open-ended
  clinical text or hallucination auditing), since this pipeline only maps free text
  to A–E labels and loses information when responses are judged simply correct or
  incorrect
tools:
- name: GPT-3.5
  role: Extraction model called via API to map each raw textual response to a scorable
    option label
- name: OpenAI Python package
  role: Client for communication with the OpenAI GPT-3.5 extraction calls
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes the API interactions used for structured-output extraction
- name: Python 3.11
  role: Runtime environment for the extraction pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:a1f839fa802aea70d4c1b78dae930e9788c67e6648a5c2cb5a3c36e31f4b66d2
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mcq-answer-label-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A semi-automated pipeline that uses the GPT-3.5 API to extract the selected option (A–E) from raw free-text LLM responses on gastroenterology MCQs, producing a scorable normalized label per response (extracted_options.csv). It converts unstructured model answers into a fixed label space so benchmark accuracy can be computed automatically.

## When to use

Use when raw free-text LLM answers to MCQs must be mapped to scorable option labels (A–E), or when building a benchmarking pipeline that scores heterogeneous LLM/VLM textual responses.

## When NOT to use

- Responses are already structured single-option labels (e.g., clean 'A'–'E' strings) requiring no free-text parsing
- The evaluation task does not involve fixed-option MCQ answers (e.g., open-ended clinical text or hallucination auditing), since this pipeline only maps free text to A–E labels and loses information when responses are judged simply correct or incorrect

## Inputs

- Raw free-text LLM responses to ACG-SA gastroenterology MCQs (from Sdamirsa/LLM-VLM-in-Gastroenterology)
- Prompt/configuration selected during Experiment 0 parameter optimization with GPT-3.5

## Outputs

- extracted_options.csv with one normalized option label (A–E or sentinel NOP/EOP/2OP/error) per LLM response
- Manually validated 100-answer sample review confirming ~98.8% extraction accuracy

## How to apply

Load the raw LLM textual responses (e.g., for the 162 text-only and 138 image-inclusive ACG-SA 2022 questions) from the Sdamirsa/LLM-VLM-in-Gastroenterology repository using Python 3.11. Configure the GPT-3.5 API via the OpenAI Python package with LangChain (0.1.17/0.2.4) for standardized interaction, using structured-output extraction at temperature 0 and max_token 50 so each call deterministically maps one raw response to a single label (A–E) or a sentinel label (NOP, EOP, 2OP, error) for ambiguous responses (no option, end-of-passage, two options). Model settings (function, prompt, temperature, max-token) were pre-optimized on a subset of the dataset with GPT-3.5. Run the extraction over every response, emitting one normalized label per LLM response, then manually review a 100-answer sample against the raw responses; only accept the output file (extracted_options.csv) if extraction accuracy reaches the paper-reported ~98.8%.

## Related tools

- **GPT-3.5** (Extraction model called via API to map each raw textual response to a scorable option label)
- **OpenAI Python package** (Client for communication with the OpenAI GPT-3.5 extraction calls)
- **LangChain (0.1.17 and 0.2.4)** (Standardizes the API interactions used for structured-output extraction)
- **Python 3.11** (Runtime environment for the extraction pipeline)

## Examples

```
from openai import OpenAI; r = OpenAI().chat.completions.create(model='gpt-3.5-turbo', temperature=0, max_tokens=50, messages=[{'role':'user','content': raw_response}])
```

## Evaluation signals

- Manual review of a 100-answer sample of extracted labels against raw responses reaches ~98.8% accuracy before accepting extracted_options.csv
- Every LLM response maps to exactly one label in {A, B, C, D, E, NOP, EOP, 2OP, error}; no response is left unlabeled
- Extraction calls run at temperature 0 and max_token 50, so re-running the pipeline yields deterministic labels for the same input
- Row count of extracted_options.csv equals the number of raw LLM textual responses processed (one label per response)

## Limitations

- The GPT-3.5 configuration was optimized only for OpenAI models and reused for other LLMs; its generalizability to other LLMs is uncertain with only limited validation performed
- Extraction collapses responses to a single option label, causing information loss (e.g., hallucinations may remain even when the correct option is selected)
- Sentinel labels (NOP, EOP, 2OP, error) flag ambiguous responses that require human review and cannot be scored automatically

## Evidence

- [other] Pipeline workflow definition: [evidence span withheld — anchor status: not_found]
- [other] Validation step: [evidence span withheld — anchor status: not_found]
