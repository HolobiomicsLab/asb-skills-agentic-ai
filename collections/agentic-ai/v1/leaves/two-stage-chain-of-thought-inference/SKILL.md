---
name: two-stage-chain-of-thought-inference
description: Use when evaluating a multimodal CoT model on ScienceQA-style QA by first
  generating rationales from question+context+image (QCM→R) and then answering conditioned
  on those rationales (QCMR→A), especially when comparing perturbed-image vs black-image
  inputs.
when_to_use_negative:
- Evaluating single-stage (direct QCM→A) models that do not produce intermediate rationales,
  since the QCM→R then QCMR→A protocol does not apply
- Datasets other than ScienceQA-style QA benchmarks (e.g., M3CoT) unless the same
  rationale references and subject/grade splits (NAT, TXT, SOC, G1–6) are available
- Input is already a completed Table 3 comparison — no need to rerun the two-stage
  inference
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: VSSD code repository
  role: Provides the environment, checkpoints, and inference scripts for running the
    two-stage QCM→R and QCMR→A evaluation on ScienceQA
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:efdea967d7b12c80ad19d4c48bd98153eea0a12b93ff5c841a0d7f7e4041c06b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - VSSD code repository
  - VSSD code repository (github.com/BGWH123/VSSD)
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# two-stage-chain-of-thought-inference

## Summary

Evaluates multimodal chain-of-thought distillation models via a two-stage inference protocol on ScienceQA: stage-1 (QCM→R) generates rationales scored with RougeL, then stage-2 (QCMR→A) infers answers scored by accuracy, comparing attention-guided perturbed-image (PI) settings against black-image (BKG) baselines.

## When to use

Use when evaluating a multimodal CoT model on ScienceQA-style QA by first generating rationales from question+context+image (QCM→R) and then answering conditioned on those rationales (QCMR→A), especially when comparing perturbed-image vs black-image inputs.

## When NOT to use

- Evaluating single-stage (direct QCM→A) models that do not produce intermediate rationales, since the QCM→R then QCMR→A protocol does not apply
- Datasets other than ScienceQA-style QA benchmarks (e.g., M3CoT) unless the same rationale references and subject/grade splits (NAT, TXT, SOC, G1–6) are available
- Input is already a completed Table 3 comparison — no need to rerun the two-stage inference

## Inputs

- ScienceQA test split
- Trained VSSD Base checkpoint
- Trained VSSD Large checkpoint
- Reference rationales for RougeL scoring
- PI (perturbed-image) and BKG (black-image) input settings

## Outputs

- Per-setting RougeL scores for rationale generation (stage 1)
- Overall and per-category (NAT, TXT, SOC) and per-grade-group (G1–6) answer accuracy (stage 2)
- table3_baseline_comparison.csv with columns Model, Setting, RougeL, Accuracy

## How to apply

Set up the VSSD environment with 'pip install -r requirements.txt' and load the ScienceQA test split together with the trained VSSD Base and Large checkpoints. Run stage-1 inference (QCM→R) for each model under both the PI (attention-guided perturbed image) and BKG (black image) settings, computing RougeL between generated and reference rationales. Then run stage-2 inference (QCMR→A) for the same model/setting pairs, computing answer accuracy overall and per subject category (NAT, TXT, SOC) and grade group (G1–6). Assemble RougeL and accuracy into a four-column comparison mirroring Table 3 (Model, Setting, RougeL, Accuracy) and save it as table3_baseline_comparison.csv. The rationale is that the two-stage protocol decouples rationale generation quality from answer inference quality, so PI-vs-BKG gaps can be attributed to visual saliency steering rather than single-pass answering.

## Related tools

- **VSSD code repository** (Provides the environment, checkpoints, and inference scripts for running the two-stage QCM→R and QCMR→A evaluation on ScienceQA) — https://github.com/BGWH123/VSSD

## Examples

```
pip install -r requirements.txt && python run_inference.py --dataset ScienceQA --checkpoint vssd_base --image_setting PI --stage QCM2R --alpha 0.2
```

## Evaluation signals

- Reproduced PI accuracy values exceed BKG for both Base (89.67 vs 87.95) and Large (93.40 vs 90.64), consistent with reported trends
- Reproduced PI RougeL values exceed BKG for both Base (98.11 vs 97.81) and Large (98.43 vs 98.13)
- CSV contains rows for both Base and Large under PI and BKG with RougeL and accuracy fields present
- Per-subject breakdown includes NAT, TXT, SOC and grade groups G1–6
- Best overall accuracy is obtained at steering strength α = 0.2 on ScienceQA

## Limitations

- Two-stage setting on ScienceQA only; broader multimodal reasoning tasks are left to future work per the discussion section
- Steering vector extraction is not yet adaptive; more adaptive extraction deferred to future work
- No quantitative results, hyperparameters, or model versions reported in the discussion section itself, so validation must rely on the abstract/methods numbers

## Evidence

- [abstract] Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference.: "Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference."
- [methods] alpha best overall accuracy: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [readme] pip install requirements: "pip install -r requirements.txt"
- [discussion] future work scope: [evidence span withheld — anchor status: not_found]
