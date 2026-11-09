---
name: llm-fine-tuning-dataset-schema-validation
description: Use when converting agent or gameplay observation-action logs into Alpaca-format
  JSONL for LLaMA fine-tuning, and a schema/vocabulary check is needed before training.
when_to_use_negative:
- When the goal is RL training in the KSPDG environments themselves — KSPDG explicitly
  provides evaluation environments, NOT training environments, since KSP was not designed
  for massively parallel faster-than-real-time RL execution
- When the input is already a validated Alpaca JSONL dataset with the four required
  fields and a checked action vocabulary — re-running conversion would duplicate work
- When preparing GPT (OpenAI API) fine-tuning data, which in this article was a separate
  minor fine-tuning with LRM hyperparameter adjustments rather than this Alpaca/navball-bot
  LLaMA pipeline
tools:
- name: LLaMA
  role: Target model family whose fine-tuning consumes the validated Alpaca-format
    JSONL datasets (10/25/50-file variants)
- name: Hugging Face
  role: Hosts the resulting LLaMA fine-tuning datasets (LLaMA 10 files sliding window,
    35 files quantized, 50 files) for reuse
- name: kspdg
  role: Source of the navball-bot gameplay log files in the agents-data directory
    used as raw input for conversion
  repo: https://github.com/ARCLab-MIT/kspdg
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:27a9cfeca4c03ed74fb85421cff729e8496b9bdac660f3b1175bb0a835065b35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ARCLab-MIT/kspdg
  tools:
  - Hugging Face
  - LLaMA
  - kspdg
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# LLM fine-tuning dataset schema validation

## Summary

Validates that LLaMA fine-tuning datasets built from navball-bot flight logs (KSPDG scenario E3) conform to the Alpaca schema — records must contain exactly the 'instruction', 'output', 'system', and 'history' fields, with output action labels drawn from the throttle vocabulary plus the added 'none' label.

## When to use

Use when converting agent or gameplay observation-action logs into Alpaca-format JSONL for LLaMA fine-tuning, and a schema/vocabulary check is needed before training.

## When NOT to use

- When the goal is RL training in the KSPDG environments themselves — KSPDG explicitly provides evaluation environments, NOT training environments, since KSP was not designed for massively parallel faster-than-real-time RL execution
- When the input is already a validated Alpaca JSONL dataset with the four required fields and a checked action vocabulary — re-running conversion would duplicate work
- When preparing GPT (OpenAI API) fine-tuning data, which in this article was a separate minor fine-tuning with LRM hyperparameter adjustments rather than this Alpaca/navball-bot LLaMA pipeline

## Inputs

- Navball-bot gameplay log files (LLaMA fine-tuning dataset files collected with the navball bot in KSPDG scenario E3) from the kspdg agents-data directory
- Per-step observation-action records with observation augmentation including prograde direction
- Throttle action vocabulary including the added 'none' no-action label

## Outputs

- JSONL dataset file of Alpaca-format records with instruction, output, system, and history fields
- Validation report confirming field presence and action-label vocabulary compliance for all records
- Fine-tuning datasets matching the Alpaca schema used for the LLaMA 10/25/50-file variants (hosted on Hugging Face)

## How to apply

Parse each per-step navball-bot log record into an observation-action pair (observations augmented with the prograde direction per the LLM Agent pseudocode), then map it to an Alpaca record: 'instruction' is the HUMAN prompt asking for the best throttle to capture the evader given the observations, 'output' is the assistant throttle decision, 'system' is the autonomous pursuit-agent prompt (throttles in vessel frame, x right, y toward target, z up, maximum throttle 1, 'Reason step-by-step.'), and 'history' holds prior dialogue turns. Assign the discrete 'none' action label to steps where no valid throttle action was produced (failed/invalid action invocations). Serialize records to a JSONL file, then run two validations: a direct field-presence check that every record contains 'instruction', 'output', 'system', and 'history', and an action-label vocabulary check that every 'output' label belongs to the throttle vocabulary {forward/left/right/up/down throttles ..., 'none'}. The validated dataset must match the Alpaca schema used for the LLaMA 10/25/50-file fine-tuning variants (fine-tune 10/25/50 files variants trained in the article).

## Related tools

- **LLaMA** (Target model family whose fine-tuning consumes the validated Alpaca-format JSONL datasets (10/25/50-file variants))
- **Hugging Face** (Hosts the resulting LLaMA fine-tuning datasets (LLaMA 10 files sliding window, 35 files quantized, 50 files) for reuse)
- **kspdg** (Source of the navball-bot gameplay log files in the agents-data directory used as raw input for conversion) — https://github.com/ARCLab-MIT/kspdg

## Examples

```
import ; recs=[.loads(l) for l in open('alpaca_dataset.l')]; assert all(set(r)=={'instruction','output','system','history'} for r in recs) and all(r['output'] in THROTTLE_VOCAB|{'none'} for r in recs)
```

## Evaluation signals

- Field-presence check: every JSONL record contains exactly the 'instruction', 'output', 'system', and 'history' fields
- Action-label vocabulary check: every 'output' label is one of the forward/left/right/up/down throttle labels or 'none'
- The 'none' label appears for steps where no valid throttle action was produced (failed/invalid action invocations)
- Resulting dataset matches the Alpaca schema used for the LLaMA 10/25/50-file fine-tuning variants
- All dataset files trace back to navball-bot-collected logs, not other agents or human gameplay

## Limitations

- The 'none' no-action label is a new addition to the discrete action vocabulary, so downstream training/evaluation must handle it consistently
- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are not reported, limiting exact reproducibility of the LLaMA variants trained on these datasets
- LLaMA training/inference outcomes are highly hardware-dependent and hardware specifications are not specified
- KSPDG environments are evaluation-only; data must be collected via the navball bot rather than by RL training in the simulator

## Evidence

- [methods] Dataset provenance: [evidence span withheld — anchor status: not_found]
- [abstract] Dataset hosting: "the trained models and datasets are available on Hugging Face"
- [results] Chain-of-thought prompting benefit: "The application of the Chain of Thought approach demonstrates a significant improvement in the generalization of spacecraft piloting techniques"
- [readme] KSPDG is evaluation-only, not for training data generation via RL: [evidence span withheld — anchor status: not_found]
