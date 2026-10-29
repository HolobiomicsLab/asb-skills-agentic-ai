---
name: hallucination-repetition-artifact-filtering
description: Use when raw LVLM output from document-image extraction (e.g., Qwen2.5-VL-7B
  generating structured entity records) must be converted into clean, schema-conformant
  JSON with hallucinated tokens, repetition artifacts, and malformed dates repaired
  before scoring.
when_to_use_negative:
- Inputs are OCR text routed to the lightweight keyword-based document-type classifier
  or to the two-stage OCR & UIE pipeline (app/two_stage) rather than LVLM free-text
  generation
- The upstream prompt was not built by the ICL-based prompt engineering module, so
  the expected landmark schema and 'unidentified'/'year-month' conventions were never
  requested and post-hoc validation would be meaningless
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Qwen2.5-VL-7B
  role: Zero-shot LVLM whose raw JSON predictions are parsed and cleaned by this post-processing
    stage
  repo: https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT
- name: FairmeHIT/Multi-VIE
  role: Repository containing the inference preprocessing (app/data_process) and LVLM
    call interface (app/llm) whose outputs feed this filtering stage
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:867529bbc665dd4cc481fcb1b03fce1973fa6c2410ea03316b82253a2fb0fd2e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FairmeHIT/Multi-VIE
  tools:
  - FairmeHIT/Multi-VIE
  - Qwen2.5-VL-7B
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Hallucination & repetition artifact filtering for LVLM structured output

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Standardized post-processing and validation stage applied to raw Qwen2.5-VL-7B JSON predictions in the classification-guided LVLM framework: parse model output, normalize dates to 'year-month', unify punctuation/capitalization, remove degenerate repeated-token artifacts, and verify landmark completeness with 'unidentified' fills.

## When to use

Use when raw LVLM output from document-image extraction (e.g., Qwen2.5-VL-7B generating structured entity records) must be converted into clean, schema-conformant JSON with hallucinated tokens, repetition artifacts, and malformed dates repaired before scoring.

## When NOT to use

- Inputs are OCR text routed to the lightweight keyword-based document-type classifier or to the two-stage OCR & UIE pipeline (app/two_stage) rather than LVLM free-text generation
- The upstream prompt was not built by the ICL-based prompt engineering module, so the expected landmark schema and 'unidentified'/'year-month' conventions were never requested and post-hoc validation would be meaningless

## Inputs

- Raw generated text from Qwen2.5-VL-7B given a document image and a classification-conditioned prompt
- Prompt-specified landmark field list for the predicted document type

## Outputs

- Valid pure JSON object with exactly the requested landmark fields (entity-level field name/value pairs)
- Date values normalized to 'year-month' format
- Missing/unreadable entities filled with 'unidentified'
- Generated values cleaned of degenerate repeated-token artifacts, punctuation and capitalization unified

## How to apply

Feed the classification-conditioned prompt (shared instruction block plus type-specific landmarks and in-context demonstrations) together with the document image to Qwen2.5-VL-7B and generate the raw structured prediction, which the prompt instructs to be pure JSON with date fields in 'year-month' format and 'unidentified' for missing/unreadable values. Parse the raw model output as JSON to obtain entity-level field name/value pairs, since the model can emit surrounding text or malformed JSON. Standardize by normalizing date fields to 'year-month', unifying punctuation and capitalization, and removing obvious repetition artifacts (degenerate repeated-token outputs) from generated values. Finally validate that the output is valid pure JSON containing exactly the requested landmark fields, that missing/unreadable entities are filled with 'unidentified', and that date fields conform to the 'year-month' convention; the article treats this as an architectural post-processing stage rather than an independently validated module, with pipeline-level acceptance at entity-level F1 of 86.43% and NED of 0.9012 on the zero-shot benchmark.

## Related tools

- **Qwen2.5-VL-7B** (Zero-shot LVLM whose raw JSON predictions are parsed and cleaned by this post-processing stage) — https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT
- **FairmeHIT/Multi-VIE** (Repository containing the inference preprocessing (app/data_process) and LVLM call interface (app/llm) whose outputs feed this filtering stage) — https://github.com/FairmeHIT/Multi-VIE

## Examples

```
pred = .loads(raw_output); {k: (v if isinstance(v, str) and v.strip() else 'unidentified') for k, v in pred.items()}
```

## Evaluation signals

- Output parses as valid pure JSON with no surrounding prose or code fences
- Field set matches exactly the requested landmark fields for the predicted certificate type
- Every missing/unreadable entity is the literal string 'unidentified' rather than an empty value or hallucinated content
- All date fields conform to the 'year-month' convention
- Pipeline-level acceptance on the real-world bidding benchmark reaches entity-level F1 of 86.43% and NED of 0.9012 for the zero-shot framework

## Limitations

- The article describes this module architecturally; it is not independently validated, so artifact-removal correctness can only be judged via downstream entity-level F1/NED
- No statistical significance testing or variance reporting exists for the reported F1-score and edit-distance improvements
- Degenerate repeated-token removal is heuristic ('obvious repetition artifacts') and may miss subtle hallucinations in low-contrast documents degraded by seals and watermarks

## Evidence

- [methods] Feed the prompt and image to the Qwen2.5-VL-7B model (the zero-shot LVLM used in the framework) and generate the raw structured prediction: [evidence span withheld — anchor status: not_found]
- [discussion] as a described architectural component rather than an independently validated stage: [evidence span withheld — anchor status: not_found]
