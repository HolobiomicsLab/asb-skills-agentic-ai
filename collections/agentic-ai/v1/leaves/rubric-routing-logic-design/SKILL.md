---
name: rubric-routing-logic-design
description: Use when implementing targeted, per-module prompt/rubric updates from
  verifier critiques in an iterative annotation or refinement loop, especially when
  updates must be isolated to a single generation module per iteration.
when_to_use_negative:
- You need simultaneous rubric updates across multiple generation modules in a single
  iteration — the routing function π is defined to update exactly one module per refinement
  step
- Your verifier feedback is scalar-only (no step-by-step critiques); without CoT critiques
  there are no failure patterns to analyze for a targeted ΔR
- You need cross-image prompt learning — optimization state is deliberately reset
  per image
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepSeek V3
  role: Prompt optimizer model called via the DeepSeek API to analyze critiques from
    failed samples and emit the structured refinement delta ΔR
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: AutoVQA-G
  role: Framework containing modules.py, data_structures.py, and prompts/system_prompts.py
    in which the routing function π and iteration memory H(t) are implemented
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Python
  role: Implementation language for the routing function, per-image memory dataclass,
    and pipeline integration (pipeline.py)
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:0fb0579743b1009698b896ad189db2f411f1a710cbf6c8288c554bb470b6d954
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - DeepSeek
  - DeepSeek V3
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# rubric-routing-logic-design

## Summary

Implements the parsing/routing function of AutoVQA-G's Prompt Optimization agent: parse the structured refinement delta (ΔR) emitted by DeepSeek V3 and route it to exactly one target module (cap, vqa, or vg) so that only that module's rubric set is updated within the generate-evaluate-refine loop.

## When to use

Use when implementing targeted, per-module prompt/rubric updates from verifier critiques in an iterative annotation or refinement loop, especially when updates must be isolated to a single generation module per iteration.

## When NOT to use

- You need simultaneous rubric updates across multiple generation modules in a single iteration — the routing function π is defined to update exactly one module per refinement step
- Your verifier feedback is scalar-only (no step-by-step critiques); without CoT critiques there are no failure patterns to analyze for a targeted ΔR
- You need cross-image prompt learning — optimization state is deliberately reset per image

## Inputs

- CoT critiques and consistency scores from the Consistency Evaluation module (Qwen2.5-VL-72B-Instruct via DashScope)
- Per-image iteration memory H(t): generated drafts, verifier critiques, and current rubric sets
- System prompt template for the Prompt Optimization agent (prompts/system_prompts.py)
- OPTIMIZER_API_KEY environment variable (DeepSeek API credentials)

## Outputs

- Structured refinement delta ΔR parsed and routed to exactly one module among {cap, vqa, vg}
- Updated rubric set for the routed module only, appended to H(t)
- Refined generation prompts fed back into the generate-evaluate-refine loop

## How to apply

Load the AutoVQA-G codebase (data_structures.py, modules.py, prompts/system_prompts.py) and the CoT critiques and consistency scores produced by the Consistency Evaluation module. Maintain per-image iteration memory H(t) (Eq. 10) recording drafts, critiques, and current rubric sets, honoring the design constraint that prompt optimization is reset per image so one sample's history does not affect the next. Call DeepSeek V3 via the DeepSeek API (OPTIMIZER_API_KEY) with a system prompt that instructs the model to analyze critiques from failed samples, identify failure patterns, and emit a structured refinement delta ΔR. Implement the parsing/routing function π (Eq. 11) that maps the parsed ΔR to exactly one target module among {cap, vqa, vg} and updates only that module's rubric set; append the updated state to H(t) and expose the refined prompts to the loop. The rationale for single-target routing is that verifier critiques diagnose a specific stage (captioning, VQA, or grounding), so confining the update prevents unrelated rubric drift.

## Related tools

- **DeepSeek V3** (Prompt optimizer model called via the DeepSeek API to analyze critiques from failed samples and emit the structured refinement delta ΔR) — https://github.com/rohnson1999/AutoVQA-G
- **AutoVQA-G** (Framework containing modules.py, data_structures.py, and prompts/system_prompts.py in which the routing function π and iteration memory H(t) are implemented) — https://github.com/rohnson1999/AutoVQA-G
- **Python** (Implementation language for the routing function, per-image memory dataclass, and pipeline integration (pipeline.py)) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
export OPTIMIZER_API_KEY="your_optimizer_api_key"; python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Integrated into the loop with default settings (consistency threshold τ = 0.9, max_iter = 5), routed rubric updates drive annotations to pass the consistency threshold
- Refined outputs carry a consistency_score field meeting τ (e.g. the README example shows consistency_score 0.92 with is_successful true)
- Diffing rubric state between iterations shows that only the routed module's rubric changed
- Each parsed ΔR maps to exactly one target module among {cap, vqa, vg} with no unmapped or multi-target updates
- H(t) is reset between images, so a new sample's first iteration carries no rubric history from the previous sample

## Limitations

- Prompt optimization is reset per image, so one sample's refinement history does not affect the next sample — no cumulative cross-sample learning
- The loop only terminates on success when S_t ≥ τ; otherwise it stops at max_iter = 5, so persistently failing images fall back to the highest-scoring iteration
- Current computational overhead is a stated limitation of the iterative refinement approach

## Evidence
