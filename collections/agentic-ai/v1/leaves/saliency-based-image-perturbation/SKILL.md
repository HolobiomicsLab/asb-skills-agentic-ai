---
name: saliency-based-image-perturbation
description: Use when training a small multimodal chain-of-thought model that confuses
  tiny cross-modal features, and you need task-sensitive visual feature directions
  from an MLLM to guide inter-layer distillation on benchmarks like ScienceQA or M3CoT.
when_to_use_negative:
- The task is a broader multimodal reasoning problem beyond small-model multimodal
  chain-of-thought settings; the paper explicitly defers extension to broader multimodal
  reasoning to future work.
- You need an adaptive steering-vector extraction scheme; the paper states current
  extraction is not yet adaptive and defers adaptive extraction to future work.
- You have no access to MLLM attention maps or cannot mask question-relevant image
  regions, since the perturbation depends on saliency from the MLLM.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: VSSD code repository
  role: Reference implementation for generating attention-guided perturbed images,
    SVD steering-vector extraction, and two-stage chain-of-thought training with α/β
    hyperparameter sweeps on ScienceQA
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:1e8d50627b916ccf444a4307668195bf0b07e7b07e8e3675ae2b42995eb12541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - VSSD code repository
  - VSSD code repository (github.com/BGWH123/VSSD)
  - pip / requirements.txt environment setup
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# saliency-based-image-perturbation

## Summary

Generates attention-guided perturbed images by masking the question-relevant image regions via a multimodal LLM's attention maps, then applies singular value decomposition to the original-vs-perturbed decoder feature difference to extract a dominant steering vector. This steering vector is injected into intermediate layers of a small multimodal chain-of-thought model (VSSD) to amplify fine-grained visual-semantic sensitivity during distillation.

## When to use

Use when training a small multimodal chain-of-thought model that confuses tiny cross-modal features, and you need task-sensitive visual feature directions from an MLLM to guide inter-layer distillation on benchmarks like ScienceQA or M3CoT.

## When NOT to use

- The task is a broader multimodal reasoning problem beyond small-model multimodal chain-of-thought settings; the paper explicitly defers extension to broader multimodal reasoning to future work.
- You need an adaptive steering-vector extraction scheme; the paper states current extraction is not yet adaptive and defers adaptive extraction to future work.
- You have no access to MLLM attention maps or cannot mask question-relevant image regions, since the perturbation depends on saliency from the MLLM.

## Inputs

- ScienceQA or M3CoT image-question pairs (multimodal benchmark samples)
- Attention maps from a multimodal large language model (MLLM)
- Decoder features from the original image
- Decoder features from the attention-guided perturbed (masked) image

## Outputs

- Attention-guided perturbed image capturing task-sensitive feature directions
- Dominant steering vector extracted via SVD of the original-vs-perturbed feature difference
- Steering-vector injections into the model's intermediate layers at strength α

## How to apply

Use the multimodal LLM's attention maps to identify and mask the image regions most relevant to the question, producing negatively perturbed semantic samples. Run the decoder on both the original and perturbed images and compute their feature difference. Apply singular value decomposition (SVD) to this difference to extract the dominant steering vector, which represents the optimal direction for amplifying fine-grained visual-semantic discrepancies. Inject this steering vector into the model's intermediate layers, calibrating the steering strength with the hyperparameter α: α = 0.2 yields the best overall ScienceQA accuracy, while α ≤ 0.1 under-activates fine-grained discrimination and α ≥ 0.3 distorts feature representations and degrades complex categories like SOC. Pair the injection with the visually saliency-guided distillation loss weighted against the main task loss at β = 0.2.

## Related tools

- **VSSD code repository** (Reference implementation for generating attention-guided perturbed images, SVD steering-vector extraction, and two-stage chain-of-thought training with α/β hyperparameter sweeps on ScienceQA) — https://github.com/BGWH123/VSSD

## Examples

```
pip install -r requirements.txt  # from https://github.com/BGWH123/VSSD, then train VSSD on ScienceQA with steering strength α = 0.2 and distillation loss weight β = 0.2
```

## Evaluation signals

- α = 0.2 with β = 0.2 yields the best overall accuracy on ScienceQA among swept steering-strength settings.
- Peak performance is observed across most subcategories such as NAT, TXT, and G1–6 at α = 0.2.
- No degradation occurs on complex categories like SOC at α = 0.2 (degradation is expected for α ≥ 0.3).
- SVD of the original-vs-perturbed decoder feature difference yields a single dominant steering vector suitable for layer-wise injection.
- Compiled accuracies (e.g., scienceqa_results.csv) show consistent improvement in answer inference over same-size baselines such as Enigma-COT (VSSD Base 223M: 89.67%).

## Limitations

- Method is limited to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work.
- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work.
- α is sensitive: too small (e.g., 0.1) provides insufficient steering signal; too large (≥0.3) distorts feature representations and notably degrades performance on complex categories like SOC.

## Evidence

- [abstract] VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions: "VSSD leverages the attention maps of multimodal large language models to generate perturbed images that capture task-sensitive feature directions, and then applies singular value decomposition to"
- [intro] VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples: "VSSD leverages MLLMs to identify and mask the image regions most relevant to the question, generating negatively perturbed semantic samples."
- [methods] the hyperparameter α controls the strength with which the steering vector—derived from the feature difference between the original and perturbed images—is injected into the model's intermediate layers: "the hyperparameter α controls the strength with which the steering vector—derived from the feature difference between the original and perturbed images—is injected into the model's intermediate layers"
- [readme] α = 0.2 yields the best overall accuracy on the ScienceQA dataset, along with peak performance across most subcategories such as NAT, TXT, and G1–6: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset, along with peak performance across most subcategories such as NAT, TXT, and G1–6"
- [discussion] Future work will extend VSSD to broader multimodal reasoning tasks and explore more adaptive steering vector extraction: "Future work will extend VSSD to broader mul-
timodal reasoning tasks
and explore more adaptive steering
vector extraction."
