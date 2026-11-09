---
name: icl-dynamic-prompt-construction-for-vie
description: Use when extracting structured JSON entities from heterogeneous certificate/document
  images whose type has already been predicted (e.g., 16 bidding certificate classes)
  and a zero-shot LVLM inference prompt must be conditioned on that type.
when_to_use_negative:
- When the document type is unknown or the classification stage has not yet been run
  — prompts are assembled conditionally on the predicted class
- When labeled data and training are available and a supervised/OCR&UIE pipeline is
  preferred; the zero-shot configuration is designed for minimal task-specific training
- When the task involves a single homogeneous document type with a fixed layout —
  the dynamic per-class conditioning offers no benefit over a static prompt
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Qwen2.5-VL-7B
  role: Frozen LVLM that receives the assembled ICL prompt plus image and generates
    structured JSON predictions
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: PaddleOCR
  role: Provides OCR-extracted text for the keyword-based classifier whose prediction
    conditions prompt assembly
- name: Faiss
  role: Feature vector index used to identify the K nearest neighbors for document-type
    prediction that drives dynamic prompt construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:125f95a81fabec9a1435eca9bac155f279aea786fc17b4f247e5dc99456afa9f
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
  - Faiss
  - OpenCV
  - PaddleOCR
  - Pillow
  - Qwen2.5-VL-7B
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# icl-dynamic-prompt-construction-for-vie

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles type-specific in-context learning (ICL) prompts for a frozen LVLM (Qwen2.5-VL-7B) by combining a shared instruction block with the predicted document type's landmark list and 2-4 demonstrations, enabling robust zero-shot multi-type visual information extraction without modifying model parameters.

## When to use

Use when extracting structured JSON entities from heterogeneous certificate/document images whose type has already been predicted (e.g., 16 bidding certificate classes) and a zero-shot LVLM inference prompt must be conditioned on that type.

## When NOT to use

- When the document type is unknown or the classification stage has not yet been run — prompts are assembled conditionally on the predicted class
- When labeled data and training are available and a supervised/OCR&UIE pipeline is preferred; the zero-shot configuration is designed for minimal task-specific training
- When the task involves a single homogeneous document type with a fixed layout — the dynamic per-class conditioning offers no benefit over a static prompt

## Inputs

- Document/certificate image (preprocessed/rotation-aligned)
- Predicted document type from the training-free image classifier (ResNet-101 + Faiss cosine similarity, tau = 0.9, majority voting; PaddleOCR keyword refinement)
- Type-specific landmark list and 2-4 in-context demonstrations per class
- Shared instruction block (task definition, layout hints, JSON output rules)

## Outputs

- Structured JSON entity predictions per document (dates in 'year-month' format, 'unidentified' placeholders for unreadable values)
- Post-processed predictions standardized for date format, punctuation, and capitalization, scored by entity-level F1 and normalized edit distance (NED)

## How to apply

After the document type is predicted (ResNet-101 features searched against a Faiss index with cosine similarity, tau = 0.9 threshold with majority voting, refined by a PaddleOCR keyword classifier), dynamically assemble a concise, task-specific prompt by combining a shared instruction block — task definition, layout/spatial hints, and output rules (pure JSON only, standardized 'year-month' dates, 'unidentified' for missing/unreadable values) — with the predicted class's type-specific landmark list and 2-4 in-context demonstrations. Feed the prompt plus the (rotation-corrected) image to the frozen Qwen2.5-VL-7B model, then post-process the JSON output to standardize dates, punctuation, capitalization and remove repetition artifacts. The rationale is conditional computation: relevance-aware prompting reduces context redundancy and mitigates attention dilution, improving information efficiency without fine-tuning the LVLM.

## Related tools

- **Qwen2.5-VL-7B** (Frozen LVLM that receives the assembled ICL prompt plus image and generates structured JSON predictions) — https://github.com/FairmeHIT/Multi-VIE
- **PaddleOCR** (Provides OCR-extracted text for the keyword-based classifier whose prediction conditions prompt assembly)
- **Faiss** (Feature vector index used to identify the K nearest neighbors for document-type prediction that drives dynamic prompt construction)

## Examples

```
# app/llm: assemble ICL prompt from predicted class, then call frozen LVLM
python -m app.llm --image app/data/cert.jpg --class-label <predicted_type> --n-demos 3 --output predictions.
```

## Evaluation signals

- Zero-shot aggregate result reproduces the paper's full-framework figures: F1 86.43% and NED 0.9012 (Table 6) on the real-world bidding dataset with 16 certificate types
- Per-class improvement over the supervised OCR&UIE baseline of +18.35 F1 points (86.43% vs. 68.08%) and +0.23 NED (0.90 vs. 0.67)
- Model outputs parse as valid JSON only — output rules mandate pure JSON with standardized 'year-month' dates and 'unidentified' placeholders for missing values
- No repetition artifacts remain after post-processing, and date formats, punctuation, and capitalization are standardized
- Prompt length stays concise and type-relevant: only the predicted class's landmark list and 2-4 demonstrations are included, not the full corpus of type-specific components

## Limitations

- General-purpose LVLMs exhibit limitations in non-Latin script recognition, table understanding, fine-grained perception, and robustness to perturbations such as seals, watermarks, and low contrast
- Zero-shot performance (86.43% F1) falls short of the optional fine-tuned variant (93.65% F1, 0.93 NED), so heavily degraded documents may still fail
- No statistical significance testing or variance reporting is provided for the reported F1/NED improvements, and the real-world bidding dataset has no stated public availability

## Evidence
