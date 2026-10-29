---
name: scienceqa-accuracy-rougel-evaluation
description: Use when tuning the steering-vector layer depth L of the VSSD distillation
  pipeline and you need paired accuracy/RougeL curves on ScienceQA to decide which
  decoder layers to inject into.
when_to_use_negative:
- When tuning a different hyperparameter — for α or β sweeps use the dedicated alpha/beta
  analysis plots (all_vs_alpha.png, beta_vs_accuracy.png) instead
- When the target benchmark is not ScienceQA (e.g., M3CoT), since the optimal L=2
  finding was validated on ScienceQA
- When evaluating a model that does not inject SVD steering vectors into intermediate
  decoder layers — the L sweep has no meaning without steering-vector injection
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Implements the distillation runs and evaluation loop for each L setting
  repo: https://github.com/BGWH123/VSSD
- name: PyTorch
  role: Runs the VSSD model with SVD steering vectors injected into the selected last-L
    decoder layers
  repo: https://github.com/BGWH123/VSSD
- name: matplotlib
  role: Plots accuracy and RougeL against L as line curves for layer_vs_accuracy.png
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:9186a2f818d2df161b2f9dc903a69c24f6abe45f092889cd38c11a0c9de575fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - PyTorch
  - Python
  - matplotlib
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# scienceqa-accuracy-rougel-evaluation

## Summary

Sweeps the number of last decoder layers L from which VSSD's SVD-derived steering vectors are extracted, and evaluates answer accuracy plus RougeL over generated chain-of-thought rationales on ScienceQA to locate the optimal L.

## When to use

Use when tuning the steering-vector layer depth L of the VSSD distillation pipeline and you need paired accuracy/RougeL curves on ScienceQA to decide which decoder layers to inject into.

## When NOT to use

- When tuning a different hyperparameter — for α or β sweeps use the dedicated alpha/beta analysis plots (all_vs_alpha.png, beta_vs_accuracy.png) instead
- When the target benchmark is not ScienceQA (e.g., M3CoT), since the optimal L=2 finding was validated on ScienceQA
- When evaluating a model that does not inject SVD steering vectors into intermediate decoder layers — the L sweep has no meaning without steering-vector injection

## Inputs

- ScienceQA benchmark (images, questions, answers, subject subsets)
- VSSD distillation pipeline checkpoint with SVD steering-vector extraction
- Fixed hyperparameters α=0.2 and β=0.2
- Generated chain-of-thought rationales per L setting

## Outputs

- layer_sweep_results.csv (accuracy and RougeL per L value, overall and per subject subset)
- layer_vs_accuracy.png (line curves of accuracy and RougeL vs L in the Fig. 3(a) style)

## How to apply

Clone the VSSD repository (BGWH123/VSSD) and install the environment with 'pip install -r requirements.txt'. Fix the already-optimized hyperparameters α=0.2 (steering injection strength) and β=0.2 (distillation loss weight) so the sweep isolates the effect of L. For each setting (L=1, L=2, L>2), run the saliency-guided distillation with SVD steering vectors injected into the selected last-L decoder layers, then compute answer accuracy on ScienceQA overall and per subject subset (NAT, TXT, SOC, NO, G1–6) and RougeL on the generated chain-of-thought rationales. Tabulate results into layer_sweep_results.csv and plot accuracy and RougeL against L as line curves in the paper's Fig. 3(a) style (layer_vs_accuracy.png). Validate that both curves peak at L=2: with L=1 the steering signal is too weak, and with L>2 earlier layers contribute redundant or noisy information.

## Related tools

- **Python** (Implements the distillation runs and evaluation loop for each L setting) — https://github.com/BGWH123/VSSD
- **PyTorch** (Runs the VSSD model with SVD steering vectors injected into the selected last-L decoder layers) — https://github.com/BGWH123/VSSD
- **matplotlib** (Plots accuracy and RougeL against L as line curves for layer_vs_accuracy.png)

## Examples

```
git clone https://github.com/BGWH123/VSSD && pip install -r requirements.txt
```

## Evaluation signals

- Both accuracy and RougeL curves peak at L=2, matching Fig. 3(a) of the paper
- L=1 underperforms L=2 (steering signal too weak to activate fine-grained discriminative capability)
- L>2 degrades relative to L=2 (earlier layers inject redundant/noisy information)
- layer_sweep_results.csv contains one accuracy and one RougeL row per L setting, overall plus per-subject (NAT, TXT, SOC, NO, G1–6)
- α and β remain fixed at 0.2 across all runs so the sweep is attributable to L alone

## Limitations

- Steering-vector extraction is not adaptive; the paper defers more adaptive extraction to future work
- VSSD is limited to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work
- The README documents only the α and β sweeps, so exact L-sweep numbers must be reproduced from the repository rather than read from documentation

## Evidence

- [readme] α = 0.2 yields the best overall accuracy on the ScienceQA dataset: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [readme] β balances losses: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
- [abstract] VSSD improves rationale generation: "Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference."
- [readme] alpha controls steering injection strength: "α controls the strength with which the steering vector—derived from the feature difference between the original and perturbed images—is injected into the model's intermediate layers"
