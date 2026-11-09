---
name: percentage-claim-verification
description: Use when a text (e.g., the RLHF Book) states an approximate percentage
  ('only about 5% of their overall compute in post-training') alongside raw numerator
  and denominator figures that the claim can be recomputed from.
when_to_use_negative:
- The source text gives only the percentage with no raw numerator/denominator figures
  to recompute from
- The two figures being compared come from different units or hardware types (e.g.,
  GPU hours on different accelerators), making the ratio not a like-for-like share
- The claim is qualitative or order-of-magnitude rather than a specific percentage
  that can be reconciled numerically
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Python
  role: Compute the ratio of the two reported GPU-hour figures and reconcile it against
    the stated ~5% claim
- name: NumPy
  role: Hold the reported figures (147,000 and 2,800,000 GPU hours) in an array for
    the division and share computation
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:5738b663f6166bd16dfe50f04ac6da6008b60e61b3fb9c84ccc0591c3be16fff
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# percentage-claim-verification

## Summary

A numerical reconciliation skill that recomputes a stated percentage (e.g., a claimed compute share) from its two reported raw figures and checks whether the quotient is consistent with the claim, quantifying the absolute and relative discrepancy. In this source, it reconciles DeepSeek R1's RL post-training GPU hours against DeepSeek V3 pretraining GPU hours.

## When to use

Use when a text (e.g., the RLHF Book) states an approximate percentage ('only about 5% of their overall compute in post-training') alongside raw numerator and denominator figures that the claim can be recomputed from.

## When NOT to use

- The source text gives only the percentage with no raw numerator/denominator figures to recompute from
- The two figures being compared come from different units or hardware types (e.g., GPU hours on different accelerators), making the ratio not a like-for-like share
- The claim is qualitative or order-of-magnitude rather than a specific percentage that can be reconciled numerically

## Inputs

- Numerator figure: 147,000 H800 GPU hours for DeepSeek R1 RL (post-training) stage
- Denominator figure: 2,800,000 GPU hours for DeepSeek V3 base-model pretraining
- Stated claim string from the source text (~5% of overall compute in post-training)

## Outputs

- Computed ratio (147000/2800000 ≈ 0.0525, i.e., ~5.25%)
- Absolute discrepancy vs. the stated claim (≈0.25 percentage points)
- Results table post_training_compute_ratio.csv with columns: training stage, GPU hours, share of pretraining compute

## How to apply

Load the two reported figures — here 147,000 H800 GPU hours for DeepSeek R1 RL training and 2,800,000 GPU hours for DeepSeek V3 base-model pretraining — into a NumPy array in Python. Compute the ratio of the post-training (RL) figure to the pretraining figure: 147000/2800000 ≈ 0.0525, i.e., ~5.25% of pretraining compute. Compare the recomputed ratio to the stated claim (~5%), reporting the absolute discrepancy in percentage points (≈0.25 pp) and confirming the claim rounds correctly. Persist the result as a CSV table with columns for training stage, GPU hours, and share of pretraining compute so the reconciliation is auditable. The rationale is that stated round-number claims should be reproducible from the underlying raw quantities within a stated tolerance.

## Related tools

- **Python** (Compute the ratio of the two reported GPU-hour figures and reconcile it against the stated ~5% claim)
- **NumPy** (Hold the reported figures (147,000 and 2,800,000 GPU hours) in an array for the division and share computation)

## Examples

```
import numpy as np; hours = np.array([147000, 2800000]); share = hours[0]/hours[1]*100; print(f"R1 RL / V3 pretraining = {share:.2f}%")
```

## Evaluation signals

- Computed ratio 147000/2800000 rounds to ≈5.3% and matches the book's stated 'only about 5%' within a 1-percentage-point tolerance
- Absolute discrepancy between recomputed share (~5.25%) and stated claim (~5%) is ≈0.25 percentage points, within the accepted tolerance
- Output CSV post_training_compute_ratio.csv contains rows for both the R1 RL-training (post-training) stage and the V3 pretraining stage
- CSV columns match the expected schema: training stage, GPU hours, share of pretraining compute
- Reported GPU-hour inputs (147K H800 hours; 2.8M hours) are echoed verbatim in the results table for auditability

## Limitations

- The reconciliation assumes both figures are reported on the same hardware basis (H800 GPU hours); the 147K figure covers R1's RL training stage only and excludes other post-training stages, so the 'share of overall compute' interpretation depends on what denominator the claim refers to
- Round-number claims (e.g., '~5%') carry inherent ambiguity; acceptance is by tolerance (within 1 percentage point) rather than exact equality
- Verification can only check internal consistency of reported numbers, not their underlying accuracy

## Evidence

- [other] Finding card: ratio reconciles with book claim: [evidence span withheld — anchor status: not_found]
- [methods] Workflow card: NumPy-based ratio computation: [evidence span withheld — anchor status: not_found]
