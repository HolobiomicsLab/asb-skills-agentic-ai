---
name: annotation-operational-statistics-aggregation
description: Use when a batch annotation run over an image set has completed and you
  need per-dataset efficiency metrics (success rate, avg iterations, tokens per success)
  from the output records.
when_to_use_negative:
- You only have single-pass annotation outputs with no iteration or is_successful
  fields — the iteration/token-per-success metrics cannot be computed.
- The goal is quality/accuracy evaluation of the annotations (e.g., grounding accuracy
  vs. multimodal LLMs) rather than operational efficiency of the loop.
- The run used a different acceptance threshold (τ) or max_iter than 0.9/5, making
  direct comparison to the reported 91.8%/89.2% figures invalid.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: AutoVQA-G
  role: Annotation pipeline whose generate-evaluate-refine loop produces the per-image
    records that are aggregated into operational statistics
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server for caption, VQA, and mention generation during the
    loop whose token usage is counted
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier (via DashScope) producing the weighted consistency score that
    gates acceptance
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: DeepSeek V3
  role: Prompt optimizer (via DeepSeek API) whose refinement iterations determine
    the iteration count per image
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: GroundingDINO
  role: Visual grounding HTTP endpoint providing bounding boxes for the Visual Grounding
    stage
- name: Python
  role: Runtime for main.py batch annotation and for aggregating per-image records
    into statistics
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:4f4a4eeba7f40b23ee9dd430f72a99a88ff2e5bd5105a9ec4088f7c9dbeb7c9a
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# annotation-operational-statistics-aggregation

## Summary

Aggregates per-image records from an automated annotation loop (AutoVQA-G) into operational statistics — success rate, average iterations per success, and token cost per success — per dataset, to quantify annotation efficiency on Visual7W and Viz Wiz.

## When to use

Use when a batch annotation run over an image set has completed and you need per-dataset efficiency metrics (success rate, avg iterations, tokens per success) from the output records.

## When NOT to use

- You only have single-pass annotation outputs with no iteration or is_successful fields — the iteration/token-per-success metrics cannot be computed.
- The goal is quality/accuracy evaluation of the annotations (e.g., grounding accuracy vs. multimodal LLMs) rather than operational efficiency of the loop.
- The run used a different acceptance threshold (τ) or max_iter than 0.9/5, making direct comparison to the reported 91.8%/89.2% figures invalid.

## Inputs

- JSON annotation file (list of dicts, each with at least an 'image' key) per dataset (Visual7W, Viz Wiz)
- Image directory for each dataset
- Per-image output records from the AutoVQA-G loop (JSON in output_dir with question, answer, object_description, bbox, consistency_score, iteration, is_successful fields)
- Model endpoint configuration: MiniCPM-o 2.6 generation server, Qwen2.5-VL-72B-Instruct CoT verifier via DashScope, DeepSeek V3 prompt optimizer, GroundingDINO HTTP endpoint

## Outputs

- Per-dataset operational statistics: success rate, average iterations per success, tokens per success
- Per-image annotation records in output_dir JSON with consistency_score and iteration fields
- Optionally, content statistics (average question/answer/object-mention lengths and bounding-box area fractions)

## How to apply

Run the AutoVQA-G loop separately for each dataset (Visual7W and Viz Wiz) using 'python main.py --annotations <file> --image_dir <dir> --output_dir output/ --threshold 0.9 --max_iter 5', which accepts the first sample with weighted consistency score S_t ≥ 0.9 or, after 5 iterations, the highest-scoring iteration. Each image contributes one JSON record with question, answer, object_description, bbox, consistency_score, iteration, and is_successful fields. Aggregate the per-image records per dataset: compute success rate as the fraction of records with is_successful=true, average iterations per success over accepted records, and tokens consumed per success. Expect more effort on the harder Viz Wiz set (89.2% success, 2.15 avg iterations, 3.1K tokens) than Visual7W (91.8%, 1.62, 2.1K), and treat reproduction within reasonable variance of these figures as validation.

## Related tools

- **AutoVQA-G** (Annotation pipeline whose generate-evaluate-refine loop produces the per-image records that are aggregated into operational statistics) — https://github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server for caption, VQA, and mention generation during the loop whose token usage is counted) — https://github.com/rohnson1999/AutoVQA-G
- **Qwen2.5-VL-72B-Instruct** (CoT verifier (via DashScope) producing the weighted consistency score that gates acceptance) — https://github.com/rohnson1999/AutoVQA-G
- **DeepSeek V3** (Prompt optimizer (via DeepSeek API) whose refinement iterations determine the iteration count per image) — https://github.com/rohnson1999/AutoVQA-G
- **GroundingDINO** (Visual grounding HTTP endpoint providing bounding boxes for the Visual Grounding stage)
- **Python** (Runtime for main.py batch annotation and for aggregating per-image records into statistics) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations visual7w_annotations. --image_dir data/visual7w/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Every output record contains the required fields: question, answer, object_description, bbox, consistency_score, iteration (and is_successful).
- Visual7W success rate ≈ 91.8% and Viz Wiz ≈ 89.2% within reasonable variance.
- Average iterations per success ≈ 1.62 (Visual7W) and ≈ 2.15 (Viz Wiz).
- Tokens per success ≈ 2.1K (Visual7W) and ≈ 3.1K (Viz Wiz), with Viz Wiz consistently costlier.
- Each image contributes exactly one selected result: the first sample with consistency_score ≥ threshold (0.9) or the highest-scoring iteration when max_iter (5) is reached.

## Limitations

- Current computational overhead of the iterative loop is a stated limitation of the framework.
- Prompt optimization history is reset per image, so per-image iteration counts are independent but aggregate cost can be high for hard image sets.
- Comparison to reported figures is only valid at the same settings: threshold τ = 0.9 and max_iter = 5.

## Evidence

- [results] Running the AutoVQA-G loop yields a 91.8% success rate on Visual7W and 89.2% on Viz Wiz, with 1.62 vs 2.15 average iterations and 2.1K vs 3.1K tokens per success: [evidence span withheld — anchor status: not_found]
- [readme] Results are saved as a JSON file in output_dir. Each image contributes one selected result: the first sample that passes the threshold, or otherwise the highest-scoring iteration for that image.: "Each image contributes one selected result: the first sample that passes the threshold, or otherwise the highest-scoring iteration for that image."
- [readme] The loop terminates when the consistency score S_t ≥ τ or the maximum number of iterations is reached. Prompt optimization is reset per image: "The loop terminates when the consistency score `S_t ≥ τ` or the maximum number of iterations is reached. Prompt optimization is reset per image"
- [readme] Two-stage assessment — content quality verification + bbox accuracy verification — producing weighted consistency scores: "Two-stage assessment — content quality verification + bbox accuracy verification — producing weighted consistency scores"
- [discussion] While current computational overhead presents a limitation, AutoVQA-G offers a robust blueprint: [evidence span withheld — anchor status: not_found]
