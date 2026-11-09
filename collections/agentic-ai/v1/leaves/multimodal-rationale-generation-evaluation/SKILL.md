---
name: multimodal-rationale-generation-evaluation
description: Use when reproducing or benchmarking a two-stage multimodal CoT evaluation
  on ScienceQA/M3CoT, comparing attention-guided perturbed-image (PI) vs black-image
  (BKG) settings, or reporting RougeL rationale quality plus answer accuracy for Base/Large
  model checkpoints.
when_to_use_negative:
- When evaluating single-stage answer-only inference without a rationale-generation
  component (QCM→A only) — this protocol requires the QCM→R and QCMR→A two-stage split
- When the benchmark lacks paired reference rationales for RougeL scoring, since stage-1
  evaluation depends on them
- When the goal is to evaluate large multimodal LLMs rather than small distilled multimodal
  CoT models — the article scopes VSSD to small-model settings and defers broader
  tasks to future work
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: VSSD
  role: Provides the two-stage (QCM→R, QCMR→A) inference pipeline, PI/BKG image settings,
    and attention-guided perturbed-image generation used for evaluation
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:9d75f2bd6c00a3c40a6b0b5b4315a3cef50900c7fa282c62f69a3c2824d685e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - VSSD
  - VSSD code repository (github.com/BGWH123/VSSD)
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multimodal-rationale-generation-evaluation

## Summary

A two-stage evaluation protocol for multimodal chain-of-thought distillation methods like VSSD: stage 1 (QCM→R) generates rationales scored with RougeL, stage 2 (QCMR→A) computes answer inference accuracy, enabling PI (perturbed-image) vs BKG (black-image) baseline comparisons on ScienceQA and M3CoT.

## When to use

Use when reproducing or benchmarking a two-stage multimodal CoT evaluation on ScienceQA/M3CoT, comparing attention-guided perturbed-image (PI) vs black-image (BKG) settings, or reporting RougeL rationale quality plus answer accuracy for Base/Large model checkpoints.

## When NOT to use

- When evaluating single-stage answer-only inference without a rationale-generation component (QCM→A only) — this protocol requires the QCM→R and QCMR→A two-stage split
- When the benchmark lacks paired reference rationales for RougeL scoring, since stage-1 evaluation depends on them
- When the goal is to evaluate large multimodal LLMs rather than small distilled multimodal CoT models — the article scopes VSSD to small-model settings and defers broader tasks to future work

## Inputs

- ScienceQA test split
- M3CoT dataset (for the parallel benchmark)
- Trained VSSD Base and Large model checkpoints
- VSSD code repository (github.com/BGWH123/VSSD) with requirements.txt
- Reference rationales and reference answers for QCM→R and QCMR→A scoring

## Outputs

- Stage-1 RougeL scores comparing PI vs BKG settings per model (e.g. Base 98.11 vs 97.81; Large 98.43 vs 98.13)
- Stage-2 answer inference accuracy overall and per subject (NAT, TXT, SOC) and grade group (G1–6)
- Four-column comparison table (Model, Setting, RougeL, Accuracy) saved as table3_baseline_comparison.csv

## How to apply

Set up the VSSD environment from the repository (pip install -r requirements.txt) and load the ScienceQA test split together with the trained VSSD Base and Large checkpoints. Run stage-1 inference (QCM→R) for each model under both PI and BKG image settings, computing RougeL between generated and reference rationales. Then run stage-2 inference (QCMR→A) under the same settings, computing answer accuracy overall and per subject category (NAT, TXT, SOC) and grade group (G1–6). Assemble the results into a four-column comparison (Model, Setting, RougeL, Accuracy) mirroring Table 3, e.g. VSSD Base PI RougeL 98.11 / accuracy 89.67 vs BKG RougeL 97.81 / accuracy 87.95, and VSSD Large PI RougeL 98.43 / accuracy 93.40 vs BKG RougeL 98.13 / accuracy 90.64. Use the optimal steering hyperparameter α = 0.2, which yields the best overall accuracy on ScienceQA, and β = 0.2 for the distillation loss weighting.

## Related tools

- **VSSD** (Provides the two-stage (QCM→R, QCMR→A) inference pipeline, PI/BKG image settings, and attention-guided perturbed-image generation used for evaluation) — https://github.com/BGWH123/VSSD

## Examples

```
pip install -r requirements.txt
```

## Evaluation signals

- Reproduced PI accuracy must exceed BKG accuracy for both Base and Large on ScienceQA (Base 89.67 vs 87.95; Large 93.40 vs 90.64)
- Reproduced PI RougeL must exceed BKG RougeL for both models (Base 98.11 vs 97.81; Large 98.43 vs 98.13)
- The output CSV contains rows for both Base and Large under PI and BKG, each with RougeL and accuracy fields populated
- Accuracy with α = 0.2 is the best overall on ScienceQA with peak performance across most subcategories such as NAT, TXT, and G1–6; deviations (α ≥ 0.3) degrade performance on SOC
- β is fixed at 0.2, which should balance the main task loss and the visually saliency-guided distillation loss; any other β value should show degradation across NAT, TXT, NO, and G1–6

## Limitations

- Method is limited to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work
- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work
- α calibration is critical: too small (e.g. 0.1) gives an insufficient steering signal, while too large (≥0.3) distorts feature representations and notably degrades performance on complex categories like SOC

## Evidence

- [abstract] Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference.: "Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference."
- [methods] α = 0.2 yields the best overall accuracy on the ScienceQA dataset: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [readme] the hyperparameter α controls the strength with which the steering vector—derived from the feature difference between the original and perturbed images—is injected into the model's intermediate layers: "the hyperparameter α controls the strength with which the steering vector—derived from the feature difference between the original and perturbed images—is injected into the model’s intermediate layers"
- [readme] β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
- [discussion] Future work will extend VSSD to broader multimodal reasoning tasks: "Future work will extend VSSD to broader mul-
timodal reasoning tasks"
