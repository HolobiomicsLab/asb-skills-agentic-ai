---
name: entity-record-parsing
description: Use when a document image plus an assembled type-specific prompt must
  be turned into structured entity records (field name/value pairs) via LVLM generation
  followed by deterministic post-processing and validation.
when_to_use_negative:
- The document type has not been predicted yet — this stage depends on the upstream
  keyword-based classifier and dynamic prompt assembly to condition the LVLM.
- The goal is trainable OCR+UIE pipeline extraction with supervised layout-specific
  models (the app/two_stage baseline), not zero-shot LVLM generation.
- Output requirements are free-form text or prose summarization rather than strict
  landmark-field JSON with an 'unidentified' fill convention.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_3577
tools:
- name: Qwen2.5-VL-7B
  role: Zero-shot LVLM that receives the document image and conditioned prompt and
    generates the raw structured JSON prediction
  repo: https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT
- name: FairmeHIT/Multi-VIE
  role: Repository hosting the LVLM configuration and call interface (app/llm) and
    inference preprocessing (app/data_process) used in this stage
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:fea7013882830c2799fc7c44403fd48ade4360a14bb36bf87100bd1b11ca1bc4
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
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# -entity-record-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs an LVLM (Qwen2.5-VL-7B) on a document image together with a classification-conditioned prompt to generate a raw structured prediction, then parses it as JSON and standardizes values (year-month dates, punctuation/case, repeated-token removal) into entity-level field name/value records.

## When to use

Use when a document image plus an assembled type-specific prompt must be turned into structured entity records (field name/value pairs) via LVLM generation followed by deterministic post-processing and validation.

## When NOT to use

- The document type has not been predicted yet — this stage depends on the upstream keyword-based classifier and dynamic prompt assembly to condition the LVLM.
- The goal is trainable OCR+UIE pipeline extraction with supervised layout-specific models (the app/two_stage baseline), not zero-shot LVLM generation.
- Output requirements are free-form text or prose summarization rather than strict landmark-field JSON with an 'unidentified' fill convention.

## Inputs

- Document image (extracted from PDF/bid documents via PyMuPDF, pdf2image, Pillow and preprocessed with OpenCV)
- Classification-conditioned prompt assembled by the upstream ICL prompt-engineering module (shared instruction block + type-specific landmarks + in-context demonstrations)

## Outputs

- Structured entity records: JSON object of entity-level field name/value pairs restricted to the prompt's landmark fields
- Date fields normalized to 'year-month'; missing/unreadable entities filled with 'unidentified'

## How to apply

Feed the document image and the classification-conditioned prompt (shared instruction block plus type-specific landmarks and in-context demonstrations) to Qwen2.5-VL-7B and generate the raw structured prediction; the prompt instructs pure-JSON output with date fields in 'year-month' format, only the specified landmarks, and 'unidentified' for missing or unreadable values. Parse the raw output as JSON to obtain entity-level field name/value pairs. Then standardize: normalize date fields to 'year-month', unify punctuation and capitalization, and remove degenerate repeated-token artifacts from generated values. Finally validate that the output is valid pure JSON containing exactly the requested landmark fields, that missing/unreadable entities are filled with 'unidentified', and that dates conform to the 'year-month' convention. On the real-world bidding benchmark this stage yields entity-level F1 of 86.43% and NED of 0.9012 in the zero-shot configuration, rising to 93.65% F1 / 0.93 NED with optional LoRA fine-tuning.

## Related tools

- **Qwen2.5-VL-7B** (Zero-shot LVLM that receives the document image and conditioned prompt and generates the raw structured JSON prediction) — https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT
- **FairmeHIT/Multi-VIE** (Repository hosting the LVLM configuration and call interface (app/llm) and inference preprocessing (app/data_process) used in this stage) — https://github.com/FairmeHIT/Multi-VIE

## Examples

```
cd app/llm && python infer.py --image doc.png --prompt conditioned_prompt.json --model Qwen2.5-VL-7B --output raw_prediction.json
```

## Evaluation signals

- Output parses as valid pure JSON with exactly the landmark fields requested in the conditioned prompt.
- Every missing or unreadable entity value is the literal string 'unidentified'.
- All date fields conform to the 'year-month' convention after normalization.
- No degenerate repeated-token artifacts remain in generated values after post-processing.
- Pipeline-level acceptance on the benchmark: entity-level F1 ≈ 86.43% and NED ≈ 0.9012 for the zero-shot framework (93.65% F1 / 0.93 NED with fine-tuning).

## Limitations

- The LVLM inference and post-processing module is described as an architectural component rather than an independently validated stage; acceptance is measured at the pipeline level.
- No statistical significance testing or variance reporting is given for the F1/NED improvements, and fine-tuning details (training data size, procedure) are not specified.
- General-purpose LVLMs exhibit limitations in non-Latin script recognition, table understanding, fine-grained perception, and robustness to perturbations such as seals, watermarks, and low contrast.

## Evidence
