---
name: prompt-optimization-refinement
description: Use when automated annotation generation produces samples below the consistency
  threshold and you need to iteratively refine generation prompts from CoT verification
  critiques rather than discard the failures.
when_to_use_negative:
- When running the no-routing or no-memory ablation configurations of AutoVQA-G, where
  these components are intentionally disabled to measure their contribution
- When a single-pass annotation already passes the consistency threshold (S_t >= 0.9),
  since invoking the optimizer adds computational overhead without benefit
- When annotating a task outside VQA-G (visual question answering with grounding),
  as the rubric sections and bbox accuracy verification are specific to this format
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: DeepSeek V3
  role: Prompt optimizer that analyzes verification critiques and dynamically refines
    generation prompts via the DeepSeek API (OPTIMIZER_API_KEY)
  repo: github.com/rohnson1999/AutoVQA-G
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier whose step-by-step visual critiques supply the failure signals
    the optimizer uses for rubric updates
  repo: github.com/rohnson1999/AutoVQA-G
- name: AutoVQA-G
  role: Framework implementing the generate-evaluate-refine loop in which memory-augmented
    prompt optimization runs (pipeline.py, modules.py)
  repo: github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server that regenerates caption, VQA, and mention outputs
    using the optimized prompts
  repo: github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:a0c7e9337e1144483786d97e10dc81a0fb38176345a2c940f461296aa6dc5caa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - AutoVQA-G
  - DeepSeek
  - DeepSeek V3
  - GroundingDINO
  - MiniCPM-o
  - MiniCPM-o 2.6
  - Qwen2.5-VL-72B-Instruct
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# prompt-optimization-refinement

## Summary

Memory-augmented prompt optimization in AutoVQA-G: an agent (DeepSeek V3) analyzes CoT critiques from failed VQA-G samples, maintains iteration history, and dynamically routes rubric updates to refine generation prompts until the consistency score reaches the acceptance threshold (tau=0.9).

## When to use

Use when automated annotation generation produces samples below the consistency threshold and you need to iteratively refine generation prompts from CoT verification critiques rather than discard the failures.

## When NOT to use

- When running the no-routing or no-memory ablation configurations of AutoVQA-G, where these components are intentionally disabled to measure their contribution
- When a single-pass annotation already passes the consistency threshold (S_t >= 0.9), since invoking the optimizer adds computational overhead without benefit
- When annotating a task outside VQA-G (visual question answering with grounding), as the rubric sections and bbox accuracy verification are specific to this format

## Inputs

- CoT critiques and verifiable quality scores from the Consistency Evaluation module (content quality verification + bbox accuracy verification)
- Iteration history of previously refined generation prompts and their failure patterns
- Generation prompt templates from prompts/system_prompts.py
- JSON annotations file with an 'image' key per record, plus the image directory

## Outputs

- Dynamically refined generation rubrics/prompts for the MiniCPM-o 2.6 generation server
- Per-image selected result with consistency_score, iteration index, is_successful flag, and CoT_critique in the output JSON
- Logs of prompt refinements written to output_dir

## How to apply

Wire the Prompt Optimization agent (DeepSeek V3 via the OPTIMIZER_API_KEY / DeepSeek API) into the generate-evaluate-refine loop implemented in pipeline.py, so that each failed evaluation feeds the CoT critiques (from the Qwen2.5-VL-72B-Instruct Consistency Evaluation module) back into prompt refinement. Maintain a memory of past iteration attempts per image — the README notes 'Prompt optimization is reset per image, so one sample's refinement history does not affect the next sample' — and use dynamic routing to apply targeted rubric updates only to the rubric sections implicated by the failure pattern. Run the loop with --threshold 0.9 and --max_iter 5; accept the first sample whose consistency score S_t >= tau, otherwise keep the highest-scoring iteration. Ablation evidence shows removing memory or dynamic routing degrades grounding quality (full-model mIoU 0.634 vs. 0.582 without memory and 0.561 without routing), so both components should stay enabled for full fidelity.

## Related tools

- **DeepSeek V3** (Prompt optimizer that analyzes verification critiques and dynamically refines generation prompts via the DeepSeek API (OPTIMIZER_API_KEY)) — github.com/rohnson1999/AutoVQA-G
- **Qwen2.5-VL-72B-Instruct** (CoT verifier whose step-by-step visual critiques supply the failure signals the optimizer uses for rubric updates) — github.com/rohnson1999/AutoVQA-G
- **AutoVQA-G** (Framework implementing the generate-evaluate-refine loop in which memory-augmented prompt optimization runs (pipeline.py, modules.py)) — github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server that regenerates caption, VQA, and mention outputs using the optimized prompts) — github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Full-model run on Visual7W reproduces VQAScore 0.896 and mIoU 0.634; removing memory drops mIoU to 0.582 and removing dynamic routing to 0.561, so a correctly configured optimizer should keep results at the full-model level
- Selected outputs contain consistency_score >= 0.9 (threshold tau) or the highest-scoring iteration when no sample passes within max_iter=5
- Optimization memory is reset per image: refinement history from one sample must not leak into the next sample's prompts
- Output JSON records the iteration index at which the accepted sample was produced, showing the prompt actually changed across iterations rather than staying static

## Limitations

- Current computational overhead is a stated limitation of the framework
- Prompt optimization is reset per image, so no cross-image learning accumulates
- The loop caps refinement at --max_iter 5 per image; samples never reaching the 0.9 threshold fall back to their best iteration, which may still be low fidelity
- No changelog available for the repository

## Evidence

- [other] Ablation reference values: [evidence span withheld — anchor status: not_found]
