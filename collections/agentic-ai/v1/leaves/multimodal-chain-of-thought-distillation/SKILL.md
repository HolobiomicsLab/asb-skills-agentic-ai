---
name: multimodal-chain-of-thought-distillation
description: Use when training or evaluating a small multimodal model for chain-of-thought
  answer/rationale generation on benchmarks like ScienceQA or M3CoT, and cross-modal
  sensitivity to fine-grained visual-semantic differences must be improved via distillation
  from a larger MLLM.
when_to_use_negative:
- The task targets broader multimodal reasoning beyond small multimodal CoT settings
  — the paper explicitly defers this to future work
- An adaptive, input-dependent steering-vector extraction is required — current extraction
  is not adaptive and adaptive extraction is deferred to future work
- No paired question-image data or attention maps from a larger MLLM are available
  to generate the negatively perturbed semantic samples
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: VSSD code repository
  role: Provides the released implementation, requirements.txt environment, and hyperparameter-analysis
    scripts for reproducing the steering-distillation pipeline
  repo: https://github.com/BGWH123/VSSD
- name: pip / requirements.txt
  role: Environment setup for the VSSD training and evaluation pipeline
  repo: https://github.com/BGWH123/VSSD
- name: PyTorch
  role: Runs the saliency-guided distillation with SVD steering-vector injection and
    evaluates accuracy/RougeL across layer settings
- name: matplotlib
  role: Plots accuracy and RougeL against L in the paper's hyperparameter-analysis
    figure style (layer_vs_accuracy.png)
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:bb278407859adb82da2afa91ca5e391c208119a31f7077ada079c07dbf637664
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - PyTorch
  - Python
  - VSSD code repository
  - VSSD code repository (github.com/BGWH123/VSSD)
  - matplotlib
  - pip / requirements.txt
  - pip / requirements.txt environment setup
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# multimodal-chain-of-thought-distillation

## Summary

Implements Visual Saliency-guided Steering Distillation (VSSD), which distills large multimodal LLM reasoning into small models for multimodal chain-of-thought tasks by injecting SVD-extracted steering vectors from saliency-perturbed images into intermediate decoder layers.

## When to use

Use when training or evaluating a small multimodal model for chain-of-thought answer/rationale generation on benchmarks like ScienceQA or M3CoT, and cross-modal sensitivity to fine-grained visual-semantic differences must be improved via distillation from a larger MLLM.

## When NOT to use

- The task targets broader multimodal reasoning beyond small multimodal CoT settings — the paper explicitly defers this to future work
- An adaptive, input-dependent steering-vector extraction is required — current extraction is not adaptive and adaptive extraction is deferred to future work
- No paired question-image data or attention maps from a larger MLLM are available to generate the negatively perturbed semantic samples

## Inputs

- Multimodal chain-of-thought benchmark data (ScienceQA, M3CoT) with question-image-answer items
- Attention maps from a multimodal large language model used to mask question-relevant image regions
- VSSD repository code (github.com/BGWH123/VSSD) with requirements.txt environment
- Swept hyperparameter settings for α (steering strength), β (distillation loss weight), and L (number of last decoder layers)

## Outputs

- scienceqa_results.csv with overall and subcategory (NAT, TXT, SOC, G1–6) accuracies
- layer_sweep_results.csv with accuracy and RougeL per L setting
- layer_vs_accuracy.png line plot of accuracy and RougeL vs L
- Trained small multimodal CoT model (e.g., VSSD Base 223M) with answer accuracy and generated rationales

## How to apply

First set up the released VSSD environment with 'pip install -r requirements.txt'. Generate attention-guided perturbed images by using the MLLM's attention maps to mask the image regions most relevant to the question, producing negatively perturbed semantic samples; compute the difference between original and perturbed decoder features and apply singular value decomposition (SVD) to extract the dominant steering vector. Train with a two-stage chain-of-thought scheme, injecting the steering vector into intermediate decoder layers at steering strength α = 0.2 — the swept optimum, where α = 0.1 gives an insufficient signal and α ≥ 0.3 distorts features and degrades complex categories like SOC. Weight the visually saliency-guided distillation loss against the main task loss with β = 0.2, which balances the two losses in the current architecture. Extract steering vectors from only the last L = 2 decoder layers, since L = 1 gives too weak a signal and L > 2 introduces redundant or noisy information. Evaluate answer-inference accuracy on ScienceQA overall and across subcategories (NAT, TXT, SOC, G1–6) and RougeL on generated rationales, tabulating results (e.g., scienceqa_results.csv, layer_sweep_results.csv, layer_vs_accuracy.png).

## Related tools

- **VSSD code repository** (Provides the released implementation, requirements.txt environment, and hyperparameter-analysis scripts for reproducing the steering-distillation pipeline) — https://github.com/BGWH123/VSSD
- **pip / requirements.txt** (Environment setup for the VSSD training and evaluation pipeline) — https://github.com/BGWH123/VSSD
- **PyTorch** (Runs the saliency-guided distillation with SVD steering-vector injection and evaluates accuracy/RougeL across layer settings)
- **matplotlib** (Plots accuracy and RougeL against L in the paper's hyperparameter-analysis figure style (layer_vs_accuracy.png))

## Examples

```
git clone https://github.com/BGWH123/VSSD && cd VSSD && pip install -r requirements.txt
```

## Evaluation signals

- At α = 0.2 and β = 0.2 the model achieves the best overall accuracy on ScienceQA among the swept settings, peaking across most subcategories such as NAT, TXT, and G1–6
- Accuracy and RougeL curves plotted against L peak at L = 2, matching Fig. 3(a) of the paper
- VSSD Base (223M) reaches 89.67% accuracy on ScienceQA, outperforming same-size baselines such as Enigma-COT
- No performance degradation occurs on complex categories like SOC at the optimal α setting
- Increasing or decreasing β away from 0.2 leads to performance degradation across NAT, TXT, NO, G1–6 and overall average accuracy

## Limitations

- Method is limited to small multimodal CoT settings; extension to broader multimodal reasoning tasks is left to future work
- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work
- α must be precisely calibrated: α too small (e.g., 0.1) gives insufficient steering signal, α too large (e.g., ≥0.3) distorts feature representations and notably degrades SOC performance
- L > 2 introduces redundant or noisy information from earlier decoder layers

## Evidence

- [abstract] VSSD leverages attention maps to generate perturbed images capturing task-sensitive feature directions via SVD steering vectors: "VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions"
- [methods] α = 0.2 yields best overall accuracy on ScienceQA with peak performance across subcategories: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [readme] α = 0.2 optimal with small α insufficient and large α degrading SOC: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset, along with peak performance across most subcategories such as NAT, TXT, and G1–6"
- [readme] β = 0.2 balances main task loss and distillation loss: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
- [abstract] Experiments on ScienceQA and M3CoT demonstrate improved rationale generation and answer inference: "Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference."
