---
name: baseline-comparison-table-assembly
description: Use when reproducing VSSD-Base results on M3CoT and you must verify the
  reported 73.19% total average accuracy and its ity over fine-tuned baselines such
  as MM-CoT-Large (48.73) and MC-CoT-Large (57.69).
when_to_use_negative:
- The goal is hyperparameter calibration (sweeping α or β) rather than evaluating
  a single configured model against baselines
- The evaluation is on ScienceQA subcategories (NAT, TXT, SOC, G1–6) without the M3CoT
  benchmark and Table 2 baseline numbers
- No reproduced model outputs are available and only the article's reported numbers
  are to be cited without any reproduction
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Runs M3CoT answer inference with the VSSD-Base checkpoint, computes exact-match
    accuracy, and writes results_table.csv
- name: BGWH123/VSSD
  role: Source of the VSSD-Base checkpoint and environment (requirements.txt) needed
    to reproduce inference on M3CoT
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:a28027b05291f6353c38271042ebf04a77ef3e206f98c1a52cd00154613faea5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - BGWH123/VSSD
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# baseline-comparison-table-assembly

## Summary

Assembles a benchmark comparison table that places a reproduced VSSD-Base total average accuracy on M3CoT (exact-match answer inference) next to the reported fine-tuned baselines (MM-CoT-Large, MC-CoT-Large, LLaVA, CogVLM) from Table 2, enabling validation of reproduction fidelity and outperformance claims.

## When to use

Use when reproducing VSSD-Base results on M3CoT and you must verify the reported 73.19% total average accuracy and its superiority over fine-tuned baselines such as MM-CoT-Large (48.73) and MC-CoT-Large (57.69).

## When NOT to use

- The goal is hyperparameter calibration (sweeping α or β) rather than evaluating a single configured model against baselines
- The evaluation is on ScienceQA subcategories (NAT, TXT, SOC, G1–6) without the M3CoT benchmark and Table 2 baseline numbers
- No reproduced model outputs are available and only the article's reported numbers are to be cited without any reproduction

## Inputs

- M3CoT benchmark test split
- Trained VSSD-Base checkpoint from the released VSSD repository
- Reported fine-tuned baseline accuracies from Table 2 (MM-CoT-Large 48.73, MC-CoT-Large 57.69, LLaVA, CogVLM)

## Outputs

- results_table.csv containing accuracy entries for VSSD-Base and all four fine-tuned baselines
- Reproduced total average accuracy (exact-match) for VSSD-Base on M3CoT

## How to apply

Load the M3CoT test split and the VSSD-Base checkpoint from the released BGWH123/VSSD repository (set up with the repo's requirements.txt). Run multimodal chain-of-thought answer inference with VSSD-Base over all M3CoT examples and record predicted answers. Compute total average accuracy as exact-match of predicted answers across the benchmark categories. Assemble a comparison table placing the measured VSSD-Base accuracy beside the reported fine-tuned baseline accuracies from Table 2 (MM-CoT-Large, MC-CoT-Large, LLaVA, CogVLM) and save it as results_table.csv. The article's hyperparameter context (α = 0.2 and β = 0.2 for steering strength and distillation loss weighting) should be respected if re-running training before comparison.

## Related tools

- **Python** (Runs M3CoT answer inference with the VSSD-Base checkpoint, computes exact-match accuracy, and writes results_table.csv)
- **BGWH123/VSSD** (Source of the VSSD-Base checkpoint and environment (requirements.txt) needed to reproduce inference on M3CoT) — https://github.com/BGWH123/VSSD

## Examples

```
pip install -r requirements.txt  # then run VSSD-Base inference over M3CoT and write results_table.csv
```

## Evaluation signals

- Reproduced VSSD-Base total average accuracy on M3CoT matches the reported 73.19% within ±1% tolerance
- results_table.csv contains accuracy entries for VSSD-Base and all four fine-tuned baselines (MM-CoT-Large, MC-CoT-Large, LLaVA, CogVLM)
- Baseline values in the table match Table 2 exactly (e.g., MM-CoT-Large 48.73, MC-CoT-Large 57.69)
- Accuracy is computed as exact-match of predicted answers aggregated across all M3CoT benchmark categories
- Reproduced VSSD-Base accuracy exceeds every fine-tuned baseline entry in the table

## Limitations

- The reproduced accuracy is validated within a ±1% tolerance, so small deviations from 73.19% are expected
- The discussion section reports only qualitative claims ('consistent improvements') with no quantitative results, hyperparameters, or model versions, so validation relies on the paper's Table 2 values
- Steering vector extraction is not adaptive and the method is scoped to small multimodal CoT settings, limiting the generality of conclusions drawn from this comparison
- Hyperparameters α = 0.2 and β = 0.2 were established as optimal on ScienceQA, not M3CoT, so reproduced M3CoT runs inherit these settings

## Evidence

- [abstract] Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference.: "Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference."
- [readme] alpha optimum on ScienceQA: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset, along with peak performance across most subcategories such as NAT, TXT, and G1–6"
- [readme] beta optimum: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss within the current architecture"
- [discussion] discussion quantitative gap: "Experiments on ScienceQA and M3CoT show
consistent improvements"
