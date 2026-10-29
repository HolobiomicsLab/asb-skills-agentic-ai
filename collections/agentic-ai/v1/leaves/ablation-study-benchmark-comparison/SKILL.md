---
name: ablation-study-benchmark-comparison
description: Use when you need to verify or extend a published framework's ablation
  table by re-running full and component-removed configurations on the same dataset
  and comparing metric deltas per component.
when_to_use_negative:
- The goal is to benchmark AutoVQA-G against external baselines (e.g., leading multimodal
  LLMs) rather than to decompose the framework's own components — that requires a
  different cross-model comparison protocol
- The input annotations already carry verified ground-truth grounding; the ablation
  loop exists to measure annotation quality, not to score pre-existing data
- The task targets a different dataset than Visual7W without re-deriving reference
  VQAScore/mIoU values, since the Table 3 tolerance checks are dataset-specific
edam_operation: http://edamontology.org/operation_0336
tools:
- name: AutoVQA-G
  role: Framework under ablation; its generate-evaluate-refine loop is toggled per
    configuration
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server (localhost:8000/v1) for caption, VQA, and mention
    generation in every run
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier via DashScope; disabled in ablation B (score-only verification)
- name: DeepSeek V3
  role: Prompt optimizer via DeepSeek API; routing (ablation C) and memory (ablation
    D) are removed from its operation
- name: GroundingDINO
  role: Visual grounding endpoint (localhost:8001/predict) supplying bbox detections
    for mIoU computation
- name: Python
  role: Execution environment; runs main.py with --threshold 0.9 and --max_iter 5
    for each configuration
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:04c0300b98929e24e2fb1192f8ab84620e9b1353f95964536849f1803d6bd942
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - DeepSeek
  - DeepSeek V3
  - GroundingDINO
  - MiniCPM-o
  - MiniCPM-o 2.6
  - Python
  - Qwen2.5-VL-72B-Instruct
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# ablation-study-benchmark-comparison

## Summary

Reproduce a paper's ablation study (Visual7W Table 3) by running the full AutoVQA-G generate-evaluate-refine pipeline alongside four degraded configurations (single-pass, no-CoT, no-routing, no-memory) and comparing VQAScore and mIoU against reference values to quantify each component's contribution.

## When to use

Use when you need to verify or extend a published framework's ablation table by re-running full and component-removed configurations on the same dataset and comparing metric deltas per component.

## When NOT to use

- The goal is to benchmark AutoVQA-G against external baselines (e.g., leading multimodal LLMs) rather than to decompose the framework's own components — that requires a different cross-model comparison protocol
- The input annotations already carry verified ground-truth grounding; the ablation loop exists to measure annotation quality, not to score pre-existing data
- The task targets a different dataset than Visual7W without re-deriving reference VQAScore/mIoU values, since the Table 3 tolerance checks are dataset-specific

## Inputs

- Visual7W dataset split (images + annotations JSON with an "image" key per entry)
- AutoVQA-G repository (main.py, pipeline.py, modules.py, prompts/system_prompts.py)
- Configured model endpoints: MiniCPM-o 2.6, Qwen2.5-VL-72B-Instruct (DashScope), DeepSeek V3, GroundingDINO

## Outputs

- Full-model VQAScore and mIoU (reference: 0.896 / 0.634)
- Per-ablation mIoU values for single-pass (0.380), no-CoT (0.495), no-routing (0.561), no-memory (0.582)
- Ablation comparison table with per-image consistency scores and selected annotations per configuration

## How to apply

Clone github.com/rohnson1999/AutoVQA-G and configure the four endpoints in modules.py: MiniCPM-o 2.6 generation server (localhost:8000/v1), Qwen2.5-VL-72B-Instruct CoT verifier via DashScope, DeepSeek V3 prompt optimizer, and GroundingDINO (localhost:8001/predict). Run the full pipeline on the Visual7W split via main.py with --threshold 0.9 and --max_iter 5, recording VQAScore and mIoU (reference: 0.896 / 0.634). Then run four ablations: (A) single-pass generation without the generate-evaluate-refine loop (ref. mIoU 0.380), (B) score-only verification with CoT critique disabled (ref. 0.495), (C) prompt optimization without dynamic routing (ref. 0.561), (D) prompt optimization without memory of past iterations (ref. 0.582). The rationale is that each ablation isolates exactly one framework component — the iterative loop, CoT verification, dynamic routing, or memory — so the ordering full model > no-memory > no-routing > no-CoT > single-pass quantifies marginal contribution. Compile per-image consistency scores and selected annotations from each run into a single comparison table with the Table 3 values as a reference column.

## Related tools

- **AutoVQA-G** (Framework under ablation; its generate-evaluate-refine loop is toggled per configuration) — https://github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server (localhost:8000/v1) for caption, VQA, and mention generation in every run)
- **Qwen2.5-VL-72B-Instruct** (CoT verifier via DashScope; disabled in ablation B (score-only verification))
- **DeepSeek V3** (Prompt optimizer via DeepSeek API; routing (ablation C) and memory (ablation D) are removed from its operation)
- **GroundingDINO** (Visual grounding endpoint (localhost:8001/predict) supplying bbox detections for mIoU computation)
- **Python** (Execution environment; runs main.py with --threshold 0.9 and --max_iter 5 for each configuration)

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Full-model run reproduces VQAScore 0.896 and mIoU 0.634 on the Visual7W split
- Each ablation run's mIoU falls within a reasonable tolerance of its Table 3 reference (0.380, 0.495, 0.561, 0.582)
- Component-removal ordering matches the reported ranking: full model > no-memory > no-routing > no-CoT > single-pass
- Single-pass ablation (A) shows the largest drop (mIoU 0.634 → 0.380), confirming the refinement loop dominates
- Output JSON per run contains one selected result per image with consistency_score ≥ threshold 0.9 or the highest-scoring iteration when max_iter 5 is exhausted

## Limitations

- Reference values are dataset-specific to Visual7W; tolerances and ordering claims do not transfer to other splits without re-derivation
- Computational overhead is a stated limitation of the framework, and running five full configurations multiplies this cost
- Prompt optimization is reset per image, so ablation D (no-memory) removes cross-iteration memory within a sample, not cross-sample effects

## Evidence

- [abstract] iterative refinement loop: "an iterative refinement loop where a Consistency Evaluation module uses Chain-of-Thought (CoT) reasoning for fine-grained visual verification"
- [readme] loop termination threshold: "The loop terminates when the consistency score `S_t ≥ τ` or the maximum number of iterations is reached."
- [methods] two-stage verification: "Two-stage assessment — content quality verification + bbox accuracy verification"
- [readme] memory reset per image: "Prompt optimization is reset per image, so one sample's refinement history does not affect the next sample."
