---
name: layer-wise-feature-distillation
description: Use when a small multimodal chain-of-thought model must acquire sensitivity
  to fine-grained visual-semantic differences by distilling task-critical feature
  directions from a teacher MLLM's decoder layers on ScienceQA/M3CoT-style data.
when_to_use_negative:
- The target task lies in broader multimodal reasoning beyond small multimodal CoT
  settings — the article explicitly defers this extension to future work
- No paired original vs. perturbed-image decoder feature differences can be obtained,
  since the steering vector is defined as the top right singular vector of that difference
  tensor
- An adaptive, input-dependent steering-vector extraction is required — the paper
  notes current extraction is not adaptive and defers adaptivity to future work
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Extract decoder hidden states for original and perturbed images and apply
    torch.linalg.svd to layer-wise difference tensors
- name: NumPy
  role: Numerical support for computing SVD of difference tensors and normalizing
    steering-vector magnitudes
- name: VSSD codebase
  role: Reference implementation of steering-vector injection (Eq. 8) and the visually
    saliency-guided distillation loss (Eq. 9)
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:7c1880895fcaa44e51ad2bb9d0e62c9ba250626340a47ef7000d25bae4c776ff
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
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# layer-wise-feature-distillation

## Summary

A distillation skill from VSSD that extracts dominant steering vectors via SVD on the difference between original and saliency-perturbed decoder hidden states, injects them into intermediate layers (alpha = 0.2), and combines a visually saliency-guided distillation loss with the NLL main-task loss (beta = 0.2) to enhance fine-grained cross-modal sensitivity in small multimodal CoT models.

## When to use

Use when a small multimodal chain-of-thought model must acquire sensitivity to fine-grained visual-semantic differences by distilling task-critical feature directions from a teacher MLLM's decoder layers on ScienceQA/M3CoT-style data.

## When NOT to use

- The target task lies in broader multimodal reasoning beyond small multimodal CoT settings — the article explicitly defers this extension to future work
- No paired original vs. perturbed-image decoder feature differences can be obtained, since the steering vector is defined as the top right singular vector of that difference tensor
- An adaptive, input-dependent steering-vector extraction is required — the paper notes current extraction is not adaptive and defers adaptivity to future work

## Inputs

- Original and saliency-perturbed (attention-masked) images from ScienceQA or M3CoT
- Per-layer decoder hidden states from the multimodal model (PyTorch tensors)
- Hyperparameters: steering strength alpha (= 0.2) and distillation-loss weight beta (= 0.2)

## Outputs

- Top-1 right singular vectors (dominant steering vectors) per decoder layer
- Magnitude-normalized steering vectors injected into intermediate layers (Eq. 8)
- Combined training loss: NLL main-task loss + beta-weighted visually saliency-guided distillation loss (Eq. 9)
- Accuracy curves vs. alpha and beta on ScienceQA (overall, NAT, TXT, SOC, NO, G1-6)

## How to apply

First load the VSSD codebase (pip install -r requirements.txt) and extract decoder hidden states for original and attention-masked saliency-perturbed images with PyTorch. For each decoder layer, compute the difference tensor E between original and perturbed-image hidden states and apply SVD (torch.linalg.svd / NumPy, E = UΣV⊤) to select the top-1 right singular vector as the dominant steering direction. Magnitude-normalize the steering vector and inject it into the model's intermediate layers following Eq. 8, with steering strength alpha = 0.2, the value reported optimal on ScienceQA. Compute the visually saliency-guided distillation loss and combine it with the NLL main-task loss via trade-off beta (Eq. 9), using beta = 0.2. Validate end-to-end that the combined loss balances both terms and reproduces peak overall accuracy on ScienceQA.

## Related tools

- **PyTorch** (Extract decoder hidden states for original and perturbed images and apply torch.linalg.svd to layer-wise difference tensors)
- **NumPy** (Numerical support for computing SVD of difference tensors and normalizing steering-vector magnitudes)
- **VSSD codebase** (Reference implementation of steering-vector injection (Eq. 8) and the visually saliency-guided distillation loss (Eq. 9)) — https://github.com/BGWH123/VSSD

## Examples

```
E = H_orig - H_pert; U, S, Vh = torch.linalg.svd(E, full_matrices=False); steering_vec = Vh[0]
```

## Evaluation signals

- Steering vector equals the top-1 right singular vector of the per-layer difference tensor E between original and perturbed-image hidden states (E = UΣV⊤)
- Injected steering vector is magnitude-normalized and applied to intermediate layers with alpha = 0.2, the reported optimal strength
- Combined loss with beta = 0.2 balances the main-task NLL loss and the distillation loss; deviating beta in either direction degrades performance
- alpha = 0.2 yields the best overall accuracy on ScienceQA plus peak performance across most subcategories (NAT, TXT, G1-6); alpha >= 0.3 distorts features and degrades SOC
- The module runs end-to-end under the VSSD environment (pip install -r requirements.txt) on ScienceQA

## Limitations

- Steering-vector extraction is not yet adaptive; the authors defer 'more adaptive steering vector extraction' to future work
- Method is scoped to small multimodal CoT settings; extension to broader multimodal reasoning tasks is left as future work
- Steering strength is sensitive: alpha too small (e.g., 0.1) gives an insufficient signal, while alpha too large (e.g., >=0.3) distorts feature representations and notably degrades SOC performance

## Evidence

- [abstract] SVD extraction of dominant steering vector: "applies singular value decomposition to extract dominant steering vectors to guide inter-layer distillation"
- [full_text] Per-layer difference tensor and top-1 right singular vector: [evidence span withheld — anchor status: not_found]
- [readme] Alpha analysis: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset, along with peak performance across most subcategories such as NAT, TXT, and G1–6"
- [methods] Beta analysis: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
- [discussion] Non-adaptive extraction limitation: "and explore more adaptive steering
vector extraction."
