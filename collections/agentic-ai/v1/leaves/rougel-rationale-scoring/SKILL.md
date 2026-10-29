---
name: rougel-rationale-scoring
description: Use when evaluating a multimodal CoT distillation model's stage-1 rationale
  generation (QCM→R) against human reference rationales on ScienceQA or M3CoT, e.g.,
  comparing PI vs BKG image settings or ablation variants (w/o PI, w/o ILD).
when_to_use_negative:
- Evaluating stage-2 answer inference quality, where answer accuracy (not RougeL)
  is the metric
- Measuring rationale quality in settings without human reference rationales to score
  against
- Comparing models on overall accuracy alone when the research question targets rationale-generation
  fidelity
tools:
- name: VSSD code repository
  role: Provides the two-stage inference pipeline and trained Base/Large checkpoints
    whose generated rationales are scored with RougeL
  repo: https://github.com/BGWH123/VSSD
- name: PyTorch
  role: Framework for training and running the VSSD ablation variants whose rationales
    are scored
- name: Hugging Face Transformers
  role: Used to load the VSSD models during inference before RougeL computation
- name: pandas
  role: Assembles RougeL and accuracy rows into ablation_results.csv and the Table-3
    comparison CSV
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:4feeadc763c80b4a6256c355ce4fa4895b6648c7f3994c95dc8ab9dfb4cba09c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - Hugging Face Transformers
  - PyTorch
  - VSSD code repository
  - VSSD code repository (github.com/BGWH123/VSSD)
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# rougel-rationale-scoring

## Summary

Computes RougeL between machine-generated chain-of-thought rationales and reference rationales to quantify rationale-generation quality in VSSD's two-stage (QCM→R then QCMR→A) evaluation on ScienceQA and M3CoT, complementing answer-accuracy metrics.

## When to use

Use when evaluating a multimodal CoT distillation model's stage-1 rationale generation (QCM→R) against human reference rationales on ScienceQA or M3CoT, e.g., comparing PI vs BKG image settings or ablation variants (w/o PI, w/o ILD).

## When NOT to use

- Evaluating stage-2 answer inference quality, where answer accuracy (not RougeL) is the metric
- Measuring rationale quality in settings without human reference rationales to score against
- Comparing models on overall accuracy alone when the research question targets rationale-generation fidelity

## Inputs

- Generated rationale strings from stage-1 (QCM→R) inference with VSSD checkpoints
- Reference rationale strings from the ScienceQA (or M3CoT) test split
- Model/setting identifiers (VSSD Base or Large; PI or BKG image setting; ablation variant)

## Outputs

- Per-sample and aggregate RougeL scores for rationale generation
- table3_baseline_comparison.csv with columns Model, Setting, RougeL, Accuracy
- ablation_results.csv with RougeL and average accuracy for w/o PI and w/o ILD variants

## How to apply

After running stage-1 inference (QCM→R) with the trained VSSD Base or Large checkpoints, compute RougeL between each generated rationale and its reference rationale on the ScienceQA test split. Report RougeL alongside stage-2 (QCMR→A) answer accuracy, stratified per subject category (NAT, TXT, SOC) and grade group (G1–6), and assemble rows for each model/setting (PI vs BKG) into a comparison CSV such as table3_baseline_comparison.csv. For ablations trained with the optimal hyperparameters α = 0.2 (steering strength) and β = 0.2 (distillation loss weight), compile variant RougeL and average accuracy into ablation_results.csv. RougeL serves as the rationale-quality column: e.g., full VSSD Base under PI reaches RougeL 98.11, while w/o PI and w/o ILD variants drop to 97.74 and 97.33, so score changes signal loss of fine-grained visual-semantic sensitivity captured by attention-guided perturbed images and SVD-extracted steering vectors.

## Related tools

- **VSSD code repository** (Provides the two-stage inference pipeline and trained Base/Large checkpoints whose generated rationales are scored with RougeL) — https://github.com/BGWH123/VSSD
- **PyTorch** (Framework for training and running the VSSD ablation variants whose rationales are scored)
- **Hugging Face Transformers** (Used to load the VSSD models during inference before RougeL computation)
- **pandas** (Assembles RougeL and accuracy rows into ablation_results.csv and the Table-3 comparison CSV)

## Examples

```
from rouge_score import rouge_scorer; s = rouge_scorer.RougeScorer(['rougeL']); scores = [s.score(ref, hyp)['rougeL'].fmeasure for ref, hyp in zip(refs, hyps)]
```

## Evaluation signals

- Reproduced PI vs BKG RougeL should follow the reported ordering: Base 98.11 (PI) vs 97.81 (BKG); Large 98.43 (PI) vs 98.13 (BKG)
- Ablation RougeL should sit below full-model PI RougeL: w/o PI ≈ 97.74 and w/o ILD ≈ 97.33 vs full VSSD Base 98.11
- The output CSV contains rows for both Base and Large under PI and BKG with RougeL and accuracy fields populated
- RougeL is reported jointly with per-subject (NAT, TXT, SOC) and per-grade (G1–6) accuracies so rationale and answer metrics can be cross-checked
- Hyperparameter consistency: scoring runs should correspond to checkpoints trained at α = 0.2 and β = 0.2, the settings reported to yield best ScienceQA accuracy

## Limitations

- RougeL only quantifies rationale-text overlap; it does not measure answer accuracy, which requires the separate QCMR→A stage
- Scores are tied to ScienceQA/M3CoT two-stage evaluation; the paper defers extension to broader multimodal reasoning tasks to future work
- RougeL differences across image settings (PI vs BKG) are small (≈0.3 points), so careful identical decoding settings are needed for a fair comparison

## Evidence

- [abstract] Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference.: "Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference."
- [methods] α = 0.2 yields the best overall accuracy on the ScienceQA dataset: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [readme] β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
- [discussion] VSSD generates attention-guided perturbed image to capture task-sensitive feature directions: "VSSD generates attention-guided perturbed image to capture task-sensitive feature directions"
