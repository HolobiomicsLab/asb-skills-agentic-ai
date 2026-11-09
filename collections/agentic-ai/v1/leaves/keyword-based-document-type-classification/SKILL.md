---
name: keyword-based-document-type-classification
description: Use when you have OCR text tokens from heterogeneous document images
  (e.g., Chinese bidding certificates) and need a fast, zero-training document-type
  label to condition a downstream extraction or prompt-construction step.
when_to_use_negative:
- When a training signal is available and a learned classifier (e.g., the paper's
  ConvNeXt-B ImageNet-22K pre-trained image classification module) would outperform
  keyword matching on visually distinctive types
- When document types lack distinctive textual keywords (e.g., layout-only or seal/watermark-based
  discrimination), since the classifier depends entirely on token–keyword matches
  in OCR text
- When OCR quality on non-Latin or low-contrast text is poor, because recognition
  errors propagate directly into keyword counts
tools:
- name: PaddleOCR
  role: Fine-tuned pre-trained model (DB detection + CRNN recognition) that produces
    the OCR text tokens consumed by the keyword classifier
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: OpenCV
  role: Image preprocessing before OCR to handle seals, watermarks, and low contrast
    in certificate scans
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:90c1cb758524aa6e71aea6075817892c1befa726e8872ea929c58370e4578bff
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
  - Doccano
  - OpenCV
  - PaddleLabel
  - PaddleOCR
  - Pillow
  - PyMuPDF
  - pdf2image
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# keyword-based-document-type-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A lightweight, training-free document-type classifier that routes each certificate image to one of 16 types by counting indicator-function matches between OCR-extracted tokens and predefined per-type keyword sets, taking the argmax. In the paper this classification stage feeds a dynamic prompt construction strategy for LVLM-based multi-VIE and is separately validated on the real-world bidding dataset.

## When to use

Use when you have OCR text tokens from heterogeneous document images (e.g., Chinese bidding certificates) and need a fast, zero-training document-type label to condition a downstream extraction or prompt-construction step.

## When NOT to use

- When a training signal is available and a learned classifier (e.g., the paper's ConvNeXt-B ImageNet-22K pre-trained image classification module) would outperform keyword matching on visually distinctive types
- When document types lack distinctive textual keywords (e.g., layout-only or seal/watermark-based discrimination), since the classifier depends entirely on token–keyword matches in OCR text
- When OCR quality on non-Latin or low-contrast text is poor, because recognition errors propagate directly into keyword counts

## Inputs

- Test-split certificate images from the real-world bidding dataset (synthetic representative examples from FairmeHIT/Multi-VIE repo, since originals are not released)
- OCR text tokens w_j extracted by fine-tuned PaddleOCR (DB + CRNN)
- Predefined keyword sets K(d_i) for each of the 16 certificate types

## Outputs

- Predicted document type d̂ per image (argmax over 16 types)
- Per-label Precision, Recall, and F1
- Weighted-average F1 and classification error rate ε = 1 − F1_weighted

## How to apply

Preprocess certificate images with OpenCV and extract text tokens using a PaddleOCR pipeline (DB detection + CRNN recognition, pre-trained model fine-tuned for Chinese documents). For each candidate document type d_i, count indicator-function matches I(w_j, k_il) between recognized tokens w_j and the predefined keyword set K(d_i), then predict the type d̂ via argmax over the 16 types. Evaluate per-label Precision, Recall, and F1, then compute the weighted-average F1 using weights w^l = (N_t^l + N_n^l) / Σ(N_t^l + N_n^l). Derive the classification error rate ε = 1 − F1_weighted; the paper reports ε is small (<2%) on the bidding dataset (Table 2, best method TEEBC at 98.45% F1). Rationale: decoupling classification from content extraction reduces task uncertainty and lets each downstream extractor or prompt see only task-relevant landmarks.

## Related tools

- **PaddleOCR** (Fine-tuned pre-trained model (DB detection + CRNN recognition) that produces the OCR text tokens consumed by the keyword classifier) — https://github.com/FairmeHIT/Multi-VIE
- **OpenCV** (Image preprocessing before OCR to handle seals, watermarks, and low contrast in certificate scans) — https://github.com/FairmeHIT/Multi-VIE

## Evaluation signals

- Weighted-average F1 ≥ 98% (equivalently ε < 2%), consistent with the paper's claim 'Empirically, Table 2 shows ε is small (<2%)'
- Per-label Precision, Recall, and F1 computed against ground-truth labels for all 16 certificate types, benchmarked against TEEBC's 98.45% F1 in Table 2
- Every image receives exactly one predicted type from the argmax over the 16 candidate keyword sets
- Indicator-function match counts I(w_j, k_il) are nonzero for the correct type in typical cases; a fully tied or zero-score prediction indicates keyword-set or OCR failures

## Limitations

- The real-world bidding dataset has no stated public availability; validation must rely on synthetic representative examples from the FairmeHIT/Multi-VIE repository
- Requires hand-curated keyword sets K(d_i) per document type; performance degrades if distinctive terms are missing from OCR output (Chinese documents, seals, watermarks, low contrast)
- The classifier is training-free but assumes document types are textually separable; the paper separately reports an image classification module for type identification, suggesting keyword matching alone is not always sufficient

## Evidence
