---
name: entity-level-f1-evaluation-for-document-extraction
description: Use when predictions from a document VIE system (LVLM zero-shot, fine-tuned,
  or OCR&UIE pipeline) must be scored against ground-truth entity annotations, especially
  to quantify degradation on seal-occluded, watermarked, or low-contrast test images
  versus clean samples.
when_to_use_negative:
- The task calls for token-level or character-level OCR accuracy (e.g., CER/WER) rather
  than entity-level extraction fidelity — this metric counts exact string-matched
  entities and edit-distance corrections, not recognition accuracy.
- Evaluating the document-type classification module itself (keyword-based classifier
  over OCR text) — it requires different classification metrics, not entity-level
  F1/NED.
- Public benchmarks with different annotation schemas (e.g., SROIE, FUNSD, POIE, COLD-CELL)
  where the paper's exact-string-match and per-impairment stratification metadata
  are unavailable.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Qwen2.5-VL-7B
  role: LVLM whose zero-shot (86.43% F1 / 0.90 NED) and fine-tuned (93.65% F1 / 0.93
    NED) prediction records are evaluated with this metric suite
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: PaddleOCR
  role: OCR stage of the supervised OCR&UIE baseline whose prediction records are
    compared against the LVLM's under the same F1/NED criteria
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:94c4b9aff1c671b25070f2952664abec840a07b9cfcef5b720939b7dd5f8c8ab
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
  - PaddleOCR
  - Qwen2.5-VL-7B
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# entity-level-f1-evaluation-for-document-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes entity-level precision, recall, F1 and normalized edit distance (NED) for document information-extraction systems (e.g., Qwen2.5-VL-7B LVLM vs. PaddleOCR-based OCR&UIE pipelines), stratified by real-world impairment conditions such as seal occlusion, watermarks, and low contrast.

## When to use

Use when predictions from a document VIE system (LVLM zero-shot, fine-tuned, or OCR&UIE pipeline) must be scored against ground-truth entity annotations, especially to quantify degradation on seal-occluded, watermarked, or low-contrast test images versus clean samples.

## When NOT to use

- The task calls for token-level or character-level OCR accuracy (e.g., CER/WER) rather than entity-level extraction fidelity — this metric counts exact string-matched entities and edit-distance corrections, not recognition accuracy.
- Evaluating the document-type classification module itself (keyword-based classifier over OCR text) — it requires different classification metrics, not entity-level F1/NED.
- Public benchmarks with different annotation schemas (e.g., SROIE, FUNSD, POIE, COLD-CELL) where the paper's exact-string-match and per-impairment stratification metadata are unavailable.

## Inputs

- Ground-truth entity annotations for the real-world bidding dataset (16 certificate types)
- Zero-shot Qwen2.5-VL-7B prediction records
- OCR&UIE (PaddleOCR-based) pipeline prediction records
- Per-sample impairment metadata tagging each test image as seal-occluded, watermarked, low-contrast/distorted, or clean

## Outputs

- Per-impairment-group precision, recall, and entity-level F1 for each method
- Average NED per impairment group (NED = 1 - (insertions + deletions + modifications)/ground-truth-length)
- Degradation table relative to clean samples, with LVLM-vs-OCR&UIE delta per impairment condition

## How to apply

Load ground-truth entity annotations and per-sample impairment metadata tagging each test image as seal-occluded, watermarked, low-contrast/distorted, or clean. Match predicted entities to ground truth using the exact string-match criterion to count true positives, then compute per-sample insertion, deletion, and modification counts. Aggregate precision, recall, and entity-level F1 per impairment group, and compute average NED per group as 1 minus (insertions + deletions + modifications)/ground-truth-length, averaged over entities. Compute each method's degradation relative to the clean-sample group per impairment condition, and tabulate the LVLM-vs-OCR&UIE delta per condition. Validate by checking that unstratified aggregates match the paper's reported overall values (LVLM F1 93.65%/NED 0.93 vs OCR&UIE F1 68.08%/NED 0.67 on Table 5) and that LVLM degradation per impairment condition is smaller than OCR&UIE degradation.

## Related tools

- **Qwen2.5-VL-7B** (LVLM whose zero-shot (86.43% F1 / 0.90 NED) and fine-tuned (93.65% F1 / 0.93 NED) prediction records are evaluated with this metric suite) — https://github.com/FairmeHIT/Multi-VIE
- **PaddleOCR** (OCR stage of the supervised OCR&UIE baseline whose prediction records are compared against the LVLM's under the same F1/NED criteria) — https://github.com/FairmeHIT/Multi-VIE

## Examples

```
ned_per_sample = 1 - (insertions + deletions + modifications) / len(ground_truth_entities); f1 = 2*P*R/(P+R)  # aggregate by impairment group: seal-occluded, watermarked, low-contrast, clean
```

## Evaluation signals

- Unstratified aggregates must reproduce the paper's reported overall values: LVLM F1 93.65% / NED 0.93 vs OCR&UIE F1 68.08% / NED 0.67 (Table 5).
- LVLM degradation relative to the clean-sample group must be smaller than the OCR&UIE degradation for every impairment condition (seals, watermarks, low contrast), reproducing the paper's robustness claim.
- The zero-shot delta over the supervised baseline must equal the reported +18.35 percentage points in F1 (86.43% vs. 68.08%) and 0.23 in NED (0.90 vs. 0.67).
- NED values must lie in [0, 1] and per-sample NED must equal 1 minus (insertions + deletions + modifications)/ground-truth-length for every entity.
- Precision/recall counts must be internally consistent: true positives from exact string matching, and F1 = 2PR/(P+R) per impairment group.

## Limitations

- Exact string-match criterion may undercount semantically equivalent but differently formatted entities, inflating modification counts and lowering NED.
- Robustness conclusions depend on impairment metadata quality; mislabeled seal-occluded or low-contrast samples would skew group-level degradation.
- No statistical significance testing or variance reporting accompanies the reported F1 and NED improvements, so degradation deltas cannot be assessed for uncertainty.
- The fine-tuned variant's training data size and procedure are described only as 'optional' with few details, limiting reproducibility of the 93.65% F1 figure.

## Evidence
