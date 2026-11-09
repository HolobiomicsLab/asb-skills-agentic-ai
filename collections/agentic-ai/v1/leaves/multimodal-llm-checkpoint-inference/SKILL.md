---
name: multimodal-llm-checkpoint-inference
description: Use when a trained multimodal CoT checkpoint and a benchmark test split
  (M3CoT or ScienceQA) are available and the goal is to measure answer accuracy or
  reproduce a reported baseline/leaderboard number.
when_to_use_negative:
- The goal is training or hyperparameter tuning (α/β analysis) rather than evaluating
  an already-trained checkpoint
- The task requires rationale/chain-of-thought text quality evaluation rather than
  exact-match answer accuracy
- The input is a unimodal text-only benchmark, since VSSD targets multimodal visual-semantic
  reasoning
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Python
  role: Drives checkpoint loading, multimodal chain-of-thought answer inference over
    M3CoT examples, and exact-match accuracy computation
- name: VSSD
  role: Source of the pretrained VSSD-Base checkpoint, requirements.txt environment
    definition, and inference code
  repo: https://github.com/BGWH123/VSSD
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/icassp55912.2026.11463006
    title: Visual Saliency Steering Distillation for Multimodal Chain-of-Thought Reasoning
  source_content_hash: sha256:13f310ff54108cdd2b579240d4a82b00607c89eb3aabe8e8e1401ef643ff51fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/VSSD
  tools:
  - Python
  - VSSD
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# multimodal-llm-checkpoint-inference

## Summary

Runs answer inference with a trained multimodal chain-of-thought checkpoint (e.g., VSSD-Base from the BGWH123/VSSD release) over a benchmark test split such as M3CoT, recording predicted answers and computing exact-match total average accuracy to reproduce reported results (73.19% on M3CoT).

## When to use

Use when a trained multimodal CoT checkpoint and a benchmark test split (M3CoT or ScienceQA) are available and the goal is to measure answer accuracy or reproduce a reported baseline/leaderboard number.

## When NOT to use

- The goal is training or hyperparameter tuning (α/β analysis) rather than evaluating an already-trained checkpoint
- The task requires rationale/chain-of-thought text quality evaluation rather than exact-match answer accuracy
- The input is a unimodal text-only benchmark, since VSSD targets multimodal visual-semantic reasoning

## Inputs

- Trained VSSD-Base checkpoint from the released BGWH123/VSSD repository
- M3CoT benchmark test split (or ScienceQA test split) with images, questions, and reference answers
- Python environment with the repository's requirements.txt dependencies
- Reported baseline accuracy table (Table 2: MM-CoT-Large 48.73, MC-CoT-Large 57.69, LLaVA, CogVLM) for comparison

## Outputs

- Per-example predicted answers recorded for all M3CoT examples
- Total average accuracy (exact-match) across benchmark categories
- results_table.csv comparing measured VSSD-Base accuracy against fine-tuned baselines

## How to apply

Load the M3CoT benchmark test split and the trained VSSD-Base checkpoint from the released VSSD repository (https://github.com/BGWH123/VSSD) after installing the environment via 'pip install -r requirements.txt'. Run multimodal chain-of-thought answer inference with VSSD-Base over all M3CoT examples, recording predicted answers per example. Compute total average accuracy as the exact match of predicted answers across the benchmark categories, then assemble a comparison table placing the measured VSSD-Base accuracy next to reported fine-tuned baseline accuracies (MM-CoT-Large at 48.73, MC-CoT-Large at 57.69, LLaVA, CogVLM) and save as results_table.csv. Validation tolerance is ±1% around the reported 73.19% average accuracy for VSSD-Base on M3CoT. If reproducing ScienceQA instead, the optimal hyperparameters from the paper's analysis are α = 0.2 (steering strength) and β = 0.2 (distillation loss weighting), which should match the checkpoint's training configuration.

## Related tools

- **Python** (Drives checkpoint loading, multimodal chain-of-thought answer inference over M3CoT examples, and exact-match accuracy computation)
- **VSSD** (Source of the pretrained VSSD-Base checkpoint, requirements.txt environment definition, and inference code) — https://github.com/BGWH123/VSSD

## Evaluation signals

- Reproduced VSSD-Base total average accuracy on M3CoT matches the reported 73.19% within ±1%
- results_table.csv contains accuracy entries for VSSD-Base and all four fine-tuned baselines (MM-CoT-Large, MC-CoT-Large, LLaVA, CogVLM)
- Predicted answers are recorded for every M3CoT test example with no missing predictions
- Accuracy is computed as exact match of predicted answers and aggregated across all benchmark categories
- Environment resolves cleanly from the repository's requirements.txt before inference

## Limitations

- Method is limited to small multimodal CoT settings; broader multimodal reasoning tasks are left to future work
- Steering vector extraction is not yet adaptive; more adaptive extraction is deferred to future work
- Hyperparameters are dataset-specific: α = 0.2 and β = 0.2 are optimal for ScienceQA; values that are too small (0.1) give insufficient steering signal, and values ≥0.3 distort feature representations, notably degrading SOC-category performance

## Evidence

- [abstract] Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference.: "Experiments on ScienceQA and M3CoT demonstrate that VSSD improves rationale generation and answer inference."
- [readme] alpha optimum: "α = 0.2 yields the best overall accuracy on the ScienceQA dataset"
- [abstract] code availability: "The code is available at https://github.com/BGWH123/VSSD."
- [readme] beta balance rationale: "β = 0.2 effectively balances the main task loss and the visually saliency-guided distillation loss"
