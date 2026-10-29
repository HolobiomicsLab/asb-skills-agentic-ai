---
name: ocr-keyword-based-type-identification
description: Use when scanned certificate/bidding documents must be routed to one
  of a fixed set of document types (e.g., 16 certificate types) before type-conditional
  information extraction, and a training-free classifier on OCR text is preferred
  over a supervised model.
when_to_use_negative:
- When a large amount of labeled data is already available for a supervised OCR+UIE
  baseline (the paper's supervised baseline reached only 68.08% F1 on this dataset,
  but if the document types are already known, this routing step is redundant)
- When the document set consists of a single known document type, so type identification
  adds no value
- When OCR text quality is too poor (seals, watermarks, low contrast) for distinctive
  keywords to be reliably detected; consider the fine-tuned LVLM variant instead
edam_operation: http://edamontology.org/operation_3281
tools:
- name: PaddleOCR
  role: Extracts OCR text from the document image that the keyword-based classifier
    matches against per-type distinctive terms
- name: Faiss
  role: Feature vector engine supporting the upstream image-classification stage whose
    top-5 nearest-neighbor result is refined by the keyword classifier
- name: OpenCV
  role: Preprocessing (including automatic image rotation to align scans horizontally)
    before OCR and classification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:ec9d61c92b83247df3650677c06535284d47bb8ce6ee88ba969bdbbf112466b9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FairmeHIT/Multi-VIE
  tools:
  - Faiss
  - OpenCV
  - PaddleOCR
  - Pillow
  - Qwen2.5-VL-7B
  edam_operation: http://edamontology.org/operation_3281
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ocr-keyword-based-type-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A training-free document-type identification step that runs PaddleOCR over a scanned certificate image and applies a lightweight keyword-based classifier matching distinctive terms per certificate type, used as the refining second stage of the classification-guided LVLM multi-VIE pipeline.

## When to use

Use when scanned certificate/bidding documents must be routed to one of a fixed set of document types (e.g., 16 certificate types) before type-conditional information extraction, and a training-free classifier on OCR text is preferred over a supervised model.

## When NOT to use

- When a large amount of labeled data is already available for a supervised OCR+UIE baseline (the paper's supervised baseline reached only 68.08% F1 on this dataset, but if the document types are already known, this routing step is redundant)
- When the document set consists of a single known document type, so type identification adds no value
- When OCR text quality is too poor (seals, watermarks, low contrast) for distinctive keywords to be reliably detected; consider the fine-tuned LVLM variant instead

## Inputs

- Document image (scan or extracted embedded image, e.g., from PDF via PyMuPDF/pdf2image/Pillow, preprocessed with OpenCV)
- OCR-extracted text from PaddleOCR
- Optional: candidate type distribution from the image classifier (Faiss cosine similarity scores with tau = 0.9 threshold and majority voting)

## Outputs

- Predicted document type label (one of 16 certificate types in the real-world bidding dataset)
- Type label used to dynamically assemble the type-specific ICL prompt for LVLM extraction

## How to apply

First preprocess the scan (automatic rotation to align it horizontally) and run PaddleOCR to extract the text. Apply the keyword-based classifier, which matches distinctive per-type terms against the OCR-extracted text; this stage refines a prior image-based classification stage that extracts features with a pre-trained ResNet, retrieves top-5 nearest neighbors via a Faiss feature vector engine, sorts cosine similarities in descending order, and applies a similarity threshold tau = 0.9 with majority voting. The keyword classifier is the training-free arbiter that resolves the predicted document type. Rationale: decoupling document-type classification from content extraction reduces task uncertainty so the LVLM prompt can be conditioned on the predicted type with type-specific landmarks and 2-4 in-context demonstrations.

## Related tools

- **PaddleOCR** (Extracts OCR text from the document image that the keyword-based classifier matches against per-type distinctive terms)
- **Faiss** (Feature vector engine supporting the upstream image-classification stage whose top-5 nearest-neighbor result is refined by the keyword classifier)
- **OpenCV** (Preprocessing (including automatic image rotation to align scans horizontally) before OCR and classification)

## Evaluation signals

- Downstream end-to-end target: aggregate zero-shot multi-VIE F1 of ~86.43% and NED ~0.90 on the 16-type bidding dataset (Table 6 full-framework figures)
- Predicted type must be a valid member of the 16 certificate-type label set
- When the upstream image classifier's top-5 cosine similarity exceeds tau = 0.9 with a clear majority, the keyword refinement should agree with the image-based vote; persistent disagreement flags either poor OCR quality or a misclassified image
- The pipeline requires no training: verify no fine-tuning or labeled type data was consumed by this stage

## Limitations

- Reliability depends on OCR quality; the dataset exhibits real-world impairments such as seals, watermarks, and low contrast that can degrade PaddleOCR text and defeat keyword matching
- Keyword sets are type-specific and must be maintained per certificate type; new document types require new distinctive terms
- The upstream similarity threshold (tau = 0.9) and top-5 retrieval choices are fixed hyperparameters; no statistical significance testing or variance reporting was provided for the reported F1/NED improvements

## Evidence

- [methods] A similarity threshold tau is applied to filter the sorted similarity scores O: [evidence span withheld — anchor status: not_found]
