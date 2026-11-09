---
name: lvlm-visual-information-extraction
description: Use when extracting structured entity records (field name/value pairs)
  from heterogeneous document images (e.g., 16 certificate types from a bidding platform)
  without layout-specific training, or when zero-shot VIE with an LVLM is preferred
  over an OCR+UIE pipeline.
when_to_use_negative:
- When a single, fixed document layout with abundant labeled training data is available,
  a supervised OCR+UIE pipeline or layout-specific model may be more appropriate and
  cheaper to serve
- When input documents are not images but structured files (e.g., PDFs with embedded
  text layers) — extract embedded images first with PyMuPDF/pdf2image/Pillow rather
  than applying LVLM inference directly
- When the entity schema requires exact numeric computation or tabular reasoning beyond
  the LVLM's fine-grained perception capabilities (general-purpose LVLMs exhibit limitations
  in table understanding and fine-grained perception)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Qwen2.5-VL-7B
  role: Frozen LVLM that receives the conditioned prompt and document image to generate
    structured JSON predictions; optionally LoRA fine-tuned (fairme/Qwen2.5-VL-7B-SFT)
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: Faiss
  role: Feature vector engine used to identify the K most similar feature vectors
    for document-type classification
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: PaddleOCR
  role: OCR-extracts text for the training-free keyword-based document-type classifier
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: OpenCV
  role: Image preprocessing including automatic rotation to align scans horizontally
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: Pillow
  role: Embedded image extraction from PDFs before preprocessing
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: PyMuPDF
  role: Extracts embedded images from PDF documents prior to preprocessing
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_006
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:3835ee84630931f31a84f165d8a7287070be4ab9f6e286309e789e296e0d558f
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
  - PyMuPDF
  - Qwen2.5-VL-7B
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lvlm-visual-information-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classification-guided visual information extraction (multi-VIE) using a frozen Qwen2.5-VL-7B LVLM: decouple document-type classification from content extraction, then run ICL-based dynamic prompt engineering to produce structured JSON entity records from document images with minimal task-specific training.

## When to use

Use when extracting structured entity records (field name/value pairs) from heterogeneous document images (e.g., 16 certificate types from a bidding platform) without layout-specific training, or when zero-shot VIE with an LVLM is preferred over an OCR+UIE pipeline.

## When NOT to use

- When a single, fixed document layout with abundant labeled training data is available, a supervised OCR+UIE pipeline or layout-specific model may be more appropriate and cheaper to serve
- When input documents are not images but structured files (e.g., PDFs with embedded text layers) — extract embedded images first with PyMuPDF/pdf2image/Pillow rather than applying LVLM inference directly
- When the entity schema requires exact numeric computation or tabular reasoning beyond the LVLM's fine-grained perception capabilities (general-purpose LVLMs exhibit limitations in table understanding and fine-grained perception)

## Inputs

- Document images (e.g., scans of 16 certificate types from a real-world electronic bidding and tendering platform)
- Faiss feature index of reference image embeddings (ResNet-101 feature vectors)
- OCR-extracted text from PaddleOCR for keyword-based type refinement
- Type-specific landmark lists and 2-4 in-context demonstration examples per certificate type
- Ground-truth entity annotations for evaluation

## Outputs

- Predicted document type per image
- Structured JSON entity records (field name / value pairs, with 'unidentified' for missing values and 'year-month' dates)
- Entity-level F1-score and average normalized edit distance (NED) metrics (reported: 86.43% F1 / 0.9012 NED zero-shot; 93.65% F1 / 0.9348 NED fine-tuned)

## How to apply

First classify each document image into one of the 16 certificate types using the training-free pipeline: extract features with a pre-trained ResNet-101 (e.g., ConvNeXt-B ImageNet-22K 224 configuration is used for related classification), search a Faiss index for the top-5 nearest neighbors, compute cosine similarities, apply similarity threshold tau = 0.9 with majority voting, and refine with a keyword-based classifier (PaddleOCR-extracted text matched against distinctive terms per type). Next, dynamically assemble an ICL prompt combining a shared instruction block (task definition, layout/spatial hints, output rules) with type-specific landmark lists and 2-4 in-context demonstrations, conditioning the frozen Qwen2.5-VL-7B model without modifying its parameters. Preprocess images with OpenCV (including automatic rotation to align scans horizontally) before feeding prompt plus image to the LVLM. Parse the raw output as pure JSON, then post-process: normalize date fields to 'year-month', unify punctuation and capitalization, fill missing/unreadable entities with 'unidentified', and remove degenerate repetition artifacts. Score against ground truth using entity-level F1 (exact-match true positives) and average normalized edit distance (NED); optionally fine-tune the LVLM via LoRA SFT on multi-granular annotations to improve robustness against seals, watermarks, and low contrast.

## Related tools

- **Qwen2.5-VL-7B** (Frozen LVLM that receives the conditioned prompt and document image to generate structured JSON predictions; optionally LoRA fine-tuned (fairme/Qwen2.5-VL-7B-SFT)) — https://github.com/FairmeHIT/Multi-VIE
- **Faiss** (Feature vector engine used to identify the K most similar feature vectors for document-type classification) — https://github.com/FairmeHIT/Multi-VIE
- **PaddleOCR** (OCR-extracts text for the training-free keyword-based document-type classifier) — https://github.com/FairmeHIT/Multi-VIE
- **OpenCV** (Image preprocessing including automatic rotation to align scans horizontally) — https://github.com/FairmeHIT/Multi-VIE
- **Pillow** (Embedded image extraction from PDFs before preprocessing) — https://github.com/FairmeHIT/Multi-VIE
- **PyMuPDF** (Extracts embedded images from PDF documents prior to preprocessing) — https://github.com/FairmeHIT/Multi-VIE

## Examples

```
git clone https://github.com/FairmeHIT/Multi-VIE && cd Multi-VIE && conda create -n multi_vie python==3.10 -y && conda activate multi_vie && pip install -r requirements.txt
```

## Evaluation signals

- Zero-shot aggregate metrics on the evaluation set should reach F1 ~86.43 and NED ~0.9012 (Table 6 full-framework figures)
- Fine-tuned (LoRA SFT) metrics should match Table 3 values of 93.65% F1 and 0.9348 NED within a small tolerance
- Every model output parses as valid pure JSON containing exactly the requested landmark fields
- Missing/unreadable entities are filled with 'unidentified' and date fields conform to the 'year-month' convention after post-processing
- Zero-shot F1 exceeds the supervised OCR&UIE baseline (68.08% F1, 0.67 NED) by ~18.35 percentage points on the same test split

## Limitations

- General-purpose LVLMs exhibit limitations in non-Latin script recognition, table understanding, fine-grained perception, and robustness to perturbations
- No statistical significance testing or variance reporting accompanies the reported F1/NED improvements
- No public availability or access conditions are stated for the real-world bidding dataset (98,600 images, 7:3 train/test split)
- Fine-tuning is described as 'optional' in parts of the text; the SFT procedure trains on 394,400 multi-granular annotations with 8 NVIDIA A100 GPUs, at ~2.6 seconds per image inference on the 7B model

## Evidence

- [methods] Similarity threshold and voting in classification: [evidence span withheld — anchor status: not_found]
- [methods] Post-processing and JSON output rules: [evidence span withheld — anchor status: not_found]
