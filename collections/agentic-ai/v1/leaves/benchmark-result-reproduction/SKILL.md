---
name: benchmark-result-reproduction
description: Use when an agent must verify published LVLM extraction metrics (entity-level
  F1, normalized edit distance) by re-running inference on a held-out document test
  split and comparing computed scores to reported values within a small tolerance.
when_to_use_negative:
- Goal is the zero-shot configuration — use the base Qwen2.5-VL-7B with ICL-based
  dynamic prompts instead of the LoRA fine-tuned checkpoint
- Input is one of the public benchmarks (SROIE, SCID, COLD-CELL, COLD-SIBR, EPHOIE,
  FUNSD, ICDAR, POIE) rather than the real-world bidding dataset, since its public
  availability is not stated
- Task is reproducing the supervised OCR & UIE baseline (68.08% F1) rather than the
  fine-tuned LVLM results
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: Qwen2.5-VL-7B
  role: Base LVLM that is LoRA fine-tuned (fairme/Qwen2.5-VL-7B-SFT) and run on the
    test images to produce structured extraction predictions
  repo: https://github.com/FairmeHIT/Multi-VIE
- name: OpenCV
  role: Preprocessing of the test document images prior to LVLM inference
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-026-49319-z
    title: Visual information extraction from documents via classification-guided
      large vision-language models
  source_content_hash: sha256:f0b0234c33f56b6574738a3f7e5f1a8a371370ea3a0e5873072880f4efb67345
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# benchmark-result-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the paper's fine-tuned (LoRA SFT) Qwen2.5-VL-7B extraction results on the real-world bidding dataset by evaluating the released checkpoint (fairme/Qwen2.5-VL-7B-SFT) against the reported 93.65% F1 and 0.9348 NED from Table 3.

## When to use

Use when an agent must verify published LVLM extraction metrics (entity-level F1, normalized edit distance) by re-running inference on a held-out document test split and comparing computed scores to reported values within a small tolerance.

## When NOT to use

- Goal is the zero-shot configuration — use the base Qwen2.5-VL-7B with ICL-based dynamic prompts instead of the LoRA fine-tuned checkpoint
- Input is one of the public benchmarks (SROIE, SCID, COLD-CELL, COLD-SIBR, EPHOIE, FUNSD, ICDAR, POIE) rather than the real-world bidding dataset, since its public availability is not stated
- Task is reproducing the supervised OCR & UIE baseline (68.08% F1) rather than the fine-tuned LVLM results

## Inputs

- Fine-tuned model checkpoint fairme/Qwen2.5-VL-7B-SFT (HuggingFace) or LoRA SFT recipe from the FairmeHIT/Multi-VIE repository
- Test split: 29,580 document images across 16 certificate types (7:3 train/test split of the 98,600-image real-world bidding dataset)
- Ground-truth annotations for entity-level scoring

## Outputs

- Computed average F1-score and normalized edit distance for the fine-tuned model (target: 93.65% F1, 0.9348 NED)
- Per-type metrics across the 16 certificate types (Table 3 reproduction)
- Structured extraction predictions per test image (~2.6 s/image inference)

## How to apply

Load the fine-tuned checkpoint fairme/Qwen2.5-VL-7B-SFT from HuggingFace (or reproduce it via the LoRA supervised fine-tuning in the repository's app/sft directory, trained on 394,400 multi-granular annotations with 8 NVIDIA A100 GPUs). Load the test split of 29,580 images across 16 certificate types from the 7:3 train/test split of the 98,600-image bidding dataset, preprocessing images with OpenCV. Run LVLM inference per image at the reported throughput of ~2.6 seconds/image, then apply post-processing to standardize date formats, punctuation, capitalization, and remove repetition artifacts. Compute entity-level F1-score via exact-match true positives (equations 19 and 22) and average normalized edit distance (equation 23) against ground-truth annotations, aggregating per-type and average metrics. Accept the reproduction if computed average F1 and NED match the reported Table 3 values (93.65% F1, 0.9348 NED) within a small tolerance.

## Related tools

- **Qwen2.5-VL-7B** (Base LVLM that is LoRA fine-tuned (fairme/Qwen2.5-VL-7B-SFT) and run on the test images to produce structured extraction predictions) — https://github.com/FairmeHIT/Multi-VIE
- **OpenCV** (Preprocessing of the test document images prior to LVLM inference)

## Examples

```
git clone https://github.com/FairmeHIT/Multi-VIE; cd app/sft && pip install -r requirements.txt  # then evaluate the fairme/Qwen2.5-VL-7B-SFT checkpoint on the 29,580-image test split
```

## Evaluation signals

- Computed average F1 equals 93.65% and NED equals 0.9348 within a small tolerance (Table 3 fine-tuned values)
- Average inference throughput matches the reported ~2.6 seconds per image on the 7B model
- Post-processed outputs show standardized date formats, punctuation, and capitalization with repetition artifacts removed before scoring
- Per-type metrics aggregated across all 16 certificate types; test set size equals 29,580 images (7:3 split of 98,600)
- Model path resolves to the released checkpoint fairme/Qwen2.5-VL-7B-SFT on HuggingFace

## Limitations

- The real-world bidding dataset has no stated public availability, so full reproduction may require the released checkpoint rather than retraining on original data
- Fine-tuning reproduction requires substantial compute: 394,400 annotations trained on 8 NVIDIA A100 GPUs
- No statistical significance testing or variance reporting accompanies the reported F1 and NED improvements, so tolerance for reproduction must be chosen ad hoc
- Reported robustness (against seals, watermarks, low contrast) is qualitative and hard to verify numerically from the metrics alone

## Evidence

- [readme] fine-tuned models at https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT: "fine-tuned models at https://huggingface.co/fairme/Qwen2.5-VL-7B-SFT"
- [abstract] Optional domain-specific fine-tuning further improves performance to 93.65% F1 and 0.93 NED: "Optional domain-specific fine-tuning further improves performance to 93.65% F1 and 0.93 NED, demonstrating superior robustness against seals, watermarks, and low contrast"
- [methods] we provide an optional enhancement through supervised fine-tuning of the base LVLM using low-rank adaptation (LoRA): "we provide an optional enhancement through supervised fine-tuning of the base LVLM using low-rank adaptation (LoRA)"
- [methods] an LVLM inference and post-processing module to generate structured predictions: "an LVLM inference and post-processing module to generate structured predictions"
- [readme] Code is available at https://github.com/FairmeHIT/Multi-VIE: "Code is available at https://github.com/FairmeHIT/Multi-VIE"
