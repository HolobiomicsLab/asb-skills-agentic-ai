---
name: visual-question-answering-annotation-generation
description: Use when you need to automatically create or reproduce VQA-G dataset
  annotations (question, answer, object mention, bounding box) with verified visual
  consistency rather than trusting single-pass multimodal LLM outputs.
when_to_use_negative:
- The task is evaluating annotation quality of an existing VQA-G dataset (e.g. reproducing
  Visual7W ablation metrics) rather than generating new annotations
- Only a single-pass annotation is acceptable and no iterative refinement API servers
  (MiniCPM-o, DashScope, DeepSeek, GroundingDINO) can be deployed
- The data has no visual grounding component and plain VQA pairs without bounding
  boxes suffice
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: AutoVQA-G
  role: Self-improving agentic framework implementing the generate-evaluate-refine
    annotation loop
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server for caption reasoning, VQA generation, and object
    mention generation
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier producing two-stage content-quality and bbox-accuracy critiques
    and weighted consistency scores
- name: DeepSeek V3
  role: Memory-augmented prompt optimizer that analyzes critiques and dynamically
    refines generation prompts
- name: GroundingDINO
  role: Visual grounding module returning bounding boxes for object mentions via HTTP
    POST
- name: Python
  role: Runtime for main.py and pipeline.py (pip install -r requirements.txt)
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:50e0eb2fbd3376438d8e0fbcba4688375a259ba918ee922377542787d5162a95
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
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# visual-question-answering-annotation-generation

## Summary

Automated generation of high-fidelity Visual Question Answering with Grounding (VQA-G) annotations using AutoVQA-G, an iterative generate-evaluate-refine agentic framework that combines CoT-based consistency verification with memory-augmented prompt optimization instead of error-prone single-pass VLM annotation.

## When to use

Use when you need to automatically create or reproduce VQA-G dataset annotations (question, answer, object mention, bounding box) with verified visual consistency rather than trusting single-pass multimodal LLM outputs.

## When NOT to use

- The task is evaluating annotation quality of an existing VQA-G dataset (e.g. reproducing Visual7W ablation metrics) rather than generating new annotations
- Only a single-pass annotation is acceptable and no iterative refinement API servers (MiniCPM-o, DashScope, DeepSeek, GroundingDINO) can be deployed
- The data has no visual grounding component and plain VQA pairs without bounding boxes suffice

## Inputs

- JSON annotations file (list of dictionaries, each with at least an 'image' key)
- Directory of input images (--image_dir)
- Environment variables: GENERATION_API_KEY, VERIFICATION_API_KEY, OPTIMIZER_API_KEY, GROUNDING_DINO_URL, GROUNDING_DINO_API_KEY

## Outputs

- JSON results file per image with question, answer, object_description, bbox, consistency_score, iteration, is_successful, CoT_critique, and timestamp
- Selected annotation per image: first sample passing threshold τ=0.9 or highest-scoring iteration
- Per-image consistency scores and iteration logs in --output_dir

## How to apply

Clone github.com/rohnson1999/AutoVQA-G and configure four endpoints in modules.py or via environment variables: MiniCPM-o 2.6 for local caption/VQA/mention generation (localhost:8000/v1), Qwen2.5-VL-72B-Instruct via DashScope as the CoT verifier, DeepSeek V3 as prompt optimizer, and a GroundingDINO HTTP endpoint (localhost:8001/predict) that returns bbox and confidence detections. Prepare an input JSON annotations file where each entry has an 'image' key, then run python main.py with --threshold 0.9 (consistency acceptance score τ) and --max_iter 5 (maximum refinement iterations per image). The pipeline runs Modular Generation (Caption Reasoning → VQA Generation → Visual Grounding via object mention plus GroundingDINO bbox), then two-stage CoT verification (content quality + bbox accuracy) producing weighted consistency scores, and a memory-augmented Prompt Optimization agent that refines generation rubrics from critiques; the loop accepts the first sample with consistency score S_t ≥ τ or the highest-scoring iteration otherwise. Note that prompt optimization resets per image, so one sample's refinement history does not affect the next.

## Related tools

- **AutoVQA-G** (Self-improving agentic framework implementing the generate-evaluate-refine annotation loop) — https://github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server for caption reasoning, VQA generation, and object mention generation)
- **Qwen2.5-VL-72B-Instruct** (CoT verifier producing two-stage content-quality and bbox-accuracy critiques and weighted consistency scores)
- **DeepSeek V3** (Memory-augmented prompt optimizer that analyzes critiques and dynamically refines generation prompts)
- **GroundingDINO** (Visual grounding module returning bounding boxes for object mentions via HTTP POST)
- **Python** (Runtime for main.py and pipeline.py (pip install -r requirements.txt)) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations path/to/annotations. --image_dir path/to/images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Each output record must contain all expected fields: image_identifier, question, answer, object_description, bbox, consistency_score, iteration, is_successful, CoT_critique, timestamp
- Accepted annotations must have consistency_score ≥ 0.9 (threshold τ), or the record must be flagged as the highest-scoring iteration with is_successful=false
- The loop must terminate within --max_iter 5 refinement iterations per image
- On the Visual7W benchmark the full pipeline should reproduce reference metrics of VQAScore 0.896 and mIoU 0.634
- GroundingDINO POST responses must return a detections list containing bbox and confidence for each query

## Limitations

- Current computational overhead of the iterative framework is a stated limitation
- Prompt optimization resets per image, so refinement history does not transfer across samples
- When no iteration passes the threshold, output quality depends on the best-effort highest-scoring iteration within the --max_iter budget

## Evidence
