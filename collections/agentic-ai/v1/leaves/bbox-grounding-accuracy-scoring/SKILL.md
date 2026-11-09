---
name: bbox-grounding-accuracy-scoring
description: 'Use when evaluating or reproducing grounding accuracy of VQA-G annotation
  pipelines: you have bbox predictions and reference boxes on Visual7W or Viz Wiz
  and need mIoU, Acc@0.5IoU, and VQA-G scores.'
when_to_use_negative:
- The task is text-only VQA evaluation without any grounding boxes — there are no
  bboxes to score mIoU or Acc@0.5IoU against
- The evaluation datasets are not Visual7W or Viz Wiz; the reference Table 2 tolerances
  and reported values (0.634/0.747, 0.649/0.737) are dataset-specific and would not
  transfer
- You only need the annotation pipeline itself (generation of VQA-G data) rather than
  scoring/comparison of grounding accuracy against baselines
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3474
tools:
- name: AutoVQA-G
  role: Framework whose generate-evaluate-refine loop produces the grounded annotations
    scored by mIoU and VQA-G metrics
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: GroundingDINO
  role: Visual grounding module producing the object-mention bounding boxes whose
    accuracy is scored
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: MiniCPM-o 2.6
  role: Local generation server for caption, VQA, and object mention generation feeding
    the grounding step
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: Qwen2.5-VL-72B-Instruct
  role: CoT verifier producing consistency scores used for per-image sample selection
    and bbox accuracy verification
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: DeepSeek V3
  role: Prompt optimizer refining generation prompts across refinement iterations
    to improve grounding fidelity
  repo: https://github.com/rohnson1999/AutoVQA-G
- name: GPT-4o
  role: Baseline multimodal LLM whose zero-shot, CoT, and tool-assisted grounding
    outputs are compared in Table 2
- name: Gemini
  role: Baseline multimodal LLM whose zero-shot, CoT, and tool-assisted grounding
    outputs are compared in Table 2
- name: Python
  role: Runtime used to execute main.py and compute mIoU / Acc@0.5IoU / VQA-G scores
  repo: https://github.com/rohnson1999/AutoVQA-G
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/icassp55912.2026.11462754
    title: 'AutoVQA-G: Self-Improving Agentic Framework for Automated Visual Question
      Answering and Grounding Annotation'
  source_content_hash: sha256:1dc7b8fb1749b75b24d4e242487c55fdedb5eba308b6143c228291625e87d1ad
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# bbox-grounding-accuracy-scoring

## Summary

Scores visual grounding quality of VQA-G annotations by comparing GroundingDINO bounding boxes against baselines, computing mIoU and Acc@0.5IoU alongside an overall VQA-G score. Used to reproduce Table 2 comparisons of AutoVQA-G versus GPT-4o and Gemini.

## When to use

Use when evaluating or reproducing grounding accuracy of VQA-G annotation pipelines: you have bbox predictions and reference boxes on Visual7W or Viz Wiz and need mIoU, Acc@0.5IoU, and VQA-G scores.

## When NOT to use

- The task is text-only VQA evaluation without any grounding boxes — there are no bboxes to score mIoU or Acc@0.5IoU against
- The evaluation datasets are not Visual7W or Viz Wiz; the reference Table 2 tolerances and reported values (0.634/0.747, 0.649/0.737) are dataset-specific and would not transfer
- You only need the annotation pipeline itself (generation of VQA-G data) rather than scoring/comparison of grounding accuracy against baselines

## Inputs

- Visual7W tellingQA annotation JSON (list of dicts with 'image' key) plus image directory
- Viz Wiz evaluation annotation JSON plus image directory
- Running GroundingDINO HTTP endpoint accepting POST with 'image', 'query', 'box_threshold', 'text_threshold' and returning a 'detections' list with 'bbox' and 'confidence'
- GPT-4o and Gemini baseline outputs (zero-shot, CoT, tool-assisted variants) on the same two datasets

## Outputs

- table2_metrics.csv with reproduced mIoU, Acc@0.5IoU, and VQA-G scores for AutoVQA-G and each baseline on both datasets
- Per-image output JSON containing question, answer, object_description, bbox, consistency_score, iteration, is_successful, and CoT_critique

## How to apply

Run main.py from the AutoVQA-G repository on the Visual7W tellingQA annotations and the Viz Wiz evaluation annotations, with the GroundingDINO bbox endpoint active, consistency threshold τ=0.9, and max_iter=5 for the generate-evaluate-refine loop. Per image, select the first sample whose consistency score passes the threshold, or otherwise the highest-scoring iteration. Run GPT-4o and Gemini baselines (zero-shot, CoT, and tool-assisted variants) on the same two evaluation conditions. Compute visual grounding mIoU, Acc@0.5IoU, and the overall VQA-G score for each system, then assemble table2_metrics.csv and compare against reference values (mIoU 0.634 / VQA-G 0.747 on Visual7W; 0.649 / 0.737 on Viz Wiz), flagging any mismatch beyond tolerance. Rationale: CoT-based Consistency Evaluation produces verifiable quality scores, and the two-stage assessment (content quality + bbox accuracy verification) yields weighted consistency scores that gate acceptance of each annotation.

## Related tools

- **AutoVQA-G** (Framework whose generate-evaluate-refine loop produces the grounded annotations scored by mIoU and VQA-G metrics) — https://github.com/rohnson1999/AutoVQA-G
- **GroundingDINO** (Visual grounding module producing the object-mention bounding boxes whose accuracy is scored) — https://github.com/rohnson1999/AutoVQA-G
- **MiniCPM-o 2.6** (Local generation server for caption, VQA, and object mention generation feeding the grounding step) — https://github.com/rohnson1999/AutoVQA-G
- **Qwen2.5-VL-72B-Instruct** (CoT verifier producing consistency scores used for per-image sample selection and bbox accuracy verification) — https://github.com/rohnson1999/AutoVQA-G
- **DeepSeek V3** (Prompt optimizer refining generation prompts across refinement iterations to improve grounding fidelity) — https://github.com/rohnson1999/AutoVQA-G
- **GPT-4o** (Baseline multimodal LLM whose zero-shot, CoT, and tool-assisted grounding outputs are compared in Table 2)
- **Gemini** (Baseline multimodal LLM whose zero-shot, CoT, and tool-assisted grounding outputs are compared in Table 2)
- **Python** (Runtime used to execute main.py and compute mIoU / Acc@0.5IoU / VQA-G scores) — https://github.com/rohnson1999/AutoVQA-G

## Examples

```
python main.py --annotations visual7w_tellingqa.json --image_dir images/ --output_dir output/ --threshold 0.9 --max_iter 5
```

## Evaluation signals

- Reproduced Table 2 ordering holds: AutoVQA-G achieves superior visual grounding accuracy compared to leading multimodal LLMs on both datasets
- Reproduced mIoU and VQA-G values match reported references: 0.634 / 0.747 on Visual7W and 0.649 / 0.737 on Viz Wiz
- Per-image output JSON has one selected result per image with is_successful=true only when consistency_score ≥ τ (0.9) or the highest-scoring iteration was selected
- Acc@0.5IoU values are ≤ mIoU values for the same system and dataset (a box passing IoU ≥ 0.5 also contributes positively to mean IoU)

## Limitations

- Current computational overhead is a stated limitation of the iterative refinement approach
- GroundingDINO bbox accuracy depends on the model's ability to localize the object mention; complex or out-of-distribution scenarios can stress the verification
- Prompt optimization state is reset per image, so cross-image transfer of learned refinement history is not used
- Maximum of max_iter=5 refinement iterations per image; samples that never reach the 0.9 consistency threshold fall back to the highest-scoring iteration

## Evidence

- [abstract] AutoVQA-G generates VQA-G datasets with superior visual grounding accuracy compared to leading multimodal LLMs: "AutoVQA-G generates VQA-G datasets with superior visual grounding accuracy compared to leading multimodal LLMs"
- [methods] Two-stage assessment — content quality verification + bbox accuracy verification: "Two-stage assessment — content quality verification + bbox accuracy verification"
- [readme] Output format description: "Each image contributes one selected result: the first sample that passes the threshold, or otherwise the highest-scoring iteration for that image."
- [readme] Consistency threshold parameter: "`--threshold` | `0.9` | Consistency score threshold (τ) for acceptance"
