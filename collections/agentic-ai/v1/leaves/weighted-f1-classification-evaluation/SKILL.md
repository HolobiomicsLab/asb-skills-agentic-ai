---
name: weighted-f1-classification-evaluation
description: Use when a multi-class document-type classifier (e.g., the keyword-based
  classifier over OCR-extracted text) must be scored on a real-world bidding/test
  split, and a single sample-weighted headline metric plus per-label diagnostics is
  needed.
when_to_use_negative:
- Evaluating the downstream information-extraction (UIE) stage, where F1-score and
  normalized edit distance over entities — not document-type F1 — are the correct
  metrics
- Comparing against unweighted macro-averaged classification scores; the paper's <2%
  error claim is defined on sample-weighted F1, so macro or micro averaging is not
  comparable
- Test data with no ground-truth type labels, since per-label F1 and the weighted
  average both require reference labels
tools:
- name: PaddleOCR
  role: Produces the OCR text tokens (DB detection + CRNN recognition, fine-tuned
    for Chinese documents) that the keyword-based classifier labels and that the F1
    computation is applied to
- name: OpenCV
  role: Preprocesses the test-split certificate images before OCR text extraction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:a575f01f9899f8de90ae160a81e879b327d580cb002d13b561fbd52197932ec4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FairmeHIT/Multi-VIE
  tools:
  - OpenCV
  - PaddleOCR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# weighted-f1-classification-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a document-type classifier by computing per-label Precision, Recall, and F1 and then a sample-weighted average F1 over the 16 certificate types of the real-world bidding dataset; the classification error rate is derived as ε = 1 − F1_weighted and checked against the paper's <2% claim.

## When to use

Use when a multi-class document-type classifier (e.g., the keyword-based classifier over OCR-extracted text) must be scored on a real-world bidding/test split, and a single sample-weighted headline metric plus per-label diagnostics is needed.

## When NOT to use

- Evaluating the downstream information-extraction (UIE) stage, where F1-score and normalized edit distance over entities — not document-type F1 — are the correct metrics
- Comparing against unweighted macro-averaged classification scores; the paper's <2% error claim is defined on sample-weighted F1, so macro or micro averaging is not comparable
- Test data with no ground-truth type labels, since per-label F1 and the weighted average both require reference labels

## Inputs

- Ground-truth document-type labels for the test split of the real-world bidding dataset (16 certificate types)
- Predicted document types from the keyword-based classifier applied to PaddleOCR (DB detection + CRNN recognition) text
- Per-label support counts (N_t^l + N_n^l) for weighting the per-label F1 scores

## Outputs

- Per-label Precision, Recall, and F1 table for the 16 document types
- Weighted-average F1 (F1_weighted)
- Classification error rate ε = 1 − F1_weighted, comparable against the Table 2 reference (TEEBC, 98.45% F1)

## How to apply

Run the trained or training-free classifier (here: keyword-indicator matching I(w_j, k_il) between PaddleOCR-recognized tokens and the predefined keyword set K(d_i), predicting via argmax over the 16 types) on the test split and collect predictions against ground truth. Compute per-label Precision, Recall, and F1 for each document type, then form the weighted-average F1 using label weights w^l = (N_t^l + N_n^l) / Σ(N_t^l + N_n^l). Derive the classification error rate as ε = 1 − F1_weighted. Accept the component when F1_weighted ≥ 98% (ε < 2%), consistent with the paper's Table 2 claim (best reference method TEEBC at 98.45% F1). Rationale: weighting by label support reflects real-world class imbalance among the 16 certificate types, so the headline number tracks the fraction of documents routed to the wrong type before prompt construction.

## Related tools

- **PaddleOCR** (Produces the OCR text tokens (DB detection + CRNN recognition, fine-tuned for Chinese documents) that the keyword-based classifier labels and that the F1 computation is applied to)
- **OpenCV** (Preprocesses the test-split certificate images before OCR text extraction)

## Examples

```
eps = 1 - sum((n_l / N) * f1_l for l in types)  # per-label f1_l over 16 doc types; accept if eps < 0.02 (F1_weighted >= 98%)
```

## Evaluation signals

- F1_weighted ≥ 98% (i.e., ε < 2%) reproduces the paper's Table 2 claim for the classification module
- Per-label Precision, Recall, and F1 are reported for each of the 16 document types, not only the aggregate
- Weighted average uses label-support weights w^l = (N_t^l + N_n^l) / Σ(N_t^l + N_n^l), so verifying the weighting scheme matches the formula is a correctness check
- Result is comparable to the Table 2 best reference method TEEBC at 98.45% F1 on the same dataset
- Predictions come from argmax over indicator-function keyword matches against the predefined per-type keyword sets K(d_i)

## Limitations

- The real-world bidding dataset is not publicly released, so the score must be reproduced on synthetic representative examples from the FairmeHIT/Multi-VIE repository rather than originals
- OCR quality is a hidden dependency: recognition errors in the fine-tuned PaddleOCR pipeline propagate directly into keyword matching and depress F1
- With 16 types of unequal support, the weighted F1 is dominated by frequent certificate types and can mask poor performance on rare document types

## Evidence
