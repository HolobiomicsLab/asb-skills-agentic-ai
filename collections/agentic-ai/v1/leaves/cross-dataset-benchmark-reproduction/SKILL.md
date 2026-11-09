---
name: cross-dataset-benchmark-reproduction
description: Use when a target paper reports a metrics table across multiple datasets
  or evaluation conditions and the goal is to independently re-run the proposed method
  plus baselines to verify the reported ordering and values.
when_to_use_negative:
- The paper reports only qualitative results or single-dataset single-metric numbers
  with no comparison table to reproduce
- The baselines require proprietary models or API keys you cannot provision, making
  a like-for-like cross-dataset comparison impossible
- The evaluation annotations or image directories for one of the datasets are unavailable,
  since the skill's validation depends on reproducing values on both conditions
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: AutoVQA-G
  role: Method under reproduction; its generate-evaluate-refine pipeline produces
    the VQA-G annotations scored in Table 2
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server for caption, VQA, and mention generation during annotation
    runs
- name: Qwen2.5-VL-72B-Instruct
  role: CoT consistency verifier via DashScope, used in the evaluation stage of each
    refinement loop
- name: DeepSeek V3
  role: Prompt optimizer via DeepSeek API, refining generation prompts across iterations
- name: GroundingDINO
  role: Bounding-box generation for the Visual Grounding stage whose mIoU/Acc@0.5IoU
    are evaluated
- name: Python
  role: Execution environment; main.py is run per dataset and metrics are assembled
    into table2_metrics.csv
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:a5288ef61329732cbd9e28d933efbeb6beb03c19200e3819cfa2cd1c6f88e459
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rohnson1999/AutoVQA-G
  tools:
  - AutoVQA-G
  - DeepSeek
  - DeepSeek V3
  - GPT-4o
  - Gemini
  - GroundingDINO
  - MiniCPM-o
  - MiniCPM-o 2.6
  - Python
  - Qwen2.5-VL-72B-Instruct
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-dataset-benchmark-reproduction

## Summary

Reproduce a paper's comparison table (here Table 2 of AutoVQA-G: VQA-G score, mIoU, Acc@0.5IoU) by re-running the method and its baselines across multiple evaluation conditions (Visual7W and Viz Wiz) and validating reproduced metrics against reference values.

## When to use

Use when a target paper reports a metrics table across multiple datasets or evaluation conditions and the goal is to independently re-run the proposed method plus baselines to verify the reported ordering and values.

## When NOT to use

- The paper reports only qualitative results or single-dataset single-metric numbers with no comparison table to reproduce
- The baselines require proprietary models or API keys you cannot provision, making a like-for-like cross-dataset comparison impossible
- The evaluation annotations or image directories for one of the datasets are unavailable, since the skill's validation depends on reproducing values on both conditions

## Inputs

- Visual7W tellingQA annotation JSON files (list of dicts with an 'image' key)
- Viz Wiz evaluation annotation JSON files
- Image directories for both datasets
- Baseline annotation outputs from GPT-4o and Gemini (zero-shot, CoT, tool-assisted variants)

## Outputs

- table2_metrics.csv with VQA-G score, mIoU, and Acc@0.5IoU for AutoVQA-G and each baseline on both datasets
- Per-image selected annotation JSON (consistency_score, bbox, iteration, is_successful, CoT_critique) per dataset
- Mismatch flags comparing reproduced vs reference values (0.634/0.747 Visual7W; 0.649/0.737 Viz Wiz)

## How to apply

Install the method's repository (AutoVQA-G) and configure all endpoints it depends on: the local MiniCPM-o 2.6 generation server for caption/VQA/mention generation, Qwen2.5-VL-72B-Instruct via DashScope as the CoT verifier, DeepSeek V3 as the prompt optimizer, and a GroundingDINO HTTP endpoint for bounding boxes. Run main.py on each evaluation condition (Visual7W tellingQA annotations, then Viz Wiz annotations) with the paper's default settings — consistency threshold 0.9 and max_iter 5 — so each image contributes either the first sample passing the threshold or the highest-scoring iteration. Run the competing baselines (zero-shot, CoT, and tool-assisted GPT-4o and Gemini variants) on the same two annotation sets, then compute visual grounding mIoU, Acc@0.5IoU, and overall VQA-G scores for every system. Assemble the metrics into a table2_metrics.csv and compare each reproduced value against the reference values (AutoVQA-G mIoU 0.634 / VQA-G 0.747 on Visual7W; 0.649 / 0.737 on Viz Wiz), flagging mismatches beyond tolerance. The reproduction is accepted only when it reproduces the paper's ordering: AutoVQA-G achieving superior visual grounding accuracy over all multimodal LLM baselines.

## Related tools

- **AutoVQA-G** (Method under reproduction; its generate-evaluate-refine pipeline produces the VQA-G annotations scored in Table 2) — https://github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server for caption, VQA, and mention generation during annotation runs)
- **Qwen2.5-VL-72B-Instruct** (CoT consistency verifier via DashScope, used in the evaluation stage of each refinement loop)
- **DeepSeek V3** (Prompt optimizer via DeepSeek API, refining generation prompts across iterations)
- **GroundingDINO** (Bounding-box generation for the Visual Grounding stage whose mIoU/Acc@0.5IoU are evaluated)
- **Python** (Execution environment; main.py is run per dataset and metrics are assembled into table2_metrics.csv) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations visual7w_tellingqa. --image_dir images/ --output_dir output/visual7w --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Reproduced AutoVQA-G mIoU and VQA-G on Visual7W match reference values 0.634 and 0.747 within tolerance
- Reproduced AutoVQA-G mIoU and VQA-G on Viz Wiz match reference values 0.649 and 0.737 within tolerance
- Table 2 ordering holds: AutoVQA-G's mIoU, Acc@0.5IoU (0.720 on Visual7W), and VQA-G score exceed all GPT-4o and Gemini baselines on both datasets
- Per-image output JSON contains exactly one selected result per image, either the first sample with consistency_score >= 0.9 (threshold tau) or the highest-scoring of at most 5 iterations
- Prompt optimization resets per image, so refinement history from one sample does not leak into the next

## Limitations

- Current computational overhead of the iterative generate-evaluate-refine loop is a stated limitation
- No changelog is available for the repository, so version drift between the paper and released code cannot be tracked
- Reproduction depends on external API endpoints (DashScope, DeepSeek, GroundingDINO HTTP) whose models may change or drift from those used in the paper

## Evidence

- [results] On Visual7W, AutoVQA-G achieves the best grounding results (mIoU 0.634, Acc@0.5IoU 0.720) and the highest overall VQA-G score (0.747), exceeding all GPT-4o and Gemini baselines reported in Table 2.: [evidence span withheld — anchor status: not_found]
