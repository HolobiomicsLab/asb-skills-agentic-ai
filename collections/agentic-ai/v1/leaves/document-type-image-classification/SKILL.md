---
name: document-type-image-classification
description: Use when a multi-type document VIE pipeline must route heterogeneous
  document images (e.g., 16 certificate types from a bidding platform) to per-type
  extraction prompts before running an LVLM.
when_to_use_negative:
- When the corpus contains a single homogeneous document type — the classification
  stage and conditional prompt assembly add no value over a fixed prompt
- When the target type has no reference images indexed in Faiss and no distinctive
  OCR keywords — the training-free classifier has no supervision signal to fall back
  on
- When extensive labeled data is available for a supervised end-to-end model, since
  the paper frames this as a minimal-supervision, training-free alternative
edam_operation: http://edamontology.org/operation_2990
tools:
- name: Faiss
  role: Feature-vector engine used to identify the K most similar feature vectors
    to the input image embedding and support cosine-similarity retrieval
- name: PaddleOCR
  role: Supplies OCR-extracted text for the lightweight keyword-based document-type
    classifier that refines image-retrieval predictions
- name: OpenCV
  role: Image preprocessing of extracted document images (e.g., rotation alignment)
    before classification
- name: Pillow
  role: Extraction of embedded images alongside PyMuPDF and pdf2image, followed by
    OpenCV preprocessing
- name: PyMuPDF
  role: Extraction of embedded document images from source files prior to preprocessing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:9192c7f7e8669b5e6d2093e3bcd4df12107d90816c76590ea62288472ab8246c
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
  - PyMuPDF
  - Qwen2.5-VL-7B
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# document-type-image-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Training-free document-type classification that decouples type prediction from content extraction before LVLM-based visual information extraction, combining image-feature retrieval (Faiss) with a keyword classifier over PaddleOCR text. It reduces task uncertainty so type-specific in-context prompts can be conditionally assembled.

## When to use

Use when a multi-type document VIE pipeline must route heterogeneous document images (e.g., 16 certificate types from a bidding platform) to per-type extraction prompts before running an LVLM.

## When NOT to use

- When the corpus contains a single homogeneous document type — the classification stage and conditional prompt assembly add no value over a fixed prompt
- When the target type has no reference images indexed in Faiss and no distinctive OCR keywords — the training-free classifier has no supervision signal to fall back on
- When extensive labeled data is available for a supervised end-to-end model, since the paper frames this as a minimal-supervision, training-free alternative

## Inputs

- Document images (e.g., certificate scans from a real-world electronic bidding/tendering platform, preprocessed with OpenCV/Pillow after extraction via PyMuPDF or pdf2image)
- Faiss index of pre-computed feature vectors for reference document images
- OCR-extracted text (PaddleOCR) containing type-distinctive keywords
- Per-type keyword lists and reference labels for the 16 certificate types

## Outputs

- Predicted document/certificate type label per input image
- Sorted cosine-similarity scores over top-K retrieved neighbors
- Class prediction used to condition downstream ICL prompt assembly for Qwen2.5-VL-7B extraction

## How to apply

Extract an image feature vector x = M(I) with a pre-trained model M (e.g., ResNet-101, or the ConvNeXt-B ImageNet-22K 224 configuration), then use a Faiss feature-vector engine to retrieve the K nearest reference vectors (K = 5 in the reproduced workflow). Compute cosine similarity between x and each retrieved vector, sort scores in descending order, and apply a similarity threshold (tau = 0.9) with majority voting among top neighbors to predict the certificate type. When image-neighbor agreement is inconclusive, refine with a lightweight, training-free keyword classifier applied to OCR-extracted text (PaddleOCR), matching distinctive terms per type. The predicted class feeds downstream dynamic prompt assembly (shared instruction block plus type-specific landmarks and 2-4 in-context demonstrations) for Qwen2.5-VL-7B inference; misclassification propagates directly into entity-level F1, so gate low-confidence cases toward the keyword fallback.

## Related tools

- **Faiss** (Feature-vector engine used to identify the K most similar feature vectors to the input image embedding and support cosine-similarity retrieval)
- **PaddleOCR** (Supplies OCR-extracted text for the lightweight keyword-based document-type classifier that refines image-retrieval predictions)
- **OpenCV** (Image preprocessing of extracted document images (e.g., rotation alignment) before classification)
- **Pillow** (Extraction of embedded images alongside PyMuPDF and pdf2image, followed by OpenCV preprocessing)
- **PyMuPDF** (Extraction of embedded document images from source files prior to preprocessing)

## Examples

```
import faiss; D, F = Index.search(x, K)  # top-K neighbor search over reference image feature vectors
```

## Evaluation signals

- Aggregate zero-shot VIE result (classification + conditional extraction) reproduces the paper's Table 6 full-framework figures: F1 86.43 and NED 0.9012 (F1 ~86%, NED ~0.90)
- Classification accuracy on the 16 certificate types is high enough that downstream entity-level F1 (exact content match) improves over the 68.08% supervised OCR&UIE baseline
- Similarity-score distributions show top-K neighbors exceeding tau = 0.9 for most inputs, with ambiguous cases correctly routed to the OCR keyword classifier
- Predicted types in app/figure_classification align with the per-type landmark lists used to assemble the ICL prompts (no prompt assembled for an empty/unknown class)
- Pipeline runs end-to-end from app/data images through the figure_classification module without type-prediction failures on seal/watermark/low-contrast scans

## Limitations

- Robustness claims depend on the keyword fallback; the article notes real-world impairments (seals, watermarks, low contrast) challenge recognition and fine-tuning is 'optional' with no detailed training-data size given
- No statistical significance testing or variance is reported for the classification-dependent F1/NED improvements (68.08%→86.43% F1, 0.67→0.90 NED)
- The real-world bidding dataset has no stated public availability, so reference-index construction must be replicated from one's own labeled images

## Evidence

- [methods] Faiss feature-vector retrieval: [evidence span withheld — anchor status: not_found]
- [methods] Thresholding and majority voting: [evidence span withheld — anchor status: not_found]
