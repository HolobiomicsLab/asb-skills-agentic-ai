---
name: cross-method-benchmark-comparison
description: Use when two document-information-extraction pipelines must be compared
  on identical test images that carry per-sample impairment tags (seal-occluded, watermarked,
  low-contrast/distorted, clean) and per-condition robustness deltas are required.
when_to_use_negative:
- The goal is a single method's absolute accuracy with no baseline comparison — use
  a simple metric computation instead
- Test images lack impairment metadata, so no stratification into seal/watermark/low-contrast/clean
  groups is possible
- The comparison target is a public benchmark (SROIE, SCID, FUNSD, EPHOIE, POIE, ICDAR,
  COLD-CELL, COLD-SIBR) rather than the impairment-stratified real-world bidding dataset
  — different evaluation protocol
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Qwen2.5-VL-7B
  role: Zero-shot classification-guided LVLM whose predictions form one arm of the
    comparison
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: PaddleOCR
  role: OCR component of the fine-tuned OCR&UIE supervised baseline pipeline whose
    predictions form the comparison arm
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:7dc609ed1b2e72f4edc68b70359858f60610ed9231280713c075393e167b5746
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-method-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking procedure that evaluates two extraction methods (a zero-shot Qwen2.5-VL-7B classification-guided LVLM and an OCR&UIE pipeline built on fine-tuned PaddleOCR) side-by-side on the same per-sample impairment-stratified test set, computing entity-level F1 and NED per condition and degradations relative to clean samples.

## When to use

Use when two document-information-extraction pipelines must be compared on identical test images that carry per-sample impairment tags (seal-occluded, watermarked, low-contrast/distorted, clean) and per-condition robustness deltas are required.

## When NOT to use

- The goal is a single method's absolute accuracy with no baseline comparison — use a simple metric computation instead
- Test images lack impairment metadata, so no stratification into seal/watermark/low-contrast/clean groups is possible
- The comparison target is a public benchmark (SROIE, SCID, FUNSD, EPHOIE, POIE, ICDAR, COLD-CELL, COLD-SIBR) rather than the impairment-stratified real-world bidding dataset — different evaluation protocol

## Inputs

- Ground-truth entity annotations for the real-world bidding dataset (16 certificate types)
- Zero-shot Qwen2.5-VL-7B prediction records on the test images
- OCR&UIE pipeline prediction records (fine-tuned PaddleOCR)
- Per-sample impairment metadata tagging each test image as seal-occluded, watermarked, low-contrast/distorted, or clean

## Outputs

- Per-impairment-group precision, recall, and entity-level F1 for each method
- Per-group average NED for each method
- Per-condition degradation table relative to the clean-sample group
- LVLM-vs-OCR&UIE robustness delta per impairment condition

## How to apply

Load the ground-truth entity annotations, the LVLM prediction records (Qwen2.5-VL-7B), and the OCR&UIE prediction records (PaddleOCR-based), plus per-sample impairment metadata. Match predicted entities to ground truth with the exact string-match criterion to count true positives, then compute per-sample insertion, deletion, and modification counts. Aggregate precision, recall, and entity-level F1 per impairment group for each method, and compute average NED per group as 1 minus (insertions + deletions + modifications)/ground-truth-length, averaged over entities. Compute each method's degradation relative to the clean-sample group for every impairment condition and tabulate the LVLM-vs-OCR&UIE delta per condition. Validate that unstratified aggregates reproduce the paper's overall values (LVLM F1 93.65%/NED 0.93 vs OCR&UIE F1 68.08%/NED 0.67, Table 5) before trusting the stratified comparison.

## Related tools

- **Qwen2.5-VL-7B** (Zero-shot classification-guided LVLM whose predictions form one arm of the comparison) — https://github.com/FairmeHIT/Multi-VIE
- **PaddleOCR** (OCR component of the fine-tuned OCR&UIE supervised baseline pipeline whose predictions form the comparison arm)

## Examples

```
python app/two_stage/run_eval.py --gt app/data/annotations. --lvlm app/llm/preds. --ocr app/two_stage/preds. --impairments app/data/impairment_meta. --metric f1,ned --stratify seal,watermark,low_contrast,clean
```

## Evaluation signals

- Unstratified aggregates must match the paper's Table 5 values: LVLM F1 93.65%/NED 0.93 vs OCR&UIE F1 68.08%/NED 0.67
- Zero-shot LVLM overall figures must match 86.43% F1 / 0.90 NED vs 68.08% F1 / 0.67 NED (+18.35 pp F1, +0.23 NED)
- LVLM degradation per impairment condition must be smaller than the OCR&UIE degradation for every condition (robustness claim reproduced)
- True positives counted via exact string match must be consistent with per-sample insertion/deletion/modification counts and ground-truth entity lengths
- Every test image must be assigned exactly one impairment group (seal-occluded, watermarked, low-contrast/distorted, or clean)

## Limitations

- No statistical significance testing or variance reporting accompanies the F1-score and edit-distance improvements, so deltas are point estimates
- Per-condition degradation relies on group sample sizes; imbalance across seal/watermark/low-contrast groups could make per-group estimates unstable
- The real-world bidding dataset has no stated public availability or access conditions, limiting third-party reproduction

## Evidence
