---
name: lvlm-prompted-image-inference
description: Use when you already have a document image and a task-specific prompt
  (shared instruction block plus type-specific landmarks and ICL demonstrations) and
  need to produce validated structured entity field/value pairs from the image.
when_to_use_negative:
- Document type is unknown — this module consumes a conditioned prompt and depends
  on the upstream keyword-based document-type classifier to select type-specific landmarks
  and demonstrations
- Task requires layout-relationship understanding rather than fixed-landmark field
  extraction — the paper reports weak results on layout-heavy benchmarks (e.g., FUNSD
  26.88% F1 / 0.4029 NED)
- A trained supervised OCR & UIE pipeline with abundant labeled data is available
  and preferred — the zero-shot LVLM route is motivated by minimal task-specific training,
  not by beating a strong supervised baseline under full supervision
tools:
- name: Qwen2.5-VL-7B
  role: Zero-shot LVLM that receives the conditioned prompt plus document image and
    generates the raw structured (JSON) prediction
  repo: https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT
- name: Multi-VIE (FairmeHIT/Multi-VIE)
  role: Repository hosting the LVLM configuration and call interface (app/llm) and
    inference preprocessing (app/data_process) used in this module
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:f82e1349b5f2f876e8be43202a1de441924c3edbfd2a4566854e151e43dabced
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FairmeHIT/Multi-VIE
  tools:
  - Multi-VIE (FairmeHIT/Multi-VIE)
  - Qwen2.5-VL-7B
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# lvlm-prompted-image-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs a zero-shot LVLM (Qwen2.5-VL-7B) over a document image together with a classification-conditioned prompt to generate structured entity records as pure JSON, then applies standardized post-processing (date normalization, punctuation/casing unification, repetition-artifact removal) and schema validation before scoring.

## When to use

Use when you already have a document image and a task-specific prompt (shared instruction block plus type-specific landmarks and ICL demonstrations) and need to produce validated structured entity field/value pairs from the image.

## When NOT to use

- Document type is unknown — this module consumes a conditioned prompt and depends on the upstream keyword-based document-type classifier to select type-specific landmarks and demonstrations
- Task requires layout-relationship understanding rather than fixed-landmark field extraction — the paper reports weak results on layout-heavy benchmarks (e.g., FUNSD 26.88% F1 / 0.4029 NED)
- A trained supervised OCR & UIE pipeline with abundant labeled data is available and preferred — the zero-shot LVLM route is motivated by minimal task-specific training, not by beating a strong supervised baseline under full supervision

## Inputs

- Document image (e.g., certificate scan extracted via PyMuPDF/pdf2image/Pillow and preprocessed with OpenCV)
- Classification-conditioned prompt: shared instruction block + type-specific landmarks + in-context demonstrations (pure-JSON instruction, 'year-month' date convention, 'unidentified' placeholder rule)

## Outputs

- Raw LVLM text output intended as JSON
- Parsed entity-level field name / value pairs (structured entity records)
- Post-processed and validated JSON containing exactly the requested landmark fields, with dates in 'year-month' and 'unidentified' for missing/unreadable entities

## How to apply

Assemble the classification-conditioned prompt from the upstream prompt-engineering module (shared instruction block plus type-specific landmarks and in-context demonstrations) and feed it with the document image to Qwen2.5-VL-7B, instructing the model to emit pure JSON with date fields in 'year-month' format, only the specified landmarks, and 'unidentified' for missing or unreadable values. Parse the raw model output as JSON to obtain entity-level field name / value pairs. Standardize post-processing: normalize date fields to 'year-month', unify punctuation and capitalization, and remove repetition artifacts (degenerate repeated-token outputs) from generated values. Finally validate that the output is pure JSON containing exactly the requested landmark fields, with 'unidentified' placeholders and conformant date fields; the zero-shot pipeline's benchmark acceptance level was entity-level F1 of 86.43% and NED of 0.9012.

## Related tools

- **Qwen2.5-VL-7B** (Zero-shot LVLM that receives the conditioned prompt plus document image and generates the raw structured (JSON) prediction) — https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT
- **Multi-VIE (FairmeHIT/Multi-VIE)** (Repository hosting the LVLM configuration and call interface (app/llm) and inference preprocessing (app/data_process) used in this module) — https://github.com/FairmeHIT/Multi-VIE

## Evaluation signals

- Output parses as valid pure JSON containing exactly the landmark fields requested in the conditioned prompt (no extra or missing fields)
- All missing/unreadable entity values are filled with the literal string 'unidentified' rather than dropped or hallucinated
- Date-typed fields conform to the 'year-month' convention after post-processing
- Generated values contain no degenerate repeated-token (repetition-artifact) strings
- Entity-level F1 and NED on the benchmark fall in the reported zero-shot range (F1 86.43%, NED 0.9012 for the framework)

## Limitations

- The module is described as an architectural component rather than an independently validated stage; acceptance is at pipeline level
- Zero-shot performance degrades on some benchmarks (e.g., COLD-SIBR 75.12%/0.8839, FUNSD 26.88%/0.4029) compared to larger/fine-tuned LVLMs
- Robustness against seals, watermarks, and low contrast improves to 93.65% F1 / 0.93 NED only with optional LoRA fine-tuning, not in the zero-shot configuration

## Evidence
