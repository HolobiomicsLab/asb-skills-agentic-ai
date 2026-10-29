---
name: lora-fine-tuned-model-inference
description: Use when zero-shot LVLM extraction on real-world certificate/bidding
  documents underperforms due to seals, watermarks, or low contrast and a fine-tuned
  checkpoint (or LoRA SFT procedure) is available to push F1 toward the reported 93.65%.
when_to_use_negative:
- Zero-shot performance suffices — the zero-shot configuration already attains 86.43%
  F1 / 0.90 NED, and fine-tuning is described as an optional enhancement requiring
  8 A100 GPUs
- The task targets public benchmarks with different entity types/layouts (e.g., FUNSD,
  SROIE, COLD-CELL) where the paper's fine-tuned model was not trained and performs
  worse (e.g., 26.88% F1 on FUNSD)
- Only one or a few document types need extraction — the lightweight keyword-based
  classifier with dynamic prompts may be simpler than LoRA SFT
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: Qwen2.5-VL-7B
  role: Base LVLM fine-tuned via LoRA SFT and used for inference to generate structured
    extraction predictions
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: OpenCV
  role: Preprocessing of test document images before LVLM inference
- name: fairme/Qwen2.5-VL-7B-SFT
  role: Hosted fine-tuned checkpoint to load directly for inference
  repo: https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:e6b702d5497881868d3395bc5256af43e331da3d0971f33180435ad307c922b4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/FairmeHIT/Multi-VIE
  repo_url: https://github.com/FairmeHIT/Multi-VIE
  tools:
  - OpenCV
  - Qwen2.5-VL-7B
  - fairme/Qwen2.5-VL-7B-SFT
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lora-fine-tuned-model-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run structured visual information extraction using the LoRA supervised fine-tuned Qwen2.5-VL-7B checkpoint (fairme/Qwen2.5-VL-7B-SFT), which reaches 93.65% F1 and 0.9348 NED on the 16-type real-world bidding dataset, with improved robustness to seals, watermarks, and low contrast.

## When to use

Use when zero-shot LVLM extraction on real-world certificate/bidding documents underperforms due to seals, watermarks, or low contrast and a fine-tuned checkpoint (or LoRA SFT procedure) is available to push F1 toward the reported 93.65%.

## When NOT to use

- Zero-shot performance suffices — the zero-shot configuration already attains 86.43% F1 / 0.90 NED, and fine-tuning is described as an optional enhancement requiring 8 A100 GPUs
- The task targets public benchmarks with different entity types/layouts (e.g., FUNSD, SROIE, COLD-CELL) where the paper's fine-tuned model was not trained and performs worse (e.g., 26.88% F1 on FUNSD)
- Only one or a few document types need extraction — the lightweight keyword-based classifier with dynamic prompts may be simpler than LoRA SFT

## Inputs

- Document images from the real-world bidding dataset test split (29,580 images, 16 certificate types)
- Fine-tuned model checkpoint fairme/Qwen2.5-VL-7B-SFT (or base Qwen2.5-VL-7B plus LoRA SFT training data: 394,400 multi-granular annotations)
- Ground-truth annotations for metric computation

## Outputs

- Structured entity extraction predictions per image (~2.6 s/image on the 7B model)
- Entity-level F1-score and average normalized edit distance (target: 93.65% F1, 0.9348 NED), with per-type and aggregate metrics

## How to apply

Load the fine-tuned checkpoint fairme/Qwen2.5-VL-7B-SFT from HuggingFace, or reproduce it by running the repository's LoRA SFT code in app/sft (Qwen2.5-VL-7B trained on 394,400 multi-granular annotations with 8 NVIDIA A100 GPUs). Preprocess test images from the 29,580-image test split (16 certificate types, 7:3 split of the 98,600-image bidding dataset) using OpenCV, then run LVLM inference at roughly 2.6 seconds per image on the 7B model. Apply post-processing to standardize outputs: date formats, punctuation, capitalization, and removal of repetition artifacts. Evaluate with entity-level exact-match F1 (equations 19 and 22) and average normalized edit distance (equation 23); accept the run if metrics match Table 3 (93.65% F1, 0.9348 NED) within a small tolerance.

## Related tools

- **Qwen2.5-VL-7B** (Base LVLM fine-tuned via LoRA SFT and used for inference to generate structured extraction predictions) — https://github.com/FairmeHIT/Multi-VIE
- **OpenCV** (Preprocessing of test document images before LVLM inference)
- **fairme/Qwen2.5-VL-7B-SFT** (Hosted fine-tuned checkpoint to load directly for inference) — https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT

## Examples

```
conda activate multi_vie && cd app/sft && pip install -r requirements.txt  # then load https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT for inference via app/llm
```

## Evaluation signals

- Computed average F1 matches 93.65% within a small tolerance
- Average normalized edit distance matches 0.9348 within a small tolerance
- Inference throughput is approximately 2.6 seconds per image on the 7B model
- Post-processed outputs show standardized date formats, punctuation, capitalization, and no repetition artifacts
- Per-type metrics across all 16 certificate types aggregate consistently to the reported averages

## Limitations

- Fine-tuning is compute-intensive: 394,400 annotations and 8 NVIDIA A100 GPUs for LoRA SFT
- No statistical significance testing or variance reporting for the reported F1/NED improvements
- No public availability or access conditions stated for the real-world bidding dataset
- Model may still be challenged by degraded real-world conditions despite improved robustness (paper highlights seals, watermarks, and low contrast as the stress cases)

## Evidence
