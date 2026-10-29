---
name: baseline-ablation-comparison-tables
description: Use when a paper reports a two-stage multimodal CoT evaluation comparing
  an ablation setting (e.g., PI vs BKG images) against a baseline across model sizes,
  and the numbers must be reproduced into a comparison table.
when_to_use_negative:
- The evaluation is single-stage (direct QCM→A) with no rationale generation, so RougeL
  cannot be computed.
- The task requires hyperparameter tuning of α or β (α = 0.2 and β = 0.2 are already
  established as optimal in the paper); use a hyperparameter-analysis workflow instead.
- The dataset is M3CoT or another benchmark whose per-category structure (NAT/TXT/SOC,
  G1–6) does not apply.
edam_operation: http://edamontology.org/operation_2422
tools:
- name: VSSD code repository
  role: Provides environment setup (requirements.txt), trained Base/Large checkpoints,
    and the PI vs BKG inference pipeline used to compute RougeL and accuracy for the
    comparison table
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:8859a510c97d5f62bc327410984f9a2fda11829a269f7fbe8578df04867c6878
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - VSSD code repository
  - VSSD code repository (github.com/BGWH123/VSSD)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# baseline-ablation-comparison-tables

## Summary

Reproduce and assemble a controlled baseline-vs-ablation comparison table (e.g., VSSD's Table 3: attention-guided perturbed-image (PI) vs black-image (BKG) settings) reporting two-stage multimodal CoT metrics — rationale RougeL and answer accuracy — for Base and Large model variants on ScienceQA.

## When to use

Use when a paper reports a two-stage multimodal CoT evaluation comparing an ablation setting (e.g., PI vs BKG images) against a baseline across model sizes, and the numbers must be reproduced into a comparison table.

## When NOT to use

- The evaluation is single-stage (direct QCM→A) with no rationale generation, so RougeL cannot be computed.
- The task requires hyperparameter tuning of α or β (α = 0.2 and β = 0.2 are already established as optimal in the paper); use a hyperparameter-analysis workflow instead.
- The dataset is M3CoT or another benchmark whose per-category structure (NAT/TXT/SOC, G1–6) does not apply.

## Inputs

- ScienceQA test split
- Trained VSSD Base and Large checkpoints
- VSSD repository (pip requirements.txt) and PI / BKG image-setting configuration
- Reference rationales for RougeL scoring

## Outputs

- table3_baseline_comparison.csv with rows for Base and Large under PI and BKG, containing Model, Setting, RougeL, and Accuracy columns
- Per-subject (NAT, TXT, SOC) and per-grade-group (G1–6) accuracy breakdowns

## How to apply

Set up the VSSD environment from github.com/BGWH123/VSSD via 'pip install -r requirements.txt', then load the ScienceQA test split and the trained VSSD Base and Large checkpoints. Run stage-1 inference (QCM→R) for each model under both image settings (attention-guided perturbed images, PI, and black-image baseline, BKG), computing RougeL between generated and reference rationales. Run stage-2 inference (QCMR→A) for each model and setting, computing answer accuracy overall and per subject category (NAT, TXT, SOC) and grade group (G1–6). Assemble RougeL and accuracy into a four-column comparison mirroring the paper's table format (Model, Setting, RougeL, Accuracy) and save as CSV; validate that PI consistently outperforms BKG and that results align with the reported optimum of α = 0.2 for best overall ScienceQA accuracy.

## Related tools

- **VSSD code repository** (Provides environment setup (requirements.txt), trained Base/Large checkpoints, and the PI vs BKG inference pipeline used to compute RougeL and accuracy for the comparison table) — https://github.com/BGWH123/VSSD

## Examples

```
pip install -r requirements.txt  # then run VSSD stage-1 (QCM→R) and stage-2 (QCMR→A) inference for Base/Large under PI and BKG settings and save table3_baseline_comparison.csv
```

## Evaluation signals

- CSV contains rows for both Base and Large under PI and BKG, with RougeL and accuracy fields populated for every row.
- Reproduced PI accuracy exceeds BKG accuracy for both model sizes (e.g., Base 89.67 vs 87.95; Large 93.40 vs 90.64), matching the reported trend.
- Reproduced RougeL for PI slightly exceeds BKG per model (Base 98.11 vs 97.81; Large 98.43 vs 98.13).
- Overall accuracy is consistent with α = 0.2 being the reported best overall setting on ScienceQA.
- Per-category accuracy breakdown includes NAT, TXT, SOC and grade groups G1–6 as in the paper's table.

## Limitations

- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work, so the PI setting is fixed by the current method.
- Method is limited to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work, so conclusions may not transfer to larger models.
- The discussion section reports no quantitative results or hyperparameters, so validation relies on the paper's tables and README analysis plots (all_vs_alpha.png, beta_vs_accuracy.png).

## Evidence

- [results] PI outperforms BKG for both Base (RougeL 98.11 vs 97.81; accuracy 89.67 vs 87.95) and Large (RougeL 98.43 vs 98.13; accuracy 93.40 vs 90.64): [evidence span withheld — anchor status: not_found]
- [readme] α = 0.2 yields the best overall accuracy on ScienceQA: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset, along with peak performance across most subcategories such as NAT, TXT, and G1–6"
- [discussion] VSSD generates attention-guided perturbed image to capture task-sensitive feature directions: "VSSD generates attention-guided perturbed image to capture task-sensitive feature directions"
- [abstract] Experiments on ScienceQA and M3CoT demonstrate improved rationale generation and answer inference: "Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference."
- [methods] β = 0.2 balances the main task loss and the visually saliency-guided distillation loss: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
