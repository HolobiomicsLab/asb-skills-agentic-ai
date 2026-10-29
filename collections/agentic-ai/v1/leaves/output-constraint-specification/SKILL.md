---
name: output-constraint-specification
description: Use when building prompts for an LVLM (e.g., Qwen2.5-VL-7B) that must
  return structured JSON entity extractions keyed by predefined landmarks, without
  post-hoc parsing fixes.
when_to_use_negative:
- The downstream consumer accepts free-form text or can tolerate non-JSON model replies,
  making strict output constraints unnecessary
- The task requires fine-tuned supervised extraction (e.g., the LoRA-based SFT variant
  reaching 93.65% F1) rather than zero-shot prompted constraints
- Entities to extract are open-ended rather than restricted to a predefined landmark
  list per document type
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Qwen2.5-VL-7B
  role: LVLM that receives the constrained prompt and must comply with the JSON-only
    output rules during zero-shot inference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:0ab144056cb23f52c390c72e43f674eb6752e363eec50e3083cd9eceec1a35c6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FairmeHIT/Multi-VIE
  tools:
  - Qwen2.5-VL-7B
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# -output-constraint-specification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Specifies and enforces strict output-format constraints (pure JSON, standardized 'year-month' dates, exclusive landmark extraction, 'unidentified' placeholders) inside the shared instruction block of ICL-based dynamic prompts, so that Qwen2.5-VL-7B zero-shot inference yields machine-parseable structured extractions.

## When to use

Use when building prompts for an LVLM (e.g., Qwen2.5-VL-7B) that must return structured JSON entity extractions keyed by predefined landmarks, without post-hoc parsing fixes.

## When NOT to use

- The downstream consumer accepts free-form text or can tolerate non-JSON model replies, making strict output constraints unnecessary
- The task requires fine-tuned supervised extraction (e.g., the LoRA-based SFT variant reaching 93.65% F1) rather than zero-shot prompted constraints
- Entities to extract are open-ended rather than restricted to a predefined landmark list per document type

## Inputs

- 16 certificate-type definitions (document type names and landmark entity lists) from the configuration file
- Shared instruction block template for the extraction task
- 2–4 curated in-context demonstrations per document type showing correct entity mapping and formatting

## Outputs

- 16 assembled conditional prompts P(d̂), each embedding JSON-only output rules and 'unidentified' placeholder conventions
- Structured JSON entity extractions (with 'year-month' dates and 'unidentified' placeholders) from Qwen2.5-VL-7B inference

## How to apply

Within the shared instruction block of the conditional prompt P(d̂), write explicit output rules: (1) the model must emit pure JSON only; (2) date fields must be standardized to 'year-month' format; (3) extraction is restricted exclusively to the type-specific landmark entity list injected for the predicted document type (one of the 16 certificate types from the bidding dataset configuration); and (4) missing or unreadable values must be filled with the literal placeholder 'unidentified' rather than omitted or hallucinated. These constraints act as implicit conditional computation, reducing task uncertainty and attention dilution while keeping the LVLM parameter-free. Append 2–4 in-context demonstrations per document type that illustrate correct entity mapping, JSON formatting adherence, and handling of missing information, so the constraints are exemplified as well as stated. Validate that each of the 16 assembled prompts contains the shared instruction block, the correct landmark list, 2–4 demonstrations, and the JSON-only/'unidentified' rules before running inference.

## Related tools

- **Qwen2.5-VL-7B** (LVLM that receives the constrained prompt and must comply with the JSON-only output rules during zero-shot inference)

## Examples

```
python app/llm/infer.py --prompt prompts/bidding_16_types. --model Qwen2.5-VL-7B --output-format
```

## Evaluation signals

- Each of the 16 generated prompts contains the shared instruction block, the correct type-specific landmark list, and 2–4 demonstrations
- Every prompt enforces pure JSON output and 'unidentified' placeholders for missing/unreadable values
- Model outputs parse as valid JSON with date fields in standardized 'year-month' format
- Output file contains exactly 16 prompt entries keyed by certificate type
- Downstream extraction quality matches expected zero-shot range (86.43% F1 / 0.90 NED on the 16-type bidding dataset)

## Limitations

- Constraints apply only to the 16 predefined certificate types of the bidding dataset; other domains require new landmark lists and demonstrations
- No statistical significance testing or variance reporting accompanies the reported F1/NED improvements, so constraint effectiveness varies across runs
- Robustness is challenged by seals, watermarks, and low contrast even with output constraints in place

## Evidence

- [other] output rules enforcing pure JSON format, standardized 'year-month' date fields, exclusive landmark extraction, and 'unidentified' placeholders for missing/unreadable values: [evidence span withheld — anchor status: not_found]
