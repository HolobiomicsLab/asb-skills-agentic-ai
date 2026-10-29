---
name: document-image-preprocessing
description: Use when raw input is PDF or mixed-format bidding/tender documents whose
  embedded certificate images must be extracted and cleaned (seals, watermarks, low
  contrast) before OCR or LVLM inference.
when_to_use_negative:
- Input is already a set of clean raster images rather than PDFs requiring embedded-image
  extraction
- Documents are text-native PDFs where text can be read directly without image rendering
- Public benchmarks with limited document variety (e.g. SROIE, SCID) are used instead
  of the multi-type real-world bidding dataset the pipeline targets
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: PyMuPDF
  role: Extracts embedded images from PDF documents
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: pdf2image
  role: Converts PDF pages to images when embedded extraction is insufficient
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: Pillow
  role: Image loading and manipulation during extraction
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: OpenCV
  role: Preprocessing of extracted images to normalize quality before OCR/LVLM inference
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:00e71d4e711f59e3dce1bb7f83eb2a8685a3156c0008aa441b18730cd81e9f55
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# document-image-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts embedded document images from PDF files and normalizes them with computer-vision preprocessing so that downstream OCR, document-type classification, and LVLM-based visual information extraction operate on clean, uniform inputs.

## When to use

Use when raw input is PDF or mixed-format bidding/tender documents whose embedded certificate images must be extracted and cleaned (seals, watermarks, low contrast) before OCR or LVLM inference.

## When NOT to use

- Input is already a set of clean raster images rather than PDFs requiring embedded-image extraction
- Documents are text-native PDFs where text can be read directly without image rendering
- Public benchmarks with limited document variety (e.g. SROIE, SCID) are used instead of the multi-type real-world bidding dataset the pipeline targets

## Inputs

- PDF documents from a real-world electronic bidding and tendering platform (98,600-image bidding dataset, 16 certificate types)
- Embedded images extracted from PDFs via PyMuPDF, pdf2image, and Pillow

## Outputs

- Preprocessed certificate images (per-page PNG/JPEG) ready for OCR, document-type classification, or LVLM inference
- Images organized in the app/data directory for the Multi-VIE pipeline

## How to apply

Extract embedded images from the source PDFs using PyMuPDF, pdf2image, and Pillow, producing individual page/certificate image files stored under a data directory (e.g. app/data). Then apply OpenCV preprocessing to each image to normalize quality for the downstream pipeline. Feed the preprocessed images either to the supervised two-stage OCR&UIE pipeline (PaddleOCR DB detection + CRNN recognition fine-tuned on Chinese documents, labeled via PaddleLabel) or to the classification-guided LVLM pipeline (Qwen2.5-VL-7B with ICL-based dynamic prompts). Decide the preprocessing aggressiveness based on downstream robustness: the fine-tuned LVLM variant is reported to be robust against seals, watermarks, and low contrast, so heavy denoising is optional there, while the OCR stage benefits from cleaner binarized inputs.

## Related tools

- **PyMuPDF** (Extracts embedded images from PDF documents) — https://github.com/FairmeHIT/Multi-VIE
- **pdf2image** (Converts PDF pages to images when embedded extraction is insufficient) — https://github.com/FairmeHIT/Multi-VIE
- **Pillow** (Image loading and manipulation during extraction) — https://github.com/FairmeHIT/Multi-VIE
- **OpenCV** (Preprocessing of extracted images to normalize quality before OCR/LVLM inference) — https://github.com/FairmeHIT/Multi-VIE

## Examples

```
conda activate multi_vie && pip install -r requirements.txt  # then run app/data_process inference preprocessing on extracted images in app/data
```

## Evaluation signals

- Every source PDF yields extracted/preprocessed images covering the expected 98,600-image corpus with a 7:3 train/test split (29,580-image test set)
- Preprocessed images pass through the downstream OCR stage without detection failures from DB + CRNN, and per-type F1 stays near the published baseline (68.08% F1, 0.67 NED)
- Keyword-based document-type classifier on OCR text of preprocessed images routes images to the correct per-type extractor across the 16 certificate types
- Images are placed under app/data and consumed by app/data_process inference preprocessing as described in the README

## Limitations

- Real-world impairments such as seals, watermarks, and low contrast degrade extraction quality; only the fine-tuned LVLM variant (93.65% F1, 0.93 NED) is claimed robust against them
- No public availability or access conditions are stated for the real-world bidding dataset, limiting reproduction of the preprocessing corpus
- No statistical significance testing or variance reporting accompanies the reported F1/NED improvements, so preprocessing effects cannot be assessed with confidence intervals

## Evidence
