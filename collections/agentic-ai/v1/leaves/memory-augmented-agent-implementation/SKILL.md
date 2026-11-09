---
name: memory-augmented-agent-implementation
description: Use when building the refinement step of an iterative generate-evaluate-refine
  annotation loop where an LLM agent must analyze verifier critiques, remember past
  attempts, and apply targeted prompt/rubric updates per sample.
when_to_use_negative:
- 'You need cross-sample or cross-image learning: the memory is deliberately reset
  per image, so this skill cannot propagate refinement history from one sample to
  the next'
- 'You want to update multiple generation modules simultaneously: routing function
  π maps ΔR to exactly one target module among {cap, vqa, vg}'
- Your verification signals are simple heuristics rather than CoT critiques — the
  optimizer's input is the step-by-step visual critique from the Consistency Evaluation
  module, not raw heuristic checks
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepSeek V3
  role: Prompt optimizer LLM called via the DeepSeek API to analyze critiques from
    failed samples and emit the structured refinement delta ΔR
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: AutoVQA-G
  role: Host framework whose data_structures.py, modules.py, and prompts/system_prompts.py
    implement iteration memory H(t), routing π, and rubric refinement
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Python
  role: Implementation language for the per-image memory dataclass, ΔR parser/router,
    and API calls
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier (via DashScope) whose critiques of failed samples are the optimizer's
    input signal
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:228ea0e5861689e46ea5c4d286ec18cd62ecc58b8396914de020b9eee8908e19
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - DeepSeek
  - DeepSeek V3
  - Python
  - Qwen2.5-VL-72B-Instruct
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# memory-augmented-agent-implementation

## Summary

Implement a memory-augmented Prompt Optimization agent (as in AutoVQA-G) that keeps per-image iteration memory H(t) of drafts, critiques, and rubric sets, calls DeepSeek V3 to emit a structured refinement delta ΔR, and routes it to exactly one target module (cap, vqa, or vg) for targeted rubric updates.

## When to use

Use when building the refinement step of an iterative generate-evaluate-refine annotation loop where an LLM agent must analyze verifier critiques, remember past attempts, and apply targeted prompt/rubric updates per sample.

## When NOT to use

- You need cross-sample or cross-image learning: the memory is deliberately reset per image, so this skill cannot propagate refinement history from one sample to the next
- You want to update multiple generation modules simultaneously: routing function π maps ΔR to exactly one target module among {cap, vqa, vg}
- Your verification signals are simple heuristics rather than CoT critiques — the optimizer's input is the step-by-step visual critique from the Consistency Evaluation module, not raw heuristic checks

## Inputs

- CoT critiques and consistency scores produced by the Consistency Evaluation module (Qwen2.5-VL-72B-Instruct verifier via DashScope)
- Per-image iteration history H(t): generated drafts, verifier critiques, current rubric sets
- Current generation rubric sets for modules cap, vqa, and vg

## Outputs

- Structured refinement delta ΔR routed to exactly one target module ({cap, vqa, vg})
- Updated rubric set for the routed module, appended to H(t)
- Refined generation prompts exposed to the generate-evaluate-refine loop
- Refined outputs carrying a consistency_score field (JSON output record with image_identifier, question, answer, object_description, bbox, consistency_score, iteration, is_successful, CoT_critique, timestamp)

## How to apply

Implement iteration memory H(t) (Eq. 10) as a per-image dataclass/dict in the AutoVQA-G codebase (data_structures.py, modules.py, prompts/system_prompts.py) recording generated drafts, CoT verifier critiques, and current rubric sets, honoring the constraint that prompt optimization is reset per image so one sample's history never affects the next. Call DeepSeek V3 via the DeepSeek API using the OPTIMIZER_API_KEY environment variable, with a system prompt instructing the model to analyze critiques from failed samples, identify failure patterns, and emit a structured refinement delta ΔR. Implement the parsing/routing function π (Eq. 11) to map parsed ΔR to exactly one target module among {cap, vqa, vg} and update only that module's rubric set. Append the updated draft/critique/rubric state to H(t) and expose refined generation prompts to the loop, which terminates when the consistency score S_t ≥ τ (default τ = 0.9) or max_iter = 5 is reached.

## Related tools

- **DeepSeek V3** (Prompt optimizer LLM called via the DeepSeek API to analyze critiques from failed samples and emit the structured refinement delta ΔR) — https://github.com/rohnson1999/AutoVQA-G
- **AutoVQA-G** (Host framework whose data_structures.py, modules.py, and prompts/system_prompts.py implement iteration memory H(t), routing π, and rubric refinement) — https://github.com/rohnson1999/AutoVQA-G
- **Python** (Implementation language for the per-image memory dataclass, ΔR parser/router, and API calls) — https://github.com/rohnson1999/AutoVQA-G
- **Qwen2.5-VL-72B-Instruct** (CoT verifier (via DashScope) whose critiques of failed samples are the optimizer's input signal)

## Examples

```
export OPTIMIZER_API_KEY="your_optimizer_api_key" && python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Integrated into the loop with default settings (τ = 0.9, max_iter = 5), routed rubric updates drive annotations to pass the consistency threshold
- Refined outputs carry a consistency_score field meeting τ = 0.9
- Only the routed module's rubric changes between iterations — the other two modules' rubric sets remain unchanged
- Prompt optimization state is reset per image: one sample's refinement history does not affect the next sample
- Loop terminates correctly when S_t ≥ τ or after the maximum number of iterations (5)

## Limitations

- Current computational overhead is a stated limitation of the iterative framework
- Prompt optimization is reset per image, so no refinement knowledge is transferred across samples
- Termination may occur without success: if max_iter = 5 is reached with S_t < τ, the highest-scoring iteration is selected rather than a threshold-passing sample

## Evidence
