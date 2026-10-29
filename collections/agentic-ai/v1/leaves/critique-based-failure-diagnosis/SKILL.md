---
name: critique-based-failure-diagnosis
description: Use when a generated annotation sample fails the CoT consistency check
  (score below threshold τ = 0.9) and you need to convert its step-by-step visual
  critique into a targeted, module-specific prompt/rubric update for the next refinement
  iteration.
when_to_use_negative:
- The sample already passes verification (consistency_score S_t ≥ τ = 0.9) — the loop
  terminates and no refinement delta should be emitted
- You need to verify annotation quality itself (bbox accuracy, CoT consistency scoring)
  — that is the Consistency Evaluation sub-task, not rubric refinement
- A single-pass annotation without iterative refinement is acceptable — the whole
  skill presumes the generate-evaluate-refine loop
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepSeek V3
  role: Prompt optimizer LLM called via DeepSeek API to analyze critiques from failed
    samples and emit the structured rubric refinement delta ΔR
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: AutoVQA-G
  role: Framework whose generate-evaluate-refine loop (pipeline.py, modules.py, prompts/system_prompts.py,
    data_structures.py) hosts iteration memory H(t), routing function π, and rubric
    updates
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier producing the critiques and consistency scores of failed samples
    that the optimizer consumes
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Python
  role: Implementation language for the iteration-memory dataclass, parsing/routing
    function π, and API calls
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:85dfa98be55bad387c97bb105a319fac8e7b5cfc35c8e798dbf2682ea72628a8
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
yes_rate: 0.875
n_claims: 8
---

# critique-based-failure-diagnosis

## Summary

Diagnose why generated VQA-G annotation samples fail verification by feeding the CoT critiques from failed samples, together with per-image iteration memory, to a prompt-optimization agent (DeepSeek V3) that identifies failure patterns and emits a targeted rubric refinement delta routed to exactly one generation module.

## When to use

Use when a generated annotation sample fails the CoT consistency check (score below threshold τ = 0.9) and you need to convert its step-by-step visual critique into a targeted, module-specific prompt/rubric update for the next refinement iteration.

## When NOT to use

- The sample already passes verification (consistency_score S_t ≥ τ = 0.9) — the loop terminates and no refinement delta should be emitted
- You need to verify annotation quality itself (bbox accuracy, CoT consistency scoring) — that is the Consistency Evaluation sub-task, not rubric refinement
- A single-pass annotation without iterative refinement is acceptable — the whole skill presumes the generate-evaluate-refine loop

## Inputs

- CoT critiques and weighted consistency scores from the two-stage Consistency Evaluation (content quality verification + bbox accuracy verification) of failed samples
- Per-image iteration memory H(t): generated drafts, verifier critiques, and current rubric sets
- Current generation rubric sets for the cap, vqa, and vg modules
- DeepSeek API credentials via the OPTIMIZER_API_KEY environment variable

## Outputs

- Structured rubric refinement delta ΔR routed to exactly one target module (cap, vqa, or vg)
- Updated rubric set and refined generation prompt for the routed module
- Updated iteration memory entry appended to H(t)

## How to apply

Load the AutoVQA-G codebase (data_structures.py, modules.py, prompts/system_prompts.py) plus the CoT critiques and consistency scores produced by the Consistency Evaluation stage. Build the per-image iteration memory H(t) (Eq. 10) recording generated drafts, verifier critiques, and current rubric sets, honoring the constraint that prompt optimization is reset per image so one sample's history does not affect the next. Call DeepSeek V3 via the DeepSeek API (OPTIMIZER_API_KEY) with a system prompt instructing it to analyze critiques from failed samples, identify failure patterns, and emit a structured refinement delta ΔR. Parse ΔR with routing function π (Eq. 11) mapping it to exactly one target module among {cap, vqa, vg}, update only that module's rubric set, append the new draft/critique/rubric state to H(t), and feed the refined prompts back into the generate-evaluate-refine loop, which terminates when the consistency score S_t ≥ τ (default 0.9) or max_iter = 5.

## Related tools

- **DeepSeek V3** (Prompt optimizer LLM called via DeepSeek API to analyze critiques from failed samples and emit the structured rubric refinement delta ΔR) — https://github.com/rohnson1999/AutoVQA-G
- **AutoVQA-G** (Framework whose generate-evaluate-refine loop (pipeline.py, modules.py, prompts/system_prompts.py, data_structures.py) hosts iteration memory H(t), routing function π, and rubric updates) — https://github.com/rohnson1999/AutoVQA-G
- **Qwen2.5-VL-72B-Instruct** (CoT verifier producing the critiques and consistency scores of failed samples that the optimizer consumes) — https://github.com/rohnson1999/AutoVQA-G
- **Python** (Implementation language for the iteration-memory dataclass, parsing/routing function π, and API calls) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
export OPTIMIZER_API_KEY="your_optimizer_api_key"; python main.py --annotations annotations. --image_dir images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Integrated into the loop with default settings (τ = 0.9, max_iter = 5), routed rubric updates drive annotations to a consistency_score field that meets τ
- Between iterations, only the routed module's rubric set changes — rubrics of the other two modules among {cap, vqa, vg} remain identical
- The refinement output is a parseable structured ΔR that maps via π to exactly one target module
- Iteration memory is reset per image: the new sample's optimization history contains no entries from the previous sample
- The loop terminates within max_iter = 5 iterations, either on S_t ≥ τ or the iteration cap

## Limitations

- Prompt optimization is reset per image, so successful rubric refinements from one sample do not transfer to the next sample
- The optimizer depends on the quality of CoT critiques produced by the Consistency Evaluation module (Qwen2.5-VL-72B-Instruct); weak critiques yield weak failure-pattern analysis
- Current computational overhead of the iterative framework is a stated limitation

## Evidence
