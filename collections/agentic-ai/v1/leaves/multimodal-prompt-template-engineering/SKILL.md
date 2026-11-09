---
name: multimodal-prompt-template-engineering
description: Use when building automated VQA-G or similar multimodal annotation pipelines
  where single-pass VLM generation is hallucination-prone and prompts must be iteratively
  refined from verifier feedback until a consistency score threshold is met.
when_to_use_negative:
- Your task is a single-pass annotation where heuristic verification suffices and
  the computational overhead of an iterative refine loop is not justified (the paper
  itself states current computational overhead is a limitation)
- Input is already a validated, human-annotated VQA-G dataset requiring no prompt-driven
  draft generation
edam_operation: http://edamontology.org/operation_0306
tools:
- name: DeepSeek V3
  role: 'Prompt optimizer (OPTIMIZER_API_KEY): analyzes critiques from failed samples
    and refines generation rubrics with memory of past attempts and dynamic routing'
  repo: github.com/rohnson1999/AutoVQA-G
- name: Qwen2.5-VL-72B-Instruct
  role: 'CoT verifier (VERIFICATION_API_KEY): produces step-by-step visual critiques
    and weighted consistency scores that drive prompt refinement'
  repo: github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server whose caption, VQA, and mention generation prompts
    are the targets of template refinement
  repo: github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:850ad91a1eac7fc6af55e95a81348c05422448baaff2a79afa657c6434a9cb7d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - AutoVQA-G
  - DeepSeek V3
  - GroundingDINO
  - MiniCPM-o
  - MiniCPM-o 2.6
  - Qwen2.5-VL-72B-Instruct
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# multimodal-prompt-template-engineering

## Summary

Engineers and iteratively refines prompt templates that drive a multimodal generate-evaluate-refine annotation loop (as in AutoVQA-G), using memory-augmented prompt optimization against CoT-based consistency critiques until annotations pass an acceptance threshold (tau = 0.9).

## When to use

Use when building automated VQA-G or similar multimodal annotation pipelines where single-pass VLM generation is hallucination-prone and prompts must be iteratively refined from verifier feedback until a consistency score threshold is met.

## When NOT to use

- Your task is a single-pass annotation where heuristic verification suffices and the computational overhead of an iterative refine loop is not justified (the paper itself states current computational overhead is a limitation)
- Input is already a validated, human-annotated VQA-G dataset requiring no prompt-driven draft generation

## Inputs

- JSON annotations file (list of dicts, each with at least an 'image' key)
- Image directory of input images (JPG paths resolved against --image_dir)
- Prompt templates in prompts/system_prompts.py
- CoT critiques and weighted consistency scores from the Consistency Evaluation module
- Environment configuration: GENERATION_API_KEY, VERIFICATION_API_KEY, OPTIMIZER_API_KEY

## Outputs

- Refined generation rubrics/prompts with per-iteration memory
- Per-image draft VQA-G records (question, answer, object_description, bbox, consistency_score, iteration, is_successful, CoT_critique, timestamp) written as a single JSON file into output_dir

## How to apply

Define per-module prompt templates (Caption Reasoning, VQA Generation, Object Mention) in a centralized prompts/system_prompts.py file and issue them to a local OpenAI-compatible generation server (MiniCPM-o 2.6 at http://localhost:8000/v1). Run the CoT Consistency Evaluation module (Qwen2.5-VL-72B-Instruct via VERIFICATION_API_KEY) which performs two-stage assessment — content quality verification plus bbox accuracy verification — producing weighted consistency scores and step-by-step visual critiques. Feed critiques from failed samples (score S_t < tau, with tau = 0.9) to a memory-augmented Prompt Optimization agent (DeepSeek V3 via OPTIMIZER_API_KEY) that maintains iteration history and dynamically routes targeted rubric updates. Run the loop up to --max_iter (default 5) refinements per image, accepting the first sample whose consistency score reaches the threshold; reset the prompt-optimization history per image so one sample's refinement history does not affect the next.

## Related tools

- **DeepSeek V3** (Prompt optimizer (OPTIMIZER_API_KEY): analyzes critiques from failed samples and refines generation rubrics with memory of past attempts and dynamic routing) — github.com/rohnson1999/AutoVQA-G
- **Qwen2.5-VL-72B-Instruct** (CoT verifier (VERIFICATION_API_KEY): produces step-by-step visual critiques and weighted consistency scores that drive prompt refinement) — github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server whose caption, VQA, and mention generation prompts are the targets of template refinement) — github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Consistency score S_t >= tau (default tau = 0.9) for accepted annotations
- Refinement loop terminates within --max_iter (default 5) iterations per image
- Output JSON is parseable and each per-image record contains question, answer, object_description, and bbox (a four-coordinate list)
- Prompt optimization state is reset per image (no cross-sample leakage of refinement history)
- is_successful flag and iteration field indicate whether the accepted sample passed the threshold or was the highest-scoring iteration

## Limitations

- Computational overhead of the iterative generate-evaluate-refine loop is a stated limitation
- Requires three separate model endpoints (local MiniCPM-o 2.6 server, DashScope Qwen2.5-VL-72B-Instruct, DeepSeek API) to be configured and reachable
- Only the first threshold-passing sample or the highest-scoring iteration is kept per image; lower-scoring candidates are discarded

## Evidence
