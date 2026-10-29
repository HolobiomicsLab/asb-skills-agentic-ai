---
name: icl-prompt-template-engineering
description: Use when a frozen LVLM (e.g., Qwen2.5-VL-7B) must extract structured
  entities from multiple heterogeneous document types and task-specific knowledge
  has to be injected via conditional, class-conditional prompts instead of fine-tuning.
when_to_use_negative:
- Labeled data and compute for supervised fine-tuning are available and a fine-tuned
  LVLM (e.g., the LoRA SFT variant reaching 93.65% F1) is preferred
- The task involves a single fixed document type where a static prompt without type-conditioned
  landmarks suffices
- Downstream output format is free text rather than the JSON schema with standardized
  'year-month' dates and 'unidentified' placeholders enforced by this prompt design
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Qwen2.5-VL-7B
  role: Target zero-shot LVLM that consumes the assembled conditional prompts P(d̂)
    to generate structured JSON extractions without parameter updates
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:c5dd65549d23121ecb4ad730f16cc41466b11e5bf2f97823a1155b626552d9cc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FairmeHIT/Multi-VIE
  repo_url: https://github.com/FairmeHIT/Multi-VIE
  tools:
  - Qwen2.5-VL-7B
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# icl-prompt-template-engineering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the ICL-based dynamic prompt engineering module of the classification-guided LVLM framework: assemble per-document-type prompts combining a shared instruction block, type-specific landmark entity lists, and 2–4 in-context demonstrations so Qwen2.5-VL-7B can perform zero-shot multi-type visual information extraction without parameter updates.

## When to use

Use when a frozen LVLM (e.g., Qwen2.5-VL-7B) must extract structured entities from multiple heterogeneous document types and task-specific knowledge has to be injected via conditional, class-conditional prompts instead of fine-tuning.

## When NOT to use

- Labeled data and compute for supervised fine-tuning are available and a fine-tuned LVLM (e.g., the LoRA SFT variant reaching 93.65% F1) is preferred
- The task involves a single fixed document type where a static prompt without type-conditioned landmarks suffices
- Downstream output format is free text rather than the JSON schema with standardized 'year-month' dates and 'unidentified' placeholders enforced by this prompt design

## Inputs

- Configuration file with 16 certificate-type definitions (document type names and landmark entity lists)
- Predicted document type d̂ for the input document image
- 2–4 curated in-context demonstrations per certificate type (input–output pairs showing entity mapping, JSON formatting, and 'unidentified' handling)

## Outputs

- Exactly 16 assembled prompt entries keyed by certificate type (conditional prompt P(d̂) per type)
- Per-type prompt string combining shared instruction block, type-specific landmark list, and ICL demonstrations

## How to apply

Load the 16 certificate-type definitions (document type names plus their landmark entity lists) from the configuration file derived from the paper's real-world bidding dataset specification. Build a shared instruction block covering the extraction task definition, general document background/purpose cues, common layout patterns and spatial hints (top/main/bottom areas), and output rules enforcing pure JSON, standardized 'year-month' date fields, exclusive landmark extraction, and 'unidentified' placeholders for missing/unreadable values. For each of the 16 document types, inject its predefined landmark list into the prompt template and append 2–4 carefully selected in-context demonstrations illustrating correct entity mapping, formatting adherence, and missing-value handling. Concatenate shared instructions, type-specific landmarks, and ICL demonstrations into a single conditional prompt P(d̂) keyed by predicted certificate type; this conditioning reduces task uncertainty and attention dilution without modifying model parameters.

## Related tools

- **Qwen2.5-VL-7B** (Target zero-shot LVLM that consumes the assembled conditional prompts P(d̂) to generate structured JSON extractions without parameter updates) — https://github.com/FairmeHIT/Multi-VIE

## Examples

```
prompts = {dtype: build_prompt(dtype, landmarks[dtype], demos[dtype][:4]) for dtype in certificate_types}  # assemble P(d̂) for all 16 types
```

## Evaluation signals

- Each of the 16 generated prompts contains the shared instruction block (task definition, layout/spatial hints, JSON-only output rules)
- Each prompt contains the correct type-specific landmark entity list for its certificate type
- Each prompt includes 2–4 in-context demonstrations per type
- Every prompt enforces JSON-only output with standardized 'year-month' date fields and 'unidentified' placeholders for missing/unreadable values
- The output file contains exactly 16 prompt entries keyed by certificate type

## Limitations

- Zero-shot performance (86.43% F1) trails the optional domain-specific fine-tuned variant (93.65% F1, 0.93 NED), particularly on seals, watermarks, and low-contrast impairments
- Prompt construction is tied to a predefined taxonomy (16 certificate types of the bidding dataset); new document types require new landmark lists and demonstrations
- The paper reports no statistical significance testing or variance for the F1/NED improvements attributed to this prompting strategy

## Evidence
