---
name: certificate-landmark-schema-definition
description: Use when a multi-type document VIE pipeline needs task-specific entity
  schemas injected into an LVLM via ICL prompts instead of parameter updates, and
  each document type has a known set of landmark entities.
when_to_use_negative:
- When adapting the LVLM itself is acceptable — the paper's alternative is supervised
  fine-tuning via LoRA, which reached 93.65% F1 but requires parameter updates
- When document types or landmark entity sets are unknown and must first be discovered
  — this skill presupposes a fixed 16-type schema; type assignment is handled downstream
  by the classification module
- When a single homogeneous document layout is all that is needed — a static prompt
  without per-type conditioning avoids this skill's 16-way conditional assembly
edam_operation: http://edamontology.org/operation_0292
tools:
- name: Qwen2.5-VL-7B
  role: Target zero-shot LVLM that consumes the assembled conditional prompts P(d̂)
    without any parameter updates
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:fba2f4a354c359c5d8c221b250c2e550981be2097d9ef5d4212c4359481b8526
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# certificate-landmark-schema-definition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Defines and assembles the 16 per-certificate-type conditional prompts used by the classification-guided LVLM framework: a shared instruction block plus type-specific landmark entity lists and 2–4 in-context demonstrations, enabling parameter-free task injection into Qwen2.5-VL-7B.

## When to use

Use when a multi-type document VIE pipeline needs task-specific entity schemas injected into an LVLM via ICL prompts instead of parameter updates, and each document type has a known set of landmark entities.

## When NOT to use

- When adapting the LVLM itself is acceptable — the paper's alternative is supervised fine-tuning via LoRA, which reached 93.65% F1 but requires parameter updates
- When document types or landmark entity sets are unknown and must first be discovered — this skill presupposes a fixed 16-type schema; type assignment is handled downstream by the classification module
- When a single homogeneous document layout is all that is needed — a static prompt without per-type conditioning avoids this skill's 16-way conditional assembly

## Inputs

- Configuration file with 16 certificate-type definitions (type names and landmark entity lists)
- Shared instruction template (task definition, layout/spatial hints, output rules)
- 2–4 curated in-context demonstration examples per document type

## Outputs

- Exactly 16 assembled conditional prompts P(d̂), one per certificate type, keyed by certificate type in a single output file
- Each prompt containing the shared instruction block, correct type-specific landmark list, 2–4 ICL demonstrations, and JSON-only output rules with 'unidentified' placeholders

## How to apply

Load the 16 certificate-type definitions (document type names plus their landmark entity lists) from the configuration file derived from the bidding dataset specification. Build one shared instruction block covering the extraction task definition, document background/purpose cues, layout and spatial hints (top/main/bottom areas), and output rules enforcing pure JSON output, standardized 'year-month' date fields, exclusive landmark extraction, and 'unidentified' placeholders for missing or unreadable values. For each of the 16 document types, inject its predefined landmark list into the prompt template and append 2–4 carefully selected in-context demonstrations showing correct entity mapping, format adherence, and missing-information handling. Concatenate the shared instruction block, type-specific landmarks, and ICL demonstrations into a single string to form the conditional prompt P(d̂) per type; the rationale is that relevance-aware prompting improves information efficiency and mitigates attention dilution without modifying LVLM parameters.

## Related tools

- **Qwen2.5-VL-7B** (Target zero-shot LVLM that consumes the assembled conditional prompts P(d̂) without any parameter updates) — https://github.com/FairmeHIT/Multi-VIE

## Evaluation signals

- Each of the 16 generated prompts contains the shared instruction block verbatim
- Each prompt contains exactly the correct type-specific landmark entity list for its certificate type
- Each prompt includes 2–4 in-context demonstrations covering entity mapping, formatting, and missing-value handling
- Every prompt enforces JSON-only output with standardized 'year-month' date fields and 'unidentified' placeholders
- The output file contains exactly 16 prompt entries, one keyed per certificate type

## Limitations

- The 16-type schema is derived from the paper's real-world bidding dataset; other domains require redefining certificate types and landmark lists
- Prompt quality depends on hand-curated demonstrations (only 2–4 per type), which are not specified in detail and may not generalize to all layouts
- The zero-shot prompt-based configuration underperforms the fine-tuned variant (86.43% vs. 93.65% F1), particularly under seals, watermarks, and low contrast
- No statistical significance testing or variance reporting was provided for the reported F1-score and edit-distance improvements

## Evidence

- [abstract] employs in-context learning (ICL)-based dynamic prompt engineering to inject task-specific knowledge, enabling robust zero-shot inference across diverse layouts: "employs in-context learning (ICL)-based dynamic prompt engineering to inject task-specific knowledge, enabling robust zero-shot inference across diverse layouts"
- [methods] dynamic prompt assembly: "dynamically assemble a concise, task-specific prompt by combining a shared instruction block with type-specific components"
- [intro] dynamic prompt construction: "We introduce a dynamic prompt construction strategy based on predicted class, which serves as an implicit conditioning mechanism that improves task relevance and reduces context redundancy without"
