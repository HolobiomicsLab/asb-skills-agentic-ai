---
name: throttle-action-label-vocabulary-mapping
description: Use when converting agent flight logs or telemetry-action records into
  Alpaca-format JSONL for LLM fine-tuning, or when defining/validating a discrete
  action vocabulary that includes a no-op 'none' label.
when_to_use_negative:
- The goal is RL training directly in the environment — the KSPDG README states it
  provides evaluation environments, NOT training environments for reinforcement learning
  algorithms.
- Records already conform to the Alpaca schema with all four fields present and valid
  action labels; re-mapping would be redundant.
- Actions require continuous throttle vectors rather than the discrete vocabulary
  {forward/left/right/up/down throttles ..., 'none'}.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: kspdg
  role: Source of the navball-bot agents and the flight-log data converted into the
    Alpaca dataset
  repo: https://github.com/ARCLab-MIT/kspdg
- name: spacegym-kspdg
  role: Upstream KSPDG environment library (pursuit-evasion scenarios) from which
    the scenario-E3 logs originate
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: LLaMA
  role: Target models (10/25/50-file variants) fine-tuned on the converted Alpaca-format
    dataset
- name: Hugging Face
  role: Hosting of the fine-tuned models and converted datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:04088fea24509d8f10df743de0b08bf351cc5b71a71d8c9da886c7dbc3393fef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ARCLab-MIT/kspdg
  tools:
  - Hugging Face
  - LLaMA
  - kspdg
  - spacegym-kspdg
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# throttle-action-label-vocabulary-mapping

## Summary

Converts KSPDG navball-bot flight logs (scenario E3) into a synthetic Alpaca-format JSONL dataset with instruction/output/system/history fields for LLaMA fine-tuning, mapping per-step observation-action pairs to a discrete throttle vocabulary extended with a 'none' label for steps where no action is better than an action.

## When to use

Use when converting agent flight logs or telemetry-action records into Alpaca-format JSONL for LLM fine-tuning, or when defining/validating a discrete action vocabulary that includes a no-op 'none' label.

## When NOT to use

- The goal is RL training directly in the environment — the KSPDG README states it provides evaluation environments, NOT training environments for reinforcement learning algorithms.
- Records already conform to the Alpaca schema with all four fields present and valid action labels; re-mapping would be redundant.
- Actions require continuous throttle vectors rather than the discrete vocabulary {forward/left/right/up/down throttles ..., 'none'}.

## Inputs

- Navball-bot flight/gameplay log files (LLaMA fine-tuning dataset files collected in scenario E3, from the kspdg agents-data directory)
- LLM Agent pseudocode observation-augmentation spec (prograde direction)

## Outputs

- Alpaca-format JSONL dataset file with instruction, output, system, and history fields per record
- Discrete action-label vocabulary: {forward/left/right/up/down throttle labels ..., 'none'}

## How to apply

Load the navball-bot gameplay log files (the LLaMA fine-tuning dataset files collected with the navball bot in scenario E3) from the kspdg agents-data directory, and parse each per-step record into an observation-action pair using the observation augmentation with the prograde direction defined in the LLM Agent pseudocode. Map each pair to an Alpaca record: 'instruction' = the HUMAN user prompt asking for the best throttle to capture the evader given the observations, 'output' = the assistant throttle decision, 'system' = the autonomous pursuit-agent prompt (throttles in the vessel frame, x right, y toward target, z up, maximum throttle 1, 'Reason step-by-step.'), and 'history' = prior dialogue turns. Assign the added 'none' action label to steps where no valid throttle action was produced (the 'Return None or a default action' failure path), since sometimes no action is better than an action. Serialize all records to a JSONL dataset file, then validate that every record contains the four required fields and that output action labels come from the throttle vocabulary including 'none' — the schema must match the one used for the LLaMA 10/25/50-file fine-tuning variants.

## Related tools

- **kspdg** (Source of the navball-bot agents and the flight-log data converted into the Alpaca dataset) — https://github.com/ARCLab-MIT/kspdg
- **spacegym-kspdg** (Upstream KSPDG environment library (pursuit-evasion scenarios) from which the scenario-E3 logs originate) — https://github.com/mit-ll/spacegym-kspdg
- **LLaMA** (Target models (10/25/50-file variants) fine-tuned on the converted Alpaca-format dataset)
- **Hugging Face** (Hosting of the fine-tuned models and converted datasets)

## Examples

```
python -c "import ; recs=[.loads(l) for l in open('alpaca_train.l')]; assert all(set(r)=={'instruction','output','system','history'} for r in recs)"
```

## Evaluation signals

- Field-presence check: every JSONL record contains exactly 'instruction', 'output', 'system', and 'history'.
- Action-label vocabulary check: every 'output' label is drawn from the throttle vocabulary including 'none'.
- The resulting dataset matches the Alpaca schema used for the LLaMA 10/25/50-file fine-tuning variants.
- 'none' is assigned only to steps corresponding to failed/invalid action invocations (the 'Return None or a default action' path).
- Each line parses as valid JSON (JSONL format) and 'system' prompts specify vessel-frame throttle semantics (x right, y toward target, z up, max throttle 1) with 'Reason step-by-step.'

## Limitations

- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are not reported, limiting reproducibility of the downstream fine-tuning.
- All dataset files were collected using only the navball bot, so the dataset reflects a single expert-mimicking behavior source.
- Outcomes are highly hardware-dependent; hardware specifications for local LLaMA training/inference are not specified.
- KSPDG is an evaluation environment, not a training environment, so the converted dataset is the training substrate — not direct RL in the simulator.

## Evidence

- [methods] Navball bot dataset provenance: [evidence span withheld — anchor status: not_found]
- [abstract] Training strategy context: "Our approach leverages prompt engineering, few-shot prompting, and fine-tuning techniques"
- [results] Chain-of-thought prompting in prompts: "The application of the Chain of Thought approach demonstrates a significant improvement in the generalization of spacecraft piloting techniques"
- [readme] KSPDG is evaluation-only: [evidence span withheld — anchor status: not_found]
