---
name: multimodal-distillation-model-training
description: Use when training or reproducing a small multimodal CoT model with saliency-guided
  distillation on ScienceQA/M3CoT, especially when reproducing the Table 4 ablations
  of the perturbed image (PI) and inter-layer distillation (ILD) components.
when_to_use_negative:
- When the goal is broader multimodal reasoning beyond small-model chain-of-thought
  tasks — the article states future work will extend VSSD to broader multimodal reasoning
  tasks.
- When no pretrained multimodal LLM is available to generate attention-guided perturbed
  images, since the steering vector is derived from the original-vs-perturbed feature
  difference.
- When you only need inference on an already-trained VSSD checkpoint rather than training
  or ablation reproduction.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Trains the full, w/o PI, and w/o ILD VSSD variants with the steering-vector
    injection and distillation loss
  repo: https://github.com/BGWH123/VSSD
- name: Hugging Face Transformers
  role: Provides the multimodal LLM backbone used to produce attention maps for perturbed-image
    generation and the small model being distilled
  repo: https://github.com/BGWH123/VSSD
- name: pandas
  role: Compiles per-subcategory accuracies and RougeL scores for both ablation variants
    into ablation_results.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:f3d1fed213031e3255bbef9d24c5591901596b5b02ea67c735e56d68fea4374f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - Hugging Face Transformers
  - PyTorch
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# multimodal-distillation-model-training

## Summary

Trains a small multimodal chain-of-thought model with Visual Saliency Steering Distillation (VSSD): attention-guided perturbed images capture task-sensitive feature directions, SVD extracts dominant steering vectors, and inter-layer distillation injects them, with ablation variants (w/o PI, w/o ILD) reproducing Table 4 on ScienceQA.

## When to use

Use when training or reproducing a small multimodal CoT model with saliency-guided distillation on ScienceQA/M3CoT, especially when reproducing the Table 4 ablations of the perturbed image (PI) and inter-layer distillation (ILD) components.

## When NOT to use

- When the goal is broader multimodal reasoning beyond small-model chain-of-thought tasks — the article states future work will extend VSSD to broader multimodal reasoning tasks.
- When no pretrained multimodal LLM is available to generate attention-guided perturbed images, since the steering vector is derived from the original-vs-perturbed feature difference.
- When you only need inference on an already-trained VSSD checkpoint rather than training or ablation reproduction.

## Inputs

- ScienceQA dataset (image-question-answer-rationale splits)
- M3CoT dataset
- VSSD repository configuration with hyperparameters α = 0.2 and β = 0.2
- A pretrained multimodal LLM used to generate attention-guided perturbed images

## Outputs

- Trained VSSD Base model checkpoints (full, w/o PI, w/o ILD variants)
- Per-subcategory accuracy (NAT, TXT, SOC, G1–6) and overall average accuracy on the ScienceQA test split
- RougeL scores on generated chain-of-thought rationales
- ablation_results.csv with reproduced Table 4 metrics (e.g., w/o PI: 87.46 accuracy / 97.74 RougeL; w/o ILD: 85.55 accuracy / 97.33 RougeL)

## How to apply

Set up the VSSD repository environment with 'pip install -r requirements.txt' and train in PyTorch with Hugging Face Transformers on the ScienceQA dataset. Enable attention-guided perturbed-image generation, in which an MLLM masks question-relevant image regions to create negatively perturbed samples, then apply singular value decomposition to the original-vs-perturbed feature difference to obtain the dominant steering vector. Use the validated hyperparameters α = 0.2 (steering strength for injecting the vector into intermediate layers) and β = 0.2 (distillation loss weight), which the paper's sensitivity analyses identify as optimal. For the Table 4 ablation, train a w/o PI variant with perturbed-image generation disabled and a w/o ILD variant with only the inter-layer steering-vector distillation loss disabled. Evaluate both variants on the ScienceQA test split, reporting accuracy per subcategory (NAT, TXT, SOC, G1–6), overall average accuracy, and RougeL on generated rationales, and compile the metrics into ablation_results.csv for comparison against Table 4.

## Related tools

- **PyTorch** (Trains the full, w/o PI, and w/o ILD VSSD variants with the steering-vector injection and distillation loss) — https://github.com/BGWH123/VSSD
- **Hugging Face Transformers** (Provides the multimodal LLM backbone used to produce attention maps for perturbed-image generation and the small model being distilled) — https://github.com/BGWH123/VSSD
- **pandas** (Compiles per-subcategory accuracies and RougeL scores for both ablation variants into ablation_results.csv)

## Examples

```
git clone https://github.com/BGWH123/VSSD && cd VSSD && pip install -r requirements.txt
```

## Evaluation signals

- w/o PI variant reproduces Table 4: average accuracy 87.46 with RougeL 97.74 on ScienceQA
- w/o ILD variant reproduces Table 4: average accuracy 85.55 with RougeL 97.33, both rows below full VSSD Base
- α = 0.2 yields the best overall ScienceQA accuracy and peak performance on most subcategories (NAT, TXT, G1–6); larger α (≥0.3) degrades SOC
- β = 0.2 gives optimal performance across NAT, TXT, NO, G1–6 and overall average accuracy; deviating in either direction degrades results
- ablation_results.csv rows match the Table 4 reference values for both variants

## Limitations

- Method is scoped to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work.
- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work.
- α and β are calibrated on ScienceQA — deviations degrade performance (e.g., α ≥ 0.3 notably degrades SOC; too small α like 0.1 gives insufficient steering signal).

## Evidence

- [abstract] abstract: "VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions, and then applies singular value decomposition to"
- [methods] methods-alpha: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [discussion] discussion-workflow: "VSSD generates attention-guided perturbed image to capture task-sensitive feature directions"
- [readme] readme-beta: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
