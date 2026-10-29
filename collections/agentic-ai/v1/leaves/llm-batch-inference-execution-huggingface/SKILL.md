---
name: llm-batch-inference-execution-huggingface
description: Use when you need to reproduce or extend LLM benchmark runs (e.g., 13
  open-source models x 4 prompting strategies) over the CausalNet/SemEval2010Task8/COPA-family
  causal datasets using the provided Python inference scripts.
when_to_use_negative:
- You only need the paper's reported results (e.g., C_detect = 49.57%) rather than
  regenerated model outputs — read experiment1_results_table.csv / Table 5 instead
  of re-running inference.
- You need a scoring/aggregation skill rather than generation — the exact-match and
  similarity scoring belong to the analysis/ stage, not this inference stage.
- Your data is not in the paper's CSV schema (label column, text fields) — the scripts
  expect the repository's data/exp1.csv and data/Experiment2.csv formats.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Runtime for the inference scripts main.py (Experiment 1) and exp2.py (Experiment
    2), invoked with --data, --model_name/--model and output-folder flags
  repo: github.com/sydneyanuyah/CausalDiscovery
- name: DeepSeek-R1-Distill-Llama-70B
  role: Best-performing detection model loaded via 4-bit quantization for causal-detection
    inference
- name: Qwen2.5-Coder-32B-Instruct
  role: Best-performing extraction model run over the 8,925-row causal-extraction
    dataset
- name: DeepSeek-R1-0528-Qwen3-8B
  role: One of the 13 open-source models executed via the inference scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1109/bigdata66926.2025.11401449
    title: Benchmarking LLMs for Pairwise Causal Discovery in Biomedical and Multi-Domain
      Contexts
  source_content_hash: sha256:45ddf634922749ed04fd723debb687107f4a8c64e11f592be74fb20d65d50274
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSeek-R1-0528-Qwen3-8B
  - DeepSeek-R1-Distill-Llama-70B
  - DeepSeek-R1-Distill-Qwen-32B
  - Llama-3-8B
  - Llama-3.1-8B-Instruct
  - Llama-3.2-3B
  - Llama-3.2-3B-Instruct
  - Meta-Llama-3.1-8B
  - Meta-Llama-3.3-70B-Instruct
  - Mistral-7B-Instruct-v0.3
  - Mixtral-8x7B-Instruct-v0.1
  - Python
  - Qwen2.5-7B-Instruct
  - Qwen2.5-Coder-32B-Instruct
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7727272727272727
n_claims: 22
---

# llm-batch-inference-execution-huggingface

## Summary

Batch-execute open-source Hugging Face LLMs over the pairwise causal-discovery benchmark datasets (3,008-row causal-detection CSV; 8,925-row causal-extraction CSV) using the paper's inference scripts (main.py / exp2.py), with seed 4000 and 4-bit model loading, to generate raw model outputs per model and prompting strategy.

## When to use

Use when you need to reproduce or extend LLM benchmark runs (e.g., 13 open-source models x 4 prompting strategies) over the CausalNet/SemEval2010Task8/COPA-family causal datasets using the provided Python inference scripts.

## When NOT to use

- You only need the paper's reported results (e.g., C_detect = 49.57%) rather than regenerated model outputs — read experiment1_results_table.csv / Table 5 instead of re-running inference.
- You need a scoring/aggregation skill rather than generation — the exact-match and similarity scoring belong to the analysis/ stage, not this inference stage.
- Your data is not in the paper's CSV schema (label column, text fields) — the scripts expect the repository's data/exp1.csv and data/Experiment2.csv formats.

## Inputs

- data/exp1.csv (causal-detection dataset, 3,008 rows; dataset families include CausalNet, CausalProbe, SemEval2010Task8, MedCaus, CauseNet, FinCausal, CausalBench, CRASS, Coling22, ECI-B, PubMed, COPA)
- data/Experiment2.csv (causal-extraction dataset, 8,925 rows, task slices 2a-2h)
- Hugging Face model name or path (e.g., DeepSeek-R1-Distill-Llama-70B, Qwen2.5-Coder-32B-Instruct, Llama-3.2-3B-Instruct)
- Prompt-definition files (instruction-only, few-shot ICL, chain-of-thought, hybrid CoT+FICL)

## Outputs

- Per-model, per-prompting-strategy raw model outputs stored under experiments/experiment1/ and experiments/experiment2/
- Evaluated outputs and timing files grouped by model and prompting strategy in results/experiment2/
- Parsed predictions ready for exact-match scoring (D_{m,b} metrics) via the analysis/ scripts

## How to apply

Navigate into the relevant experiment folder (experiments/experiment1/ for causal detection, experiments/experiment2/ for causal extraction) and invoke the inference script with the CSV data path and a Hugging Face model name or path, e.g. 'python main.py --data data/exp1.csv --model_name MODEL_NAME_OR_PATH'. The scripts are seeded with seed 4000 for reproducibility, and the Experiment 1 script uses 4-bit model loading so 70B-scale models (e.g., DeepSeek-R1-Distill-Llama-70B) fit on limited GPU memory. Run the script once per (model, prompting strategy) pair — Experiment 1 strategies are instruction-only, few-shot in-context learning, chain-of-thought, and hybrid chain-of-thought with few-shot in-context learning. Experiment 2 exposes additional generation parameters: batch size 64, top-k 40, temperature 0.7, and maximum new-token count 256. Generated outputs land in the experiment output folders (results/experiment2/ groups evaluated outputs and timing files by model and prompting strategy) before being passed to exact-match and similarity scoring in analysis/.

## Related tools

- **Python** (Runtime for the inference scripts main.py (Experiment 1) and exp2.py (Experiment 2), invoked with --data, --model_name/--model and output-folder flags) — github.com/sydneyanuyah/CausalDiscovery
- **DeepSeek-R1-Distill-Llama-70B** (Best-performing detection model loaded via 4-bit quantization for causal-detection inference)
- **Qwen2.5-Coder-32B-Instruct** (Best-performing extraction model run over the 8,925-row causal-extraction dataset)
- **DeepSeek-R1-0528-Qwen3-8B** (One of the 13 open-source models executed via the inference scripts)

## Examples

```
cd experiments/experiment1 && python main.py --data data/exp1.csv --model_name MODEL_NAME_OR_PATH
```

## Evaluation signals

- The run completes with seed 4000 and, for Experiment 1, 4-bit model loading — verify these settings were used before comparing results.
- Output volume should match the input rows: 3,008 predictions for data/exp1.csv and 8,925 for data/Experiment2.csv per (model, prompting strategy) pair.
- Reproducing the reference run should yield DeepSeek-R1-Distill-Llama-70B with mean C_detect of approximately 49.57% and Qwen2.5-Coder-32B-Instruct with C_extract of 47.12% after scoring.
- Outputs should be organized/grouped by model and prompting strategy (as results/experiment2/ does) so downstream exact-match scoring can aggregate D_{m,b} per benchmark.
- Timing files in results/experiment2/ should exist for each evaluated run, confirming generation actually completed rather than silently failing.

## Limitations

- The Experiment 1 script uses 4-bit model loading, so outputs are produced under quantization and may differ from full-precision inference.
- Fixed generation defaults in Experiment 2 (batch size 64, top-k 40, temperature 0.7, max new tokens 256) constrain output length and may truncate reasoning or extraction answers.
- Reproducibility of the reference numbers (e.g., C_detect = 49.57%) depends on using seed 4000, the same prompting-definition files, and the same model checkpoints.
- The repository README notes no changelog/discussion content, so undocumented reproducibility gaps or limitations cannot be ruled out from the README alone.

## Evidence

- [readme] Experiment 1 inference script accepts a CSV path, Hugging Face model name or path, batch size, and label column: "The Experiment 1 inference script accepts a CSV path, Hugging Face model name or path, batch size, and label column:"
- [readme] seed 4000 and 4-bit model loading: "The scripts use seed `4000`. The Experiment 1 script uses 4-bit model loading."
- [readme] The Experiment 2 script exposes defaults of batch size 64, top-k 40, temperature 0.7, and maximum new-token count 256.: "The Experiment 2 script exposes defaults of batch size `64`, top-k `40`, temperature `0.7`, and maximum new-token count `256`."
- [readme] experiments/experiment1/ contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows.: "`experiments/experiment1/` contains the causal-detection data, prompt definitions, inference script, and model outputs. The dataset file contains 3,008 rows."
- [readme] results/experiment2/ contains evaluated outputs and timing files grouped by model and prompting strategy.: "- `results/experiment2/` contains evaluated outputs and timing files grouped by model and prompting strategy."
- [abstract] prompting methods from zero-shot instructions to Chain-of-Thought and Few-shot In-Context Learning: "We tested various prompting methods, from simple instructions (zero-shot) to more complex strategies like Chain-of-Thought (CoT) and Few-shot In-Context Learning (FICL)."
