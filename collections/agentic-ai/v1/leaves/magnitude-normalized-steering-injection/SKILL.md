---
name: magnitude-normalized-steering-injection
description: Use when distilling a small multimodal CoT model and you need to steer
  intermediate decoder layers using the dominant direction of original-vs-perturbed-image
  feature differences on benchmarks like ScienceQA or M3CoT.
when_to_use_negative:
- Task is not small-model multimodal chain-of-thought reasoning; the discussion explicitly
  defers extension to broader multimodal reasoning tasks to future work
- No paired original-vs-perturbed decoder feature differences can be produced (e.g.,
  no saliency-based perturbation pipeline available), so no meaningful difference
  tensor exists for SVD
- Steering vector extraction must be adaptive — the paper notes current extraction
  is not adaptive and defers adaptive extraction to future work
tools:
- name: PyTorch
  role: Extract decoder hidden states and compute torch.linalg.svd on layer-wise difference
    tensors to obtain top-1 right singular vectors
- name: NumPy
  role: Alternative/auxiliary SVD computation on difference tensors
- name: VSSD codebase
  role: Reference implementation providing multimodal decoder, saliency perturbation,
    steering injection (Eq. 8), and distillation loss (Eq. 9)
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:8cc837cd64c65e3658b2bb3b83d9d2bde964c2f08ecb5db37cf4b50e6e5cc11e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - NumPy
  - PyTorch
  - VSSD codebase
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# magnitude-normalized-steering-injection

## Summary

Extracts dominant steering vectors via SVD on layer-wise decoder feature differences between original and saliency-perturbed images, then injects magnitude-normalized vectors into a multimodal LLM's intermediate layers (Eq. 8) to amplify sensitivity to fine-grained visual-semantic differences.

## When to use

Use when distilling a small multimodal CoT model and you need to steer intermediate decoder layers using the dominant direction of original-vs-perturbed-image feature differences on benchmarks like ScienceQA or M3CoT.

## When NOT to use

- Task is not small-model multimodal chain-of-thought reasoning; the discussion explicitly defers extension to broader multimodal reasoning tasks to future work
- No paired original-vs-perturbed decoder feature differences can be produced (e.g., no saliency-based perturbation pipeline available), so no meaningful difference tensor exists for SVD
- Steering vector extraction must be adaptive — the paper notes current extraction is not adaptive and defers adaptive extraction to future work

## Inputs

- Decoder hidden states for original images (per layer, per batch element)
- Decoder hidden states for attention-guided saliency-perturbed (masked) images
- Steering strength α (reported optimal: 0.2)
- Distillation loss trade-off β (reported optimal: 0.2)

## Outputs

- Top-1 right singular vector per decoder layer (dominant steering vector)
- Magnitude-normalized steering vectors injected into intermediate layers (Eq. 8)
- Combined loss = NLL main-task loss + β · visually saliency-guided distillation loss (Eq. 9)

## How to apply

Run the multimodal decoder from the VSSD codebase on paired original and attention-guided (saliency-masked) perturbed images, and record decoder hidden states at each layer. Compute the difference tensor between original and perturbed-image hidden states per layer and per batch element. Apply SVD (torch.linalg.svd) to each difference tensor E = UΣV⊤ and select the top-1 right singular vector as the principal editing direction, then normalize its magnitude. Inject the normalized steering vector into the model's intermediate layers per Eq. 8, scaling injection with strength α; the README reports α = 0.2 as optimal on ScienceQA (α ≤ 0.1 under-steers, α ≥ 0.3 distorts features). Combine the resulting visually saliency-guided distillation loss with the NLL main-task loss via trade-off β (Eq. 9), with β = 0.2 reported as the balance point; deviations in either direction degrade accuracy.

## Related tools

- **PyTorch** (Extract decoder hidden states and compute torch.linalg.svd on layer-wise difference tensors to obtain top-1 right singular vectors)
- **NumPy** (Alternative/auxiliary SVD computation on difference tensors)
- **VSSD codebase** (Reference implementation providing multimodal decoder, saliency perturbation, steering injection (Eq. 8), and distillation loss (Eq. 9)) — https://github.com/BGWH123/VSSD

## Examples

```
U, S, Vh = torch.linalg.svd(E); v = Vh[0]
```

## Evaluation signals

- Steering vector equals the top-1 right singular vector of Vh returned by torch.linalg.svd on the per-layer difference tensor E = UΣV⊤
- Injected vectors are magnitude-normalized before layer-wise injection per Eq. 8
- With α = 0.2, overall accuracy on ScienceQA peaks, including subcategories NAT, TXT, and G1–6 (α = 0.1 under-activates; α ≥ 0.3 degrades SOC)
- With β = 0.2, the combined loss balances main-task and distillation terms; accuracy drops when β is increased or decreased
- Pipeline runs end-to-end after pip install -r requirements.txt on ScienceQA / M3CoT with improved rationale generation and answer inference

## Limitations

- α is sensitive: α = 0.1 gives insufficient steering signal, while α ≥ 0.3 distorts feature representations and notably degrades complex categories like SOC
- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work
- Method is limited to small multimodal CoT settings; broader multimodal reasoning tasks left to future work
- Optimal α = 0.2 and β = 0.2 are calibrated for the current architecture on ScienceQA; transfer to other architectures/datasets is not established

## Evidence

- [abstract] SVD extracts dominant steering vector: "applies singular value decomposition to extract dominant steering vectors to guide inter-layer distillation"
- [methods] Alpha controls steering injection strength: "the hyperparameter α controls the strength with which the steering vector—derived from the feature difference between the original and perturbed images—is injected into the model's intermediate layers"
- [methods] Beta balances main task and distillation loss: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
- [readme] Alpha 0.2 optimal on ScienceQA: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset, along with peak performance across most subcategories such as NAT, TXT, and G1–6"
