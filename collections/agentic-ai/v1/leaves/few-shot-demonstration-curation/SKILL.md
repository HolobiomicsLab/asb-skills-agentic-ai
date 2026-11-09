---
name: few-shot-demonstration-curation
description: Use when performing multi-type visual information extraction with an
  LVLM whose parameters must stay frozen, and per-document-type prompts with entity
  lists and few-shot examples are needed to inject task-specific knowledge.
when_to_use_negative:
- The task requires adapting model weights (use the optional LoRA supervised fine-tuning
  path instead, which reaches 93.65% F1)
- Only a single document type is in scope, so a static prompt without type-conditional
  landmark lists and demonstrations suffices
- The downstream model is not prompt-conditionable (e.g., a supervised OCR & UIE pipeline
  with fixed input schema)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Qwen2.5-VL-7B
  role: Frozen LVLM that consumes the assembled conditional prompts for zero-shot
    structured extraction
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: GPT-4V
  role: Used to generate representative synthetic demonstration examples referenced
    for illustrative purposes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:a5fb869abf9fb089fef5ed37c3427a2a0e3fe08aa26fe9fe8197939f5c47b109
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
  - GPT-4V
  - Qwen2.5-VL-7B
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# few-shot-demonstration-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs ICL-based dynamic prompts for Qwen2.5-VL-7B zero-shot document information extraction by assembling a shared instruction block, type-specific landmark entity lists (16 certificate types), and 2–4 curated in-context demonstrations per type, without any model parameter updates.

## When to use

Use when performing multi-type visual information extraction with an LVLM whose parameters must stay frozen, and per-document-type prompts with entity lists and few-shot examples are needed to inject task-specific knowledge.

## When NOT to use

- The task requires adapting model weights (use the optional LoRA supervised fine-tuning path instead, which reaches 93.65% F1)
- Only a single document type is in scope, so a static prompt without type-conditional landmark lists and demonstrations suffices
- The downstream model is not prompt-conditionable (e.g., a supervised OCR & UIE pipeline with fixed input schema)

## Inputs

- Configuration file with 16 certificate-type definitions (type names and landmark entity lists)
- Predicted document type d̂ from the classification module
- Document image passed to the LVLM
- 2–4 curated in-context demonstration examples per document type

## Outputs

- Exactly 16 assembled prompt entries keyed by certificate type, each containing the shared instruction block, type-specific landmark list, and 2–4 demonstrations
- JSON-only extraction outputs from Qwen2.5-VL-7B with 'unidentified' placeholders for missing/unreadable values

## How to apply

Load the 16 certificate-type definitions (document type names plus their landmark entity lists) from the configuration file derived from the paper's bidding dataset specification. Build a shared instruction block covering the extraction task definition, general document background and purpose cues, common layout patterns and spatial hints (top/main/bottom areas), and output rules enforcing pure JSON output, standardized 'year-month' date fields, exclusive landmark extraction, and 'unidentified' placeholders for missing or unreadable values. For each of the 16 document types, inject the corresponding predefined landmark list into the prompt template and append 2–4 carefully selected in-context demonstrations illustrating correct entity mapping, formatting adherence, and handling of missing information. Concatenate the shared instruction block, type-specific landmarks, and ICL demonstrations into a single conditional prompt P(d̂) per predicted document type; the dynamic construction serves as an implicit conditioning mechanism that improves task relevance and reduces context redundancy so the frozen LVLM can perform zero-shot inference across diverse layouts.

## Related tools

- **Qwen2.5-VL-7B** (Frozen LVLM that consumes the assembled conditional prompts for zero-shot structured extraction) — https://github.com/FairmeHIT/Multi-VIE
- **GPT-4V** (Used to generate representative synthetic demonstration examples referenced for illustrative purposes)

## Examples

```
prompts = {dtype: build_prompt(share_instructions, landmarks[dtype], demos[dtype]) for dtype in cert_types}  # 16 entries keyed by certificate type
```

## Evaluation signals

- Each of the 16 generated prompts contains the shared instruction block and the correct type-specific landmark list
- Every prompt embeds 2–4 in-context demonstrations per document type
- All prompts enforce JSON-only output and include 'unidentified' placeholders for missing/unreadable values
- The output file contains exactly 16 prompt entries keyed by certificate type
- Zero-shot extraction on the 16-type bidding dataset should approach the reported 86.43% F1 and 0.90 normalized edit distance

## Limitations

- General-purpose LVLMs exhibit weaknesses in non-Latin script recognition, table understanding, fine-grained perception, and robustness to perturbations such as seals, watermarks, and low contrast
- Performance depends on the document-type classifier: an incorrect prediction d̂ conditions the LVLM on the wrong landmark list and demonstrations
- Zero-shot prompting underperforms the fine-tuned variant (86.43% vs. 93.65% F1), so heavily impaired documents may require the optional LoRA fine-tuning path

## Evidence
