---
name: navball-bot-trajectory-data-parsing
description: Use when raw navball-bot gameplay/flight log files from the kspdg agents-data
  directory must be transformed into instruction-tuning files (Alpaca JSONL) for LLaMA
  fine-tuning of a pursuit-agent controller.
when_to_use_negative:
- The goal is RL training inside KSPDG itself — the README states KSPDG 'provides
  evaluations environments, _NOT_ training environments for reinforcement learning
  algorithms', so these logs are for offline LLM fine-tuning, not environment-side
  training
- The logs come from human gameplay or the mission_generator synthetic-expert scripts
  rather than the navball bot; the paper notes 'all dataset files were collected using
  the navball bot'
- The target model is GPT via the OpenAI API with LRM hyperparameter sweeps rather
  than the locally fine-tuned LLaMA variants
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3418
tools:
- name: KSPDG
  role: simulation environment whose scenario E3 navball-bot episodes generate the
    parsed flight logs
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: LLaMA
  role: fine-tuned target model that consumes the resulting Alpaca-format JSONL dataset
    (10/25/50-file variants)
- name: Hugging Face
  role: hosts the converted datasets and fine-tuned LLaMA model repositories
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:9a69c691a24e9f6107a11be260c4c1559026edbb2a084e4e1d46f875b17e0135
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - Hugging Face
  - KSPDG
  - LLaMA
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3418
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# navball-bot-trajectory-data-parsing

## Summary

Parses flight logs collected with the navball bot (a prograde-alignment pursuit agent in the KSPDG Kerbal Space Program environment, scenario E3) and converts each observation-action pair into Alpaca-format JSONL records (instruction, output, system, history) used to fine-tune the paper's LLaMA 10/25/50-file models, including a 'none' action label for steps where no throttle action was valid.

## When to use

Use when raw navball-bot gameplay/flight log files from the kspdg agents-data directory must be transformed into instruction-tuning files (Alpaca JSONL) for LLaMA fine-tuning of a pursuit-agent controller.

## When NOT to use

- The goal is RL training inside KSPDG itself — the README states KSPDG 'provides evaluations environments, _NOT_ training environments for reinforcement learning algorithms', so these logs are for offline LLM fine-tuning, not environment-side training
- The logs come from human gameplay or the mission_generator synthetic-expert scripts rather than the navball bot; the paper notes 'all dataset files were collected using the navball bot'
- The target model is GPT via the OpenAI API with LRM hyperparameter sweeps rather than the locally fine-tuned LLaMA variants

## Inputs

- navball-bot flight log files from the kspdg agents-data directory (LLaMA fine-tuning dataset files, scenario E3)
- LLM Agent pseudocode defining observation augmentation with the prograde direction
- throttle action vocabulary (forward/left/right/up/down throttles plus the 'none' label)

## Outputs

- Alpaca-format JSONL dataset file where every record carries 'instruction', 'output', 'system', and 'history' fields
- validated dataset matching the schema used for the LLaMA 10/25/50-file fine-tuning variants

## How to apply

Load the per-step navball-bot log files (collected in scenario E3) and parse each record into an observation-action pair, augmenting the observation with the prograde direction per the LLM Agent pseudocode. Map each pair to an Alpaca record: 'instruction' is the HUMAN prompt asking for the best throttle to capture the evader given the observations; 'output' is the assistant throttle decision; 'system' is the autonomous pursuit-agent prompt (throttles in the vessel frame, x right, y toward target, z up, max throttle 1, 'Reason step-by-step.'); 'history' holds prior dialogue turns for few-shot context. Assign the added 'none' action label to steps where the agent produced a failed or invalid action invocation (the 'Return None or a default action' failure path). Serialize all records to a single JSONL file and validate field presence and the action-label vocabulary ({forward/left/right/up/down throttles ..., 'none'}).

## Related tools

- **KSPDG** (simulation environment whose scenario E3 navball-bot episodes generate the parsed flight logs) — https://github.com/mit-ll/spacegym-kspdg
- **LLaMA** (fine-tuned target model that consumes the resulting Alpaca-format JSONL dataset (10/25/50-file variants))
- **Hugging Face** (hosts the converted datasets and fine-tuned LLaMA model repositories)

## Examples

```
git clone git@github.com:mit-ll/spacegym-kspdg.git
```

## Evaluation signals

- Every serialized record contains all four required fields: 'instruction', 'output', 'system', 'history'
- Every 'output' action label belongs to the throttle vocabulary including the added 'none' label (steps with failed/invalid action invocations)
- The dataset schema matches the Alpaca format used for the LLaMA 10-file, 25-file, and 50-file fine-tuning variants
- Source coverage check: logs trace back to navball-bot runs (scenario E3), not human gameplay or Lambert-MPC baseline episodes
- Fine-tuned LLaMA performance can be compared against the navball bot's own scores (e.g. Navball (bot) 34.34 36.43) to sanity-check transfer

## Limitations

- KSPDG is explicitly an evaluation, not training, environment; all logs come from scripted navball-bot runs, limiting strategy diversity
- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) and local training hardware are not reported, and outcomes are 'highly hardware-dependent'
- The 'none' label conflates genuinely optimal no-action steps with failed/invalid action invocations from the 'Return None or a default action' failure path

## Evidence

- [methods] navball bot dataset provenance: [evidence span withheld — anchor status: not_found]
- [readme] KSPDG evaluation-not-training principle: [evidence span withheld — anchor status: not_found]
- [results] dataset hosting: "the models developed and used in this research are hosted on Hugging Face"
