---
name: classification-error-propagation-analysis
description: Use when a multi-stage document-understanding pipeline must quantify
  how errors in an upstream document-type classifier (e.g., keyword matching on OCR
  tokens across 16 certificate types) degrade downstream structured extraction.
when_to_use_negative:
- The goal is measuring extraction quality (entity F1 / normalized edit distance of
  the LVLM stage), not the upstream classification stage — the paper reports these
  metrics separately
- The document corpus does not match the bidding-domain taxonomy (e.g., public benchmarks
  like SROIE or FUNSD have few document types and layouts), so the keyword sets K(d_i)
  and the <2% threshold do not transfer
- Ground-truth document-type labels are already available and the question is purely
  downstream — then propagate labels directly instead of re-running the keyword classifier
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PaddleOCR
  role: Fine-tuned pre-trained OCR pipeline (DB detection + CRNN recognition) that
    produces the text tokens the keyword-based classifier matches against K(d_i)
- name: OpenCV
  role: Preprocessing of certificate images before OCR and classification
- name: FairmeHIT/Multi-VIE
  role: Repository supplying synthetic representative certificate images (generated
    via GPT-4V) and code modules, including the image-classification component, for
    reproducing the workflow
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:125aa7df54e0c8328f26b11ceca3913f1c04801f6332609f06f9395ffe94bb5c
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
  - FairmeHIT/Multi-VIE
  - OpenCV
  - PaddleOCR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# classification-error-propagation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the error rate of the document-type classification stage (a lightweight, training-free keyword-based classifier over OCR text) and tracks how misclassified types propagate into downstream LVLM extraction quality on the 16-type bidding dataset.

## When to use

Use when a multi-stage document-understanding pipeline must quantify how errors in an upstream document-type classifier (e.g., keyword matching on OCR tokens across 16 certificate types) degrade downstream structured extraction.

## When NOT to use

- The goal is measuring extraction quality (entity F1 / normalized edit distance of the LVLM stage), not the upstream classification stage — the paper reports these metrics separately
- The document corpus does not match the bidding-domain taxonomy (e.g., public benchmarks like SROIE or FUNSD have few document types and layouts), so the keyword sets K(d_i) and the <2% threshold do not transfer
- Ground-truth document-type labels are already available and the question is purely downstream — then propagate labels directly instead of re-running the keyword classifier

## Inputs

- Test-split certificate images (synthetic representatives from FairmeHIT/Multi-VIE, since the real-world bidding dataset is not public)
- OCR-extracted text tokens from PaddleOCR (DB detection + CRNN recognition)
- Predefined keyword sets K(d_i) for each of the 16 document types
- Ground-truth document-type labels

## Outputs

- Per-label Precision, Recall, and F1 for the 16 certificate types
- Weighted-average F1 (F1_weighted) using label-frequency weights
- Classification error rate ε = 1 − F1_weighted, compared against the Table 2 threshold (<2%)

## How to apply

Preprocess test-split certificate images with OpenCV and extract text tokens using a fine-tuned PaddleOCR pipeline (DB detection + CRNN recognition, pre-trained model fine-tuned for Chinese documents). For each candidate document type d_i, count indicator-function matches I(w_j, k_il) between recognized tokens w_j and the predefined keyword set K(d_i), predicting the type d̂ via argmax over the 16 types. Compare predictions against ground-truth labels to compute per-label Precision, Recall, and F1, then the weighted-average F1 using label-frequency weights w^l = (N_t^l + N_n^l) / Σ(N_t^l + N_n^l). Derive the classification error rate ε = 1 − F1_weighted and benchmark against the Table 2 reference (best method TEEBC at 98.45% F1). Accept the result when F1_weighted ≥ 98% (ε < 2%), consistent with the paper's claim that 'Empirically, Table 2 shows ε is small (<2%)'. Because original bidding documents are not released, use the synthetic representative examples published in the FairmeHIT/Multi-VIE repository.

## Related tools

- **PaddleOCR** (Fine-tuned pre-trained OCR pipeline (DB detection + CRNN recognition) that produces the text tokens the keyword-based classifier matches against K(d_i))
- **OpenCV** (Preprocessing of certificate images before OCR and classification)
- **FairmeHIT/Multi-VIE** (Repository supplying synthetic representative certificate images (generated via GPT-4V) and code modules, including the image-classification component, for reproducing the workflow) — https://github.com/FairmeHIT/Multi-VIE

## Examples

```
eps = 1 - f1_score(y_true, y_pred, average='weighted'); assert eps < 0.02, 'classification error rate exceeds the Table 2 (<2%) threshold'
```

## Evaluation signals

- F1_weighted ≥ 98%, i.e., classification error rate ε < 2%, matching the paper's reported bound
- Weighted F1 is in the same range as the Table 2 best method TEEBC (98.45% F1)
- Every prediction d̂ is exactly one of the 16 defined document types (argmax over the 16-type space)
- Per-label Precision, Recall, and F1 are reported for all 16 types, not only an aggregate score
- Misclassified types can be traced through the ICL prompt stage to quantify their effect on downstream extraction F1/NED

## Limitations

- The real-world bidding dataset has no public availability, so evaluation must rely on synthetic representative examples from the FairmeHIT/Multi-VIE repository rather than originals
- No statistical significance testing or variance reporting accompanies the reported F1 improvements, so single-run comparisons are unverified
- The classifier is training-free and keyword-based, so it depends entirely on OCR token quality; OCR failures on seals, watermarks, or low-contrast documents can silently inflate ε
- Fine-tuning of OCR and downstream components is described only as 'optional' with no details on training data size or procedure

## Evidence
