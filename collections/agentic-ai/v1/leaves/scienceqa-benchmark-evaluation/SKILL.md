---
name: scienceqa-benchmark-evaluation
description: Use when a small multimodal chain-of-thought model (e.g., VSSD Base,
  223M) trained on ScienceQA must be benchmarked for answer-inference accuracy overall
  and across NAT, TXT, SOC, and grade subcategories.
when_to_use_negative:
- The target task is broader multimodal reasoning beyond small multimodal CoT settings
  — the article explicitly defers this to future work.
- You need adaptive steering vector extraction rather than the fixed SVD-based extraction,
  since adaptivity is explicitly left to future work.
- The benchmark is not ScienceQA or M3CoT; the reported optimal α = 0.2 and β = 0.2
  were calibrated specifically for these datasets' architecture.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: VSSD code repository
  role: Provides requirements.txt for environment setup and the training/evaluation
    code used to run the ScienceQA benchmark at α = 0.2 and β = 0.2
  repo: https://github.com/BGWH123/VSSD
- name: pip / requirements.txt
  role: Environment setup for the VSSD training and evaluation pipeline
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:bededa54c1db8c1e557d20a9c0c3a3d09411ca86935805c583ef55f343517be0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - VSSD code repository
  - VSSD code repository (github.com/BGWH123/VSSD)
  - pip / requirements.txt
  - pip / requirements.txt environment setup
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# scienceqa-benchmark-evaluation

## Summary

Evaluates a multimodal chain-of-thought model's answer-inference accuracy on the ScienceQA benchmark, reporting overall accuracy and per-subcategory accuracies (NAT, TXT, SOC, G1–6) to compare against same-size baselines such as Enigma-COT.

## When to use

Use when a small multimodal chain-of-thought model (e.g., VSSD Base, 223M) trained on ScienceQA must be benchmarked for answer-inference accuracy overall and across NAT, TXT, SOC, and grade subcategories.

## When NOT to use

- The target task is broader multimodal reasoning beyond small multimodal CoT settings — the article explicitly defers this to future work.
- You need adaptive steering vector extraction rather than the fixed SVD-based extraction, since adaptivity is explicitly left to future work.
- The benchmark is not ScienceQA or M3CoT; the reported optimal α = 0.2 and β = 0.2 were calibrated specifically for these datasets' architecture.

## Inputs

- ScienceQA benchmark dataset (questions, images, and answer labels)
- Trained multimodal chain-of-thought model checkpoint (e.g., VSSD Base, 223M parameters)
- Steering strength α and distillation loss weight β hyperparameter settings (sweep values, optimal α = 0.2, β = 0.2)

## Outputs

- scienceqa_results.csv with overall accuracy and subcategory accuracies (NAT, TXT, SOC, G1–6)
- Per-subcategory accuracy report showing answer-inference accuracy across NAT, TXT, SOC, and G1–6
- Hyperparameter sweep curves relating α and β to overall ScienceQA accuracy

## How to apply

Set up the environment from the released VSSD repository with 'pip install -r requirements.txt'. Train the model with the two-stage chain-of-thought scheme, injecting the SVD-derived steering vector into intermediate layers at steering strength α = 0.2, and weighting the visually saliency-guided distillation loss against the main task loss with β = 0.2 — both values were established as optimal via hyperparameter sweeps. Evaluate answer-inference accuracy on ScienceQA overall and across the subcategories NAT, TXT, SOC, and G1–6, then compile overall and subcategory accuracies into scienceqa_results.csv. Validate that α = 0.2 and β = 0.2 yield the best overall accuracy among swept settings, with peak performance across most subcategories (NAT, TXT, G1–6) and no degradation on complex categories like SOC; VSSD Base (223M) should reach 89.67% overall accuracy, outperforming same-size baselines such as Enigma-COT.

## Related tools

- **VSSD code repository** (Provides requirements.txt for environment setup and the training/evaluation code used to run the ScienceQA benchmark at α = 0.2 and β = 0.2) — https://github.com/BGWH123/VSSD
- **pip / requirements.txt** (Environment setup for the VSSD training and evaluation pipeline) — https://github.com/BGWH123/VSSD

## Examples

```
git clone https://github.com/BGWH123/VSSD && cd VSSD && pip install -r requirements.txt
```

## Evaluation signals

- Overall ScienceQA accuracy at α = 0.2 and β = 0.2 should be the best among the swept hyperparameter settings.
- Peak performance should hold across most subcategories such as NAT, TXT, and G1–6.
- No performance degradation on the complex SOC category at the optimal settings (degradation occurs at α ≥ 0.3).
- VSSD Base (223M) should reach 89.67% accuracy, outperforming same-size baselines such as Enigma-COT.
- scienceqa_results.csv contains both overall and per-subcategory (NAT, TXT, SOC, G1–6) accuracies.

## Limitations

- Method is limited to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work.
- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work.
- α is sensitive: α = 0.1 gives insufficient steering signal, while α ≥ 0.3 distorts feature representations and notably degrades SOC performance.
- β must be precisely balanced; deviating from 0.2 in either direction leads to performance degradation.

## Evidence

- [methods] α = 0.2 yields the best overall accuracy on the ScienceQA dataset: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [readme] β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
- [other] VSSD Base (223M) achieves 89.67% accuracy on ScienceQA, outperforming same-size baselines such as Enigma-COT.: [evidence span withheld — anchor status: not_found]
- [readme] α significantly affects model performance... when α is too large (e.g., ≥0.3), the excessive signal distorts feature representations, notably degrading performance on complex categories like SOC: "α significantly affects model performance. Specifically, α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [abstract] Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference.: "Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference."
