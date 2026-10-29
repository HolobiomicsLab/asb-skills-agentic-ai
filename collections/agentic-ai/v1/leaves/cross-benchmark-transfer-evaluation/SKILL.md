---
name: cross-benchmark-transfer-evaluation
description: Use when attack components were optimized only on a small training subset
  (e.g., 50 AdvBench samples via the TAP repository's advbench_subset.csv) and you
  must verify strong generalization by applying them to held-out benchmarks and models.
when_to_use_negative:
- You are still in the optimization phase — co-optimizing perturbations and steering
  prompts on the AdvBench-subset belongs to the attack phase, not transfer evaluation
- The target model is text-only (no image input path); JPS's full-image adversarial
  perturbations require multimodal LLMs
- No judge models (GPT-4o Mini, LlamaGuard3, HarmBench) are reachable, since ASR/MIFR
  scoring cannot be computed
tools:
- name: InternVL2-8B
  role: Target multimodal LLM evaluated during transfer inference
  repo: https://github.com/thu-coai/JPS
- name: Qwen2-VL-7B
  role: Target multimodal LLM evaluated during transfer inference
  repo: https://github.com/thu-coai/JPS
- name: MiniGPT4-13B
  role: Target compact multimodal model evaluated during transfer inference
  repo: https://github.com/thu-coai/JPS
- name: GPT-4o Mini Judge
  role: Judge for computing ASR on transferred responses
  repo: https://github.com/thu-coai/JPS
- name: LlamaGuard3 Judge
  role: Judge for computing ASR on transferred responses
  repo: https://github.com/thu-coai/JPS
- name: HarmBench Judge
  role: Judge for computing ASR on HarmBench transferred responses
  repo: https://github.com/centerforaisafety/HarmBench
- name: MMSafetyBench
  role: Transfer benchmark for multimodal safety evaluation
  repo: https://github.com/isXinLiu/MM-SafetyBench
- name: AdvBench-subset (TAP)
  role: Source of the 50-sample optimization subset used before transfer
  repo: https://github.com/RICommunity/TAP
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1145/3746027.3754561
    title: 'JPS: Jailbreak Multimodal Large Language Models with Collaborative Visual
      Perturbation and Textual Steering'
  source_content_hash: sha256:04eacab54d26e70e10e2ef0fdeb70869e6843f6ae015fceac97a0ea890dd5b89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thu-coai/JPS
  tools:
  - AdvBench-subset (TAP)
  - CUDA
  - GPT-4o Mini Judge
  - HarmBench Judge
  - InternVL2-8B
  - LlamaGuard3 Judge
  - MMSafetyBench
  - MiniGPT4-13B
  - Python
  - Qwen2-VL-7B
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cross-benchmark-transfer-evaluation

## Summary

Protocol for measuring whether jailbreak components (adversarial images and steering prompts) optimized on a small 50-sample AdvBench-subset generalize to unseen safety benchmarks (AdvBench, MMSafetyBench, HarmBench) and target MLLMs, scored with ASR and the MIFR quality metric.

## When to use

Use when attack components were optimized only on a small training subset (e.g., 50 AdvBench samples via the TAP repository's advbench_subset.csv) and you must verify strong generalization by applying them to held-out benchmarks and models.

## When NOT to use

- You are still in the optimization phase — co-optimizing perturbations and steering prompts on the AdvBench-subset belongs to the attack phase, not transfer evaluation
- The target model is text-only (no image input path); JPS's full-image adversarial perturbations require multimodal LLMs
- No judge models (GPT-4o Mini, LlamaGuard3, HarmBench) are reachable, since ASR/MIFR scoring cannot be computed

## Inputs

- Final optimized adversarial image per model (e.g., iteration5_.png from results/qwen2vl_7b/)
- Final optimized steering prompt per model (e.g., iteration5_.txt)
- Benchmark config directories: config/advbench, config/mmsafetybench, config/harmbench (YAML with model, batch_size, max_new_tokens in inference_config)
- Benchmark datasets: AdvBench, MMSafetyBench, HarmBench
- 50-sample AdvBench-subset (advbench_subset.csv from the RICommunity/TAP repository) used only for optimization

## Outputs

- Generated responses per model-benchmark pair under results/generate/responses
- Per-model, per-benchmark ASR values from the configured judges
- Per-model, per-benchmark MIFR scores from quality_evaluate.py
- Aggregated transfer table: transfer_results.csv or jps_asr_mifr_results.tsv

## How to apply

After the attack (optimization) phase converges on the 50-sample AdvBench-subset, run the inference (transfer) phase with run_config.py --type inference on the config/advbench, config/mmsafetybench, and config/harmbench directories, applying the final optimized adversarial image (iteration5_.png) and steering prompt (iteration5_.txt) to each benchmark for InternVL2-8B, Qwen2-VL-7B, and MiniGPT4-13B with batch_size and max_new_tokens set in inference_config. Compute ASR on generated responses using the configured judges (GPT-4o Mini Judge, LlamaGuard3 Judge, HarmBench Judge), then score response quality with the MIFR metric via quality_evaluate.py on results/generate/responses. Aggregate per-model, per-benchmark ASR and MIFR values into transfer_results.csv (or jps_asr_mifr_results.tsv) and check that transfer from the 50 training samples yields consistent results across different models and datasets.

## Related tools

- **InternVL2-8B** (Target multimodal LLM evaluated during transfer inference) — https://github.com/thu-coai/JPS
- **Qwen2-VL-7B** (Target multimodal LLM evaluated during transfer inference) — https://github.com/thu-coai/JPS
- **MiniGPT4-13B** (Target compact multimodal model evaluated during transfer inference) — https://github.com/thu-coai/JPS
- **GPT-4o Mini Judge** (Judge for computing ASR on transferred responses) — https://github.com/thu-coai/JPS
- **LlamaGuard3 Judge** (Judge for computing ASR on transferred responses) — https://github.com/thu-coai/JPS
- **HarmBench Judge** (Judge for computing ASR on HarmBench transferred responses) — https://github.com/centerforaisafety/HarmBench
- **MMSafetyBench** (Transfer benchmark for multimodal safety evaluation) — https://github.com/isXinLiu/MM-SafetyBench
- **AdvBench-subset (TAP)** (Source of the 50-sample optimization subset used before transfer) — https://github.com/RICommunity/TAP

## Examples

```
python run_config.py --config_dir './config/harmbench' --type inference --gpus 4,5,6,7 && python quality_evaluate.py --input_file results/generate/responses
```

## Evaluation signals

- transfer_results.csv (or jps_asr_mifr_results.tsv) exists and contains ASR and MIFR fields for every model-benchmark combination
- MIFR scores are higher than existing methods, per the reported pattern
- Results are consistent across different models and datasets despite optimizing on only 50 training samples
- The final iteration artifacts (iteration5_.png and iteration5_.txt) were used during inference rather than earlier iterations (iteration0–4)
- All three judge types (GPT-4o Mini, LlamaGuard3, HarmBench) ran over the generated responses without missing outputs

## Limitations

- Requires a CUDA-compatible GPU with at least 24GB memory and Python 3.8+
- ASR alone overlooks whether generated responses actually fulfill the attacker's malicious intent, which is why MIFR must be reported alongside it
- The article's discussion section has no content available (no changelog found), so deployment caveats beyond the methods text are undocumented

## Evidence

- [abstract] SOTA generalization claim: "JPS sets a new state-of-the-art in both ASR and MIFR across various MLLMs and benchmarks"
- [methods] Transfer phase definition: "Then apply the optimized components to other datasets:"
- [methods] Strong generalization from 50 samples: "- **Strong Generalization**: Effective transfer from 50 training samples"
- [methods] AdvBench-subset source: "50-sample optimization subset (available at [TAP Repository](https://github.com/RICommunity/TAP/blob/main/data/advbench_subset.csv))"
- [readme] Inference command: "python run_config.py --config_dir './config/advbench' --type inference --gpus 4,5,6,7"
- [readme] Final iteration artifacts: "*_iteration5_.png    # Final adversarial image"
