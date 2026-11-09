---
name: few-shot-uie-model-training-per-document-type
description: Use when building a supervised information-extraction baseline over heterogeneous
  document types where labeled data is scarce — e.g. ~10 annotated samples per certificate
  type — and extraction fields differ per document class.
when_to_use_negative:
- When zero or minimal annotation is required — the zero-shot classification-guided
  LVLM pipeline (Qwen2.5-VL-7B with ICL prompts) achieves 86.43% F1 without training
- When the document set contains a single homogeneous document type — per-type UIE
  training is unnecessary overhead
- When target documents are in scripts/layouts far from Chinese bidding certificates,
  since the OCR stage is fine-tuned specifically for Chinese documents
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: PaddleOCR
  role: Pre-trained Chinese OCR model fine-tuned with DB detection and CRNN recognition
    to supply text for UIE training and inference
- name: Doccano
  role: Tool for preparing sequence labels for the UIE task from ~10 annotated samples
    per document type
- name: PaddleLabel
  role: Labeling tool used for the OCR stage annotations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:3f5be3891f418a0d3f110d7b3ad2277d168825ef910e945746edf1909b886b6f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FairmeHIT/Multi-VIE
  tools:
  - Doccano
  - OpenCV
  - PaddleLabel
  - PaddleOCR
  - Pillow
  - PyMuPDF
  - pdf2image
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Few-shot UIE model training per document type

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a separate Unified Information Extraction (UIE) model for each document type in the 16-type bidding dataset, using only ~10 annotated samples per type with sequence labels prepared in Doccano, as the supervised counterpart to the LVLM approach.

## When to use

Use when building a supervised information-extraction baseline over heterogeneous document types where labeled data is scarce — e.g. ~10 annotated samples per certificate type — and extraction fields differ per document class.

## When NOT to use

- When zero or minimal annotation is required — the zero-shot classification-guided LVLM pipeline (Qwen2.5-VL-7B with ICL prompts) achieves 86.43% F1 without training
- When the document set contains a single homogeneous document type — per-type UIE training is unnecessary overhead
- When target documents are in scripts/layouts far from Chinese bidding certificates, since the OCR stage is fine-tuned specifically for Chinese documents

## Inputs

- Preprocessed certificate images extracted from bidding PDFs (PyMuPDF, pdf2image, Pillow, then OpenCV preprocessing)
- OCR-extracted text per image (from the DB+CRNN PaddleOCR pipeline)
- ~10 manually annotated samples per document type as sequence labels prepared in Doccano

## Outputs

- One trained UIE model per document type (16 models)
- Structured landmark-field extractions for the 29,580-image test set
- Per-type and aggregate entity-level F1-score and normalized edit distance (NED), expected ~68.08% F1 / 0.67 NED overall

## How to apply

First run OCR (differentiable binarization DB detection + CRNN recognition, fine-tuned from a pre-trained PaddleOCR model for Chinese documents) over the 98,600-image bidding dataset to obtain text. Route each image to the correct per-type extractor using the keyword-based classifier that matches distinctive terms in OCR-extracted text. For each of the 16 document types, prepare sequence labels in Doccano from ~10 annotated samples and train a separate UIE model to extract that type's landmark fields. At inference, run OCR+UIE on the 29,580-image test set, standardize extracted field values, and score entity-level F1 (exact-match true positives) and average normalized edit distance (NED) per type and overall — expected aggregate baseline is F1 = 68.08% and NED = 0.67 (Table 5).

## Related tools

- **PaddleOCR** (Pre-trained Chinese OCR model fine-tuned with DB detection and CRNN recognition to supply text for UIE training and inference)
- **Doccano** (Tool for preparing sequence labels for the UIE task from ~10 annotated samples per document type)
- **PaddleLabel** (Labeling tool used for the OCR stage annotations)

## Examples

```
python app/two_stage/run_inference.py --ocr_model PaddleOCR-DB-CRNN --uie_models config/per_type_uie/ --test_split 29580 --metrics f1,ned
```

## Evaluation signals

- Aggregate results should match the reported baseline averages of F1 = 68.08% and NED = 0.67 (Table 5)
- Per-type F1 and NED values should fall within a small tolerance of the published OCR&UIE column
- Keyword-based classifier should route each test image to its correct per-type UIE extractor before extraction
- Entity-level F1 must be computed with exact-match true positives and NED averaged per type and overall
- Each of the 16 document types has its own trained UIE model trained on ~10 samples

## Limitations

- The supervised OCR&UIE baseline trails the zero-shot LVLM method by 18.35 F1 points (68.08% vs 86.43%) and 0.23 NED (0.67 vs 0.90)
- Per-type UIE models require manual sequence labeling in Doccano for every document type, limiting scalability to new types
- Performance depends on OCR quality over real-world impairments such as seals, watermarks, and low contrast

## Evidence

- [methods] Train a separate UIE model per image type: [evidence span withheld — anchor status: not_found]
- [methods] Fine-tuned OCR stage: [evidence span withheld — anchor status: not_found]
