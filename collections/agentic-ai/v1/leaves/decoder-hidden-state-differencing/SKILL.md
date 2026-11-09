---
name: decoder-hidden-state-differencing
description: Use when you have paired decoder hidden states from a multimodal LLM
  for original vs. attention-masked/perturbed images and need to isolate the principal
  editing direction for layer-wise steering or distillation.
when_to_use_negative:
- The task is a general/broad multimodal reasoning setting rather than a small-model
  multimodal chain-of-thought setting — the discussion explicitly defers broader multimodal
  reasoning to future work
- No paired original-vs-perturbed image feature responses are available, since the
  method fundamentally requires the difference between the two decoder hidden-state
  sets
- An adaptive steering-vector extraction is required — the paper notes current extraction
  is not yet adaptive, with adaptive extraction deferred to future work
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Extract decoder hidden states for original and saliency-perturbed images and
    compute torch.linalg.svd on layer-wise difference tensors
  repo: https://github.com/BGWH123/VSSD
- name: NumPy
  role: Alternative backend for SVD on difference tensors and for magnitude-normalizing
    the selected top-1 right singular vector
- name: pip
  role: Install the VSSD codebase dependencies via requirements.txt before running
    the extraction pipeline
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:39e0ddc2a4bfde580fc48f158d37d0d3839602b741c84889e9c42f686f09da2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - NumPy
  - PyTorch
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# decoder-hidden-state-differencing

## Summary

A VSSD technique that computes layer-wise difference tensors between decoder hidden states of original and saliency-perturbed (masked-region) images, then applies SVD to extract the top-1 right singular vector as the dominant steering direction for fine-grained visual-semantic distillation.

## When to use

Use when you have paired decoder hidden states from a multimodal LLM for original vs. attention-masked/perturbed images and need to isolate the principal editing direction for layer-wise steering or distillation.

## When NOT to use

- The task is a general/broad multimodal reasoning setting rather than a small-model multimodal chain-of-thought setting — the discussion explicitly defers broader multimodal reasoning to future work
- No paired original-vs-perturbed image feature responses are available, since the method fundamentally requires the difference between the two decoder hidden-state sets
- An adaptive steering-vector extraction is required — the paper notes current extraction is not yet adaptive, with adaptive extraction deferred to future work

## Inputs

- Per-layer decoder hidden states of a multimodal LLM for original images (e.g., ScienceQA/M3CoT samples)
- Per-layer decoder hidden states for the corresponding attention-guided negatively perturbed (masked-region) images
- Steering strength hyperparameter alpha (optimal 0.2) and distillation loss weight beta (optimal 0.2)

## Outputs

- Layer-wise difference tensors E between original and perturbed-image hidden states
- Top-1 right singular vector per layer from SVD (E = UΣV⊤) as the magnitude-normalized steering vector
- Steered intermediate-layer representations used in the visually saliency-guided distillation loss combined with the NLL main-task loss

## How to apply

Load the VSSD multimodal decoder (pip install -r requirements.txt) and run forward passes on ScienceQA/M3CoT samples to record hidden states at every decoder layer for both the original image and the MLLM-attention-guided masked/perturbed image. For each decoder layer, subtract the perturbed-image hidden states from the original-image hidden states to form a difference tensor E per batch element. Apply torch.linalg.svd (or NumPy SVD) to each difference tensor (E = UΣV⊤) and select the top-1 right singular vector as the dominant steering vector, then magnitude-normalize it before injecting into intermediate layers with strength α (optimal α = 0.2 on ScienceQA). Feed the steered states into the visually saliency-guided distillation loss, weighted against the NLL main-task loss by β = 0.2, which the README reports as the balance point where accuracy peaks across NAT, TXT, NO, and G1–6 subcategories.

## Related tools

- **PyTorch** (Extract decoder hidden states for original and saliency-perturbed images and compute torch.linalg.svd on layer-wise difference tensors) — https://github.com/BGWH123/VSSD
- **NumPy** (Alternative backend for SVD on difference tensors and for magnitude-normalizing the selected top-1 right singular vector)
- **pip** (Install the VSSD codebase dependencies via requirements.txt before running the extraction pipeline) — https://github.com/BGWH123/VSSD

## Examples

```
U, S, Vh = torch.linalg.svd(h_orig - h_pert, full_matrices=False); steer_vec = Vh[0]
```

## Evaluation signals

- The SVD of each layer-wise difference tensor yields a top-1 right singular vector whose injection with α = 0.2 reproduces peak overall accuracy on ScienceQA (with peaks in NAT, TXT, and G1–6 subcategories)
- Setting α = 0.1 produces insufficient steering (worse fine-grained discrimination) and α ≥ 0.3 degrades performance on complex categories like SOC — results should follow this monotonic pattern
- The combined loss with β = 0.2 balances the NLL main-task loss and the visually saliency-guided distillation loss; deviating β up or down degrades accuracy across NAT, TXT, NO, and G1–6
- The steering vector is magnitude-normalized before injection into intermediate layers per Eq. 8, and the module runs end-to-end on ScienceQA without shape/rank errors in the per-batch SVD

## Limitations

- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work
- Method is limited to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work
- Steering strength is sensitive: α too small (e.g., 0.1) gives insufficient signal, while α too large (e.g., ≥0.3) distorts feature representations and notably degrades SOC performance

## Evidence

- [abstract] applies SVD to extract dominant steering vectors: "VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions, and then applies singular value decomposition to"
- [intro] SVD extracts dominant steering vector from original-vs-masked difference: "Subsequently, singular value de- composition (SVD) is applied to extract the dominant steer- ing vector from this difference"
- [methods] alpha optimal at 0.2 on ScienceQA: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [discussion] beta balances main and distillation losses: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
- [readme] alpha controls steering-vector injection strength: "the hyperparameter α controls the strength with which the steering vector—derived from the feature difference between the original and perturbed images—is injected into the model's intermediate layers"
