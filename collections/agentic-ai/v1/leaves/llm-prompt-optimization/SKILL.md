---
name: llm-prompt-optimization
description: Use when an iterative generate-evaluate-refine annotation loop needs
  targeted prompt/rubric updates driven by verifier critiques, with per-image iteration
  memory and dynamic routing of refinements.
when_to_use_negative:
- The sample already passes the consistency threshold (S_t ≥ τ), so no rubric refinement
  is needed and the loop should terminate
- The task is single-pass annotation without a verification/critique signal — the
  optimizer requires CoT critiques from failed samples as input
- Cross-sample history transfer is desired — the design explicitly resets prompt optimization
  per image so one sample's refinement history does not affect the next
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: DeepSeek V3
  role: Prompt optimizer LLM invoked via the DeepSeek API to analyze critiques, identify
    failure patterns, and emit structured refinement deltas ΔR
- name: AutoVQA-G
  role: Framework providing the generate-evaluate-refine pipeline, iteration memory
    data structures (data_structures.py), routing logic (modules.py), and system prompt
    templates (prompts/system_prompts.py)
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Python
  role: Implementation language for the optimizer module, memory H(t), and the parsing/routing
    function π
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:8e315ddb6aef087a14887193c2bf130ef5fd61e79d8521d22a1438f87157984f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - DeepSeek
  - DeepSeek V3
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# llm-prompt-optimization

## Summary

Implement the memory-augmented Prompt Optimization agent of AutoVQA-G, which analyzes CoT critiques from failed samples and emits targeted rubric refinement deltas ΔR that are routed to exactly one generation module (cap, vqa, vg) per iteration.

## When to use

Use when an iterative generate-evaluate-refine annotation loop needs targeted prompt/rubric updates driven by verifier critiques, with per-image iteration memory and dynamic routing of refinements.

## When NOT to use

- The sample already passes the consistency threshold (S_t ≥ τ), so no rubric refinement is needed and the loop should terminate
- The task is single-pass annotation without a verification/critique signal — the optimizer requires CoT critiques from failed samples as input
- Cross-sample history transfer is desired — the design explicitly resets prompt optimization per image so one sample's refinement history does not affect the next

## Inputs

- CoT critiques and consistency scores from the Consistency Evaluation module (per image)
- Per-image iteration memory H(t): generated drafts, verifier critiques, current rubric sets
- Current rubric sets for the three generation modules {cap, vqa, vg}

## Outputs

- Structured refinement delta ΔR parsed from the DeepSeek V3 response
- Updated rubric set for exactly one routed target module (cap, vqa, or vg)
- Refined generation prompts exposed to the generate-evaluate-refine loop

## How to apply

Load the AutoVQA-G codebase (data_structures.py, modules.py, prompts/system_prompts.py) and the CoT critiques and consistency scores produced by the Consistency Evaluation module. Implement the iteration memory H(t) (Eq. 10) as a per-image dataclass/dict recording generated drafts, verifier critiques, and current rubric sets, honoring the constraint that prompt optimization is reset per image so one sample's history does not affect the next. Call DeepSeek V3 via the DeepSeek API (configured through the OPTIMIZER_API_KEY environment variable) with a system prompt template that instructs the model to analyze critiques from failed samples, identify failure patterns, and emit a structured refinement delta ΔR. Implement the parsing/routing function π (Eq. 11) that maps the parsed ΔR to exactly one target module among {cap, vqa, vg} and updates only that module's rubric set, then append the updated draft/critique/rubric state to H(t) and expose the refined prompts back to the generate-evaluate-refine loop, which terminates when the consistency score S_t ≥ τ (default τ = 0.9) or max_iter = 5 is reached.

## Related tools

- **DeepSeek V3** (Prompt optimizer LLM invoked via the DeepSeek API to analyze critiques, identify failure patterns, and emit structured refinement deltas ΔR)
- **AutoVQA-G** (Framework providing the generate-evaluate-refine pipeline, iteration memory data structures (data_structures.py), routing logic (modules.py), and system prompt templates (prompts/system_prompts.py)) — https://github.com/rohnson1999/AutoVQA-G
- **Python** (Implementation language for the optimizer module, memory H(t), and the parsing/routing function π)

## Examples

```
export OPTIMIZER_API_KEY="your_optimizer_api_key"; python main.py --annotations path/to/annotations. --image_dir path/to/images/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Integrated into the loop with default settings (τ = 0.9, max_iter = 5), routed rubric updates drive annotations to pass the consistency threshold
- Refined outputs carry a consistency_score field meeting the threshold τ = 0.9
- Only the routed module's rubric changes between iterations — the routing function π updates exactly one of {cap, vqa, vg}
- Iteration memory H(t) is reset per image, so no sample's draft/critique/rubric history leaks into the next sample
- The loop terminates when S_t ≥ τ or after the maximum number of iterations (max_iter = 5) is reached

## Limitations

- Prompt optimization is reset per image, so refinement history cannot be shared across samples
- The routing function π maps each ΔR to exactly one target module, so a single refinement cannot update multiple module rubrics simultaneously
- Computational overhead is a stated limitation of the iterative framework
- The optimizer is tied to the DeepSeek V3 API via OPTIMIZER_API_KEY; model or endpoint changes must be made in modules.py

## Evidence
