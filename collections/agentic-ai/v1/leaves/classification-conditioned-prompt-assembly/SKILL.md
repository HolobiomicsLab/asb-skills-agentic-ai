---
name: classification-conditioned-prompt-assembly
description: Use when a zero-shot LVLM must extract entities from multiple heterogeneous
  document types (e.g., 16 certificate types) and prompts must be conditioned on a
  predicted document class rather than fine-tuning.
when_to_use_negative:
- Domain-specific fine-tuning data is available and permissible — LoRA SFT reaches
  93.65% F1 vs. 86.43% zero-shot, so parameter-update routes may be preferable
- Only a single homogeneous document type is involved — the class-conditioned assembly
  adds redundancy-handling complexity without benefit
- You need supervised training prompts or parameter updates — this skill explicitly
  injects knowledge without modifying LVLM parameters
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Qwen2.5-VL-7B
  role: LVLM that consumes the assembled conditional prompt for zero-shot structured
    extraction; called via the app/llm configuration and call interface
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: PaddleOCR
  role: Fine-tuned pre-trained OCR model whose extracted text feeds the training-free
    keyword-based document-type classifier that selects the conditional prompt
- name: PyMuPDF
  role: Extracts embedded document images that are rendered into the LVLM input alongside
    the assembled prompt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:37051ed866c128cbee826a59c2fea3e2fef0c9733e5bc48ce044dc93b12f704b
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
  - PaddleOCR
  - PyMuPDF
  - Qwen2.5-VL-7B
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# classification-conditioned-prompt-assembly

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles a per-document-type conditional prompt P(d̂) for an LVLM by concatenating a shared instruction block, type-specific landmark entity lists, and 2–4 in-context demonstrations, so task-specific knowledge is injected without modifying model parameters.

## When to use

Use when a zero-shot LVLM must extract entities from multiple heterogeneous document types (e.g., 16 certificate types) and prompts must be conditioned on a predicted document class rather than fine-tuning.

## When NOT to use

- Domain-specific fine-tuning data is available and permissible — LoRA SFT reaches 93.65% F1 vs. 86.43% zero-shot, so parameter-update routes may be preferable
- Only a single homogeneous document type is involved — the class-conditioned assembly adds redundancy-handling complexity without benefit
- You need supervised training prompts or parameter updates — this skill explicitly injects knowledge without modifying LVLM parameters

## Inputs

- Configuration file with 16 document-type definitions (type names and landmark entity lists)
- Predicted document type d̂ from the keyword-based classifier run on OCR-extracted text
- 2–4 curated in-context demonstrations per certificate type (input-to-JSON output pairs)
- Preprocessed document image (extracted via PyMuPDF/pdf2image/Pillow and preprocessed with OpenCV)

## Outputs

- Exactly 16 assembled conditional prompt entries keyed by certificate type
- Per-image conditional prompt string P(d̂) combining shared instructions, landmarks, and demonstrations
- Structured JSON predictions with landmark fields, 'year-month' dates, and 'unidentified' placeholders for missing/unreadable values

## How to apply

Load the certificate-type definitions (document type names plus their landmark entity lists) from the configuration file, then build the shared instruction block covering the extraction task definition, document background/purpose cues, layout and spatial hints (top/main/bottom areas), and output rules (pure JSON, standardized 'year-month' dates, exclusive landmark extraction, 'unidentified' placeholders). For each of the 16 predicted document types, inject its predefined landmark list into the prompt template and append 2–4 selected in-context demonstrations showing correct entity mapping, formatting adherence, and missing-value handling. Concatenate shared instructions, type-specific landmarks, and demonstrations into a single string per type, so the LVLM (Qwen2.5-VL-7B) receives only class-relevant context — this acts as implicit conditional computation that reduces task uncertainty and mitigates attention dilution. The prompt assembly module pairs with the training-free keyword-based classifier applied to OCR-extracted text, which supplies the predicted type d̂ that selects which conditional prompt is sent to the LVLM.

## Related tools

- **Qwen2.5-VL-7B** (LVLM that consumes the assembled conditional prompt for zero-shot structured extraction; called via the app/llm configuration and call interface) — https://github.com/FairmeHIT/Multi-VIE
- **PaddleOCR** (Fine-tuned pre-trained OCR model whose extracted text feeds the training-free keyword-based document-type classifier that selects the conditional prompt)
- **PyMuPDF** (Extracts embedded document images that are rendered into the LVLM input alongside the assembled prompt)

## Examples

```
prompts = {dt: SHARED_INSTRUCTION + LANDMARKS[dt] + DEMOS[dt] for dt in cert_types}  # assemble 16 conditional prompts P(d̂)
```

## Evaluation signals

- Each of the 16 generated prompts contains the shared instruction block, the correct type-specific landmark list, and 2–4 demonstrations
- Every prompt enforces JSON-only output with 'unidentified' placeholders for missing or unreadable values
- The output file contains exactly 16 prompt entries keyed by certificate type
- Zero-shot F1 on the 16-type bidding dataset should be near the reported 86.43% (vs. 68.08% supervised baseline) if prompts are assembled correctly
- Normalized edit distance of extracted fields should approach the reported 0.90 for the zero-shot configuration

## Limitations

- Prompt quality depends on the predefined landmark lists for the 16 types; entities outside these lists are excluded by the exclusive-landmark rule
- Zero-shot performance lags the optional LoRA fine-tuned variant (93.65% F1 / 0.93 NED), particularly under seals, watermarks, and low contrast
- No statistical significance testing or variance reporting is given for the reported F1/edit-distance gains

## Evidence
