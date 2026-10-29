---
name: certificate-image-degradation-grouping
description: Use when evaluation records contain per-sample impairment metadata and
  you must attribute extraction performance drops (F1, NED) to seals, watermarks,
  or low contrast rather than reporting only unstratified aggregates.
when_to_use_negative:
- Input predictions lack per-sample impairment metadata, so grouping is impossible
  and only unstratified aggregate metrics can be computed
- You need cross-document-type accuracy (e.g., COLD-CELL, FUNSD, EPHOIE benchmark
  tables) rather than robustness to impairments within one dataset
- Your evaluation task is document-type classification accuracy, not entity-level
  extraction quality
tools:
- name: Qwen2.5-VL-7B
  role: Zero-shot classification-guided LVLM whose per-sample predictions are stratified
    and scored by impairment group
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: PaddleOCR
  role: OCR stage of the supervised OCR&UIE baseline whose predictions are evaluated
    under the same impairment-group stratification
  repo: https://github.com/FairmeHIT/Multi-VIE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:e72c5ce90525597e32ea2f7373692d081edbae0a94010a3f8c77b7a98d91782a
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Certificate image degradation grouping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies test images from a visually rich document extraction benchmark into impairment groups (seal-occluded, watermarked, low-contrast/distorted, clean) and computes per-group entity-level F1 and NED degradation to quantify robustness of extraction methods (classification-guided LVLM vs. OCR&UIE pipeline).

## When to use

Use when evaluation records contain per-sample impairment metadata and you must attribute extraction performance drops (F1, NED) to seals, watermarks, or low contrast rather than reporting only unstratified aggregates.

## When NOT to use

- Input predictions lack per-sample impairment metadata, so grouping is impossible and only unstratified aggregate metrics can be computed
- You need cross-document-type accuracy (e.g., COLD-CELL, FUNSD, EPHOIE benchmark tables) rather than robustness to impairments within one dataset
- Your evaluation task is document-type classification accuracy, not entity-level extraction quality

## Inputs

- Ground-truth entity annotations for the real-world bidding dataset (16 certificate types)
- Per-sample prediction records from the zero-shot Qwen2.5-VL-7B LVLM configuration
- Per-sample prediction records from the OCR&UIE pipeline (fine-tuned PaddleOCR + UIE)
- Per-sample impairment metadata tagging each test image as seal-occluded, watermarked, low-contrast/distorted, or clean

## Outputs

- Per-impairment-group precision, recall, entity-level F1, and average NED for each method
- Per-condition degradation relative to the clean-sample group for each method
- LVLM-vs-OCR&UIE delta table per impairment condition

## How to apply

Tag each test image with its impairment condition (seal-occluded, watermarked, low-contrast/distorted, or clean), then match predicted entities to ground truth via exact string match to count true positives and per-sample insertions, deletions, and modifications. Aggregate precision, recall, and entity-level F1 per impairment group per method (zero-shot Qwen2.5-VL-7B vs. fine-tuned PaddleOCR-based OCR&UIE pipeline), and compute average NED per group as 1 minus (insertions + deletions + modifications)/ground-truth-length, averaged over entities. Compute each method's degradation relative to its clean-sample group for every impairment condition, and tabulate the LVLM-vs-OCR&UIE delta per condition. The rationale is that unstratified aggregates (e.g., LVLM F1 93.65%/NED 0.93 vs OCR&UIE 68.08%/NED 0.67) can mask condition-specific failure modes; the per-group deltas directly reproduce the paper's robustness claim that LVLM degradation is smaller than OCR&UIE degradation under every impairment.

## Related tools

- **Qwen2.5-VL-7B** (Zero-shot classification-guided LVLM whose per-sample predictions are stratified and scored by impairment group) — https://github.com/FairmeHIT/Multi-VIE
- **PaddleOCR** (OCR stage of the supervised OCR&UIE baseline whose predictions are evaluated under the same impairment-group stratification) — https://github.com/FairmeHIT/Multi-VIE

## Examples

```
groups = ['clean','seal-occluded','watermarked','low-contrast']; f1 = {g: entity_f1(preds[g], gts[g], match='exact') for g in groups}; delta = {g: f1['clean'] - f1[g] for g in groups}
```

## Evaluation signals

- Unstratified aggregates recomputed from the grouped records must match the paper's Table 5 values: LVLM F1 93.65%/NED 0.93 and OCR&UIE F1 68.08%/NED 0.67 on the real-world bidding dataset
- For every impairment condition, the LVLM degradation relative to its clean-sample group must be smaller than the OCR&UIE degradation
- Exact string match must be used consistently for true-positive counting, so insertions + deletions + modifications per sample reconcile with ground-truth entity lengths
- Every test image must resolve to exactly one impairment label (seal-occluded, watermarked, low-contrast/distorted, or clean) with no unassigned samples
- Per-group NED must fall in [0, 1] and equal 1 minus (insertions + deletions + modifications)/ground-truth-length averaged over entities

## Limitations

- The real-world bidding dataset has no stated public availability or access conditions, so the stratified evaluation may not be independently reproducible
- No statistical significance testing or variance reporting accompanies the reported F1/NED improvements, so group-level differences rest on point estimates
- Fine-tuning details (training data size, procedure) are not specified, so the fine-tuned variant's robustness numbers cannot be independently recreated from the paper alone

## Evidence

- [abstract] Optional domain-specific fine-tuning further improves performance to 93.65% F1 and 0.93 NED, demonstrating superior robustness against seals, watermarks, and low contrast: "Optional domain-specific fine-tuning further improves performance to 93.65% F1 and 0.93 NED, demonstrating superior robustness against seals, watermarks, and low contrast"
- [readme] Evaluated on a real-world bidding dataset with 16 certificate types, our zero-shot method (based on Qwen2.5-VL-7B) outperforms a strong supervised baseline: "Evaluated on a real-world bidding dataset with 16 certificate types, our zero-shot method (based on Qwen2.5-VL-7B) outperforms a strong supervised baseline"
- [methods] DatasetQwen2.5-VL-72B F1/NEDQwen2.5-VL-7B F1/NEDOurs F1/NEDCOLD-CELL79.73%/0.9451: "DatasetQwen2.5-VL-72B F1/NEDQwen2.5-VL-7B F1/NEDOurs F1/NEDCOLD-CELL79.73%/0.9451"
