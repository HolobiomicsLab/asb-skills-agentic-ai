---
name: inter-layer-steering-vector-distillation
description: Use when reproducing or ablating VSSD on ScienceQA/M3CoT and you need
  to quantify how disabling inter-layer steering-vector distillation (w/o ILD) degrades
  accuracy and RougeL relative to full VSSD Base.
when_to_use_negative:
- When ablating the perturbed-image component instead — use the w/o PI variant, which
  disables attention-guided perturbed-image generation and SVD steering-vector extraction
  entirely (avg accuracy 87.46).
- When the goal is full VSSD Base performance rather than the ILD ablation — enable
  both PI generation and inter-layer distillation with α = 0.2, β = 0.2.
- When evaluating on benchmarks outside the small multimodal CoT setting; the article
  defers broader multimodal reasoning tasks to future work.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3393
tools:
- name: PyTorch
  role: Training framework for the w/o ILD VSSD Base variant with the distillation
    loss disabled
- name: Hugging Face Transformers
  role: Provides the multimodal LLM backbones used for ScienceQA rationale generation
    and answer inference
- name: pandas
  role: Compiles the w/o ILD evaluation metrics into ablation_results.csv for comparison
    against Table 4 reference rows
- name: VSSD
  role: Reference implementation providing training scripts, requirements.txt, and
    α/β hyperparameter analysis
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:47c4bdc07d175a9129ba07b308a1d7ac175b42631df014ad9d50a917350b3e1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - Hugging Face Transformers
  - PyTorch
  - VSSD
  - pandas
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3393
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# inter-layer-steering-vector-distillation

## Summary

Reproduces the VSSD inter-layer (steering-vector) distillation component, where an SVD-extracted steering vector from original-vs-perturbed-image feature differences is injected into intermediate layers via a distillation loss weighted by β = 0.2. This isolates the ILD contribution to ScienceQA accuracy as reported in the Table 4 ablation (w/o ILD: 85.55 avg accuracy, RougeL 97.33).

## When to use

Use when reproducing or ablating VSSD on ScienceQA/M3CoT and you need to quantify how disabling inter-layer steering-vector distillation (w/o ILD) degrades accuracy and RougeL relative to full VSSD Base.

## When NOT to use

- When ablating the perturbed-image component instead — use the w/o PI variant, which disables attention-guided perturbed-image generation and SVD steering-vector extraction entirely (avg accuracy 87.46).
- When the goal is full VSSD Base performance rather than the ILD ablation — enable both PI generation and inter-layer distillation with α = 0.2, β = 0.2.
- When evaluating on benchmarks outside the small multimodal CoT setting; the article defers broader multimodal reasoning tasks to future work.

## Inputs

- ScienceQA dataset (train/test splits with images and questions)
- VSSD Base model configuration with α = 0.2, β = 0.2
- Attention-guided perturbed images generated from MLLM attention maps
- requirements.txt dependency list from BGWH123/VSSD

## Outputs

- w/o ILD variant trained checkpoint
- Per-subcategory accuracies (NAT, TXT, SOC, G1–6) on ScienceQA test split
- Overall average accuracy (target reference: 85.55) and RougeL on CoT rationales (target reference: 97.33)
- ablation_results.csv with metrics for comparison against Table 4

## How to apply

Set up the VSSD environment from the BGWH123/VSSD repository with 'pip install -r requirements.txt' and load ScienceQA with the optimal hyperparameters α = 0.2 (steering strength) and β = 0.2 (distillation loss weight). Keep attention-guided perturbed-image generation active — MLLM attention maps mask question-relevant image regions to produce negatively perturbed samples, and SVD extracts the dominant steering vector from the original-vs-masked feature difference — but disable the distillation loss that injects this vector into intermediate layers. Train this w/o ILD variant in PyTorch, then evaluate on the ScienceQA test split computing per-subcategory accuracies (NAT, TXT, SOC, grade groups G1–6), overall average accuracy, and RougeL on generated chain-of-thought rationales. The rationale is that β = 0.2 'effectively balances the main task loss and the visually saliency-guided distillation loss', so removing ILD should cost ~1.9 points of average accuracy versus full VSSD Base (87.46 w/o PI vs 85.55 w/o ILD indicates ILD contributes more than PI).

## Related tools

- **PyTorch** (Training framework for the w/o ILD VSSD Base variant with the distillation loss disabled)
- **Hugging Face Transformers** (Provides the multimodal LLM backbones used for ScienceQA rationale generation and answer inference)
- **pandas** (Compiles the w/o ILD evaluation metrics into ablation_results.csv for comparison against Table 4 reference rows)
- **VSSD** (Reference implementation providing training scripts, requirements.txt, and α/β hyperparameter analysis) — https://github.com/BGWH123/VSSD

## Examples

```
pip install -r requirements.txt && python train.py --model vssd_base --dataset ScienceQA --alpha 0.2 --beta 0.0 --variant wo_ILD
```

## Evaluation signals

- w/o ILD average accuracy on ScienceQA should fall below full VSSD Base and match the Table 4 reference value of 85.55.
- w/o ILD RougeL on generated rationales should approximate the Table 4 reference value of 97.33.
- w/o ILD average accuracy (85.55) should be lower than the w/o PI variant (87.46), confirming ILD is the larger contributor in the ablation.
- Steering-vector extraction via SVD from the original-vs-perturbed feature difference must remain active while only the inter-layer distillation loss is disabled.
- Reproduced rows in ablation_results.csv must align with the Table 4 reference rows within acceptable tolerance.

## Limitations

- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work.
- Method is limited to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work.
- α significantly affects performance: too small (e.g., 0.1) gives insufficient steering signal, too large (≥0.3) distorts feature representations and notably degrades SOC performance, so the ablation must not deviate from α = 0.2.

## Evidence

- [intro] SVD is applied to extract the dominant steering vector from the difference between original and masked-image features, representing the optimal direction for amplifying fine-grained visual-semantic discrepancies: "singular value de- composition (SVD) is applied to extract the dominant steer- ing vector from this difference"
- [methods] β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
- [discussion] VSSD generates attention-guided perturbed image to capture task-sensitive feature directions and distills steering vectors into intermediate layers: "VSSD generates attention-guided perturbed image to capture task-sensitive feature directions"
- [other] Removing ILD yields lower accuracy than removing PI, quantified in the Table 4 ablation card: [evidence span withheld — anchor status: not_found]
- [readme] α = 0.2 is the optimal steering strength established on ScienceQA: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
