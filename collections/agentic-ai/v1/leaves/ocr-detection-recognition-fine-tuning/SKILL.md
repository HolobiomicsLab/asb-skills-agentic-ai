---
name: ocr-detection-recognition-fine-tuning
description: Use when reproducing or adapting the supervised OCR&UIE baseline on scanned/embedded
  document images (e.g., Chinese bidding certificates) where per-type extraction quality
  is benchmarked against F1-score and normalized edit distance (NED).
when_to_use_negative:
- The goal is zero-shot or minimally supervised extraction — the paper's classification-guided
  LVLM approach (Qwen2.5-VL-7B) reaches 86.43% F1 / 0.90 NED without task-specific
  training, so use OCR fine-tuning only when a supervised baseline is explicitly required.
- Input images are public benchmarks like SROIE or SCID, which the authors note are
  limited in document variety, layout complexity, and entity types relative to the
  multi-type bidding data this pipeline targets.
- No annotated samples exist at all — the pipeline still requires PaddleLabel/Doccano
  labels for the OCR and per-type UIE stages.
tools:
- name: PaddleOCR
  role: Pre-trained Chinese document OCR model whose DB detection and CRNN recognition
    stages are fine-tuned
- name: PaddleLabel
  role: Annotation tool for labeling images used to fine-tune the OCR stage
- name: Doccano
  role: Sequence-labeling tool for preparing UIE training annotations per document
    type
- name: PyMuPDF
  role: Extraction of embedded document images from source PDFs
- name: pdf2image
  role: Extraction of embedded document images from source PDFs
- name: Pillow
  role: Image handling during embedded-image extraction
- name: OpenCV
  role: Preprocessing of extracted document images before OCR fine-tuning and inference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:30f1423e99bda0cf55fbb29376bd982f2bb5aa76cb913261abeb46538a3f9d6e
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ocr-detection-recognition-fine-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tunes a supervised two-stage OCR pipeline (DB text detection + CRNN recognition) from a pre-trained PaddleOCR model on Chinese document images, as the supervised baseline for visually rich information extraction (VIE) on a real-world bidding dataset with 16 certificate types.

## When to use

Use when reproducing or adapting the supervised OCR&UIE baseline on scanned/embedded document images (e.g., Chinese bidding certificates) where per-type extraction quality is benchmarked against F1-score and normalized edit distance (NED).

## When NOT to use

- The goal is zero-shot or minimally supervised extraction — the paper's classification-guided LVLM approach (Qwen2.5-VL-7B) reaches 86.43% F1 / 0.90 NED without task-specific training, so use OCR fine-tuning only when a supervised baseline is explicitly required.
- Input images are public benchmarks like SROIE or SCID, which the authors note are limited in document variety, layout complexity, and entity types relative to the multi-type bidding data this pipeline targets.
- No annotated samples exist at all — the pipeline still requires PaddleLabel/Doccano labels for the OCR and per-type UIE stages.

## Inputs

- 98,600-image real-world bidding dataset spanning 16 certificate types, with a 7:3 train/test split
- PDF source documents whose embedded images are extracted with PyMuPDF, pdf2image, and Pillow
- PaddleLabel annotations for OCR detection/recognition fine-tuning
- Doccano sequence-labeling annotations (~10 samples per document type) for the UIE stage

## Outputs

- Fine-tuned DB detection and CRNN recognition models (PaddleOCR-based)
- OCR-extracted text per document image, routed by keyword-based classifier to per-type UIE extractors
- Standardized extracted landmark field values for the 29,580-image test set
- Per-type and aggregate entity-level F1-score and normalized edit distance (baseline: F1 = 68.08%, NED = 0.67)

## How to apply

Extract embedded document images from source PDFs with PyMuPDF, pdf2image, and Pillow, then preprocess them with OpenCV. Fine-tune the OCR stage — the differentiable binarization (DB) detection algorithm plus the CRNN recognition algorithm — on top of a pre-trained PaddleOCR model for Chinese documents, using annotations created in the PaddleLabel tool. Feed OCR-extracted text to a keyword-based document-type classifier that routes each image to a per-type UIE extractor, with sequence labels prepared in Doccano using only ~10 annotated samples per type. Run the full OCR+UIE inference over the test split (29,580 images in the paper's 7:3 split of 98,600 images) and standardize extracted field values. Judge success by computing entity-level F1 (exact-match true positives) and average NED per certificate type and overall, expecting approximately F1 = 68.08% and NED = 0.67 on this dataset (Table 5).

## Related tools

- **PaddleOCR** (Pre-trained Chinese document OCR model whose DB detection and CRNN recognition stages are fine-tuned)
- **PaddleLabel** (Annotation tool for labeling images used to fine-tune the OCR stage)
- **Doccano** (Sequence-labeling tool for preparing UIE training annotations per document type)
- **PyMuPDF** (Extraction of embedded document images from source PDFs)
- **pdf2image** (Extraction of embedded document images from source PDFs)
- **Pillow** (Image handling during embedded-image extraction)
- **OpenCV** (Preprocessing of extracted document images before OCR fine-tuning and inference)

## Examples

```
conda create -n multi_vie python==3.10 -y && conda activate multi_vie && pip install -r requirements.txt  # then run the app/two_stage OCR & UIE pipeline
```

## Evaluation signals

- Aggregate test-set metrics match the published baseline averages of F1 = 68.08% and NED = 0.67 (Table 5).
- Per-certificate-type F1 and NED values fall within a small tolerance of the published OCR&UIE column.
- Entity-level F1 is computed with exact-match true positives over standardized field values, so any formatting drift in standardization shows up as F1 loss.
- The OCR stage demonstrably benefits from fine-tuning over the pre-trained PaddleOCR model (accuracy 'enhanced by fine-tuning'), not just off-the-shelf inference.
- Pipeline runs end-to-end from the repo's app/two_stage (OCR & UIE pipeline) module within the documented conda environment.

## Limitations

- The supervised baseline requires labeled data (PaddleLabel for OCR, Doccano for UIE) and layout-specific training, limiting scalability versus the zero-shot LVLM approach that outperforms it by 18.35 F1 points.
- No statistical significance testing or variance reporting accompanies the reported F1/NED comparisons.
- No public availability or access conditions are stated for the real-world bidding dataset, complicating exact reproduction.

## Evidence

- [other] The supervised baseline attains 68.08% F1-score and 0.67 normalized edit distance on the bidding dataset: [evidence span withheld — anchor status: not_found]
