---
name: chat-format-dataset-serialization
description: Use when raw sequential environment logs (timestep observations plus
  action labels) must be reformatted into chat-role JSONL for LLM fine-tuning, e.g.
  imitation of pursuit-evasion piloting.
when_to_use_negative:
- The target is evaluation rather than training — KSPDG is explicitly designed to
  provide evaluation environments, NOT training environments for reinforcement learning
- Input logs are already in chat/JSONL format with the three role fields — no reformatting
  needed
- Actions are continuous control vectors rather than discrete vessel-frame throttle
  labels that can be rendered as perform_action calls
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3366
tools:
- name: KSPDG
  role: Source of the recorded gameplay logs and observation/action schemas that the
    serialization pipeline consumes
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: Hugging Face
  role: Hosts the resulting fine-tuning datasets (LLaMA 10/25/50-file dataset files)
    produced by the serialization workflow
  repo: https://huggingface.co
- name: openai/tiktoken
  role: Tokeniser used in the article's methods for measuring token counts of the
    formatted prompts/dataset
  repo: https://github.com/openai/tiktoken
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:6d4b1f0cfe6d0fefb8438f60b2e0ebb6951cd1e186a59f29444f3c6aef156abf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - Hugging Face
  - KSPDG
  - openai/tiktoken
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Chat-format dataset serialization

## Summary

Converts recorded agent/human gameplay episodes (e.g. KSPDG pursuit logs) into OpenAI-style fine-tuning JSONL where each timestep becomes one line with 'system', 'user', and 'assistant' role fields, pairing rendered telemetry observations with discrete thrust action labels.

## When to use

Use when raw sequential environment logs (timestep observations plus action labels) must be reformatted into chat-role JSONL for LLM fine-tuning, e.g. imitation of pursuit-evasion piloting.

## When NOT to use

- The target is evaluation rather than training — KSPDG is explicitly designed to provide evaluation environments, NOT training environments for reinforcement learning
- Input logs are already in chat/JSONL format with the three role fields — no reformatting needed
- Actions are continuous control vectors rather than discrete vessel-frame throttle labels that can be rendered as perform_action calls

## Inputs

- Recorded gameplay episode logs from the kspdg agents_data folder (human gameplay and navball-bot data)
- Per-timestep observation dictionaries (relative positions, velocities, fuel, prograde) sampled every 0.5 seconds
- Discrete thrust action labels relative to the spacecraft's reference frame (ft, rt, dt)

## Outputs

- fine_tuning_data.l — a JSONL file with one line per timestep, each containing 'system', 'user', and 'assistant' role fields
- Assistant responses embedding well-formed perform_action(ft, rt, dt) thrust labels

## How to apply

Load the recorded gameplay episode logs (human gameplay and navball-bot data from the kspdg agents_data folder) and for each logged timestep (sampled every 0.5 s) render the observation dictionary — relative positions, velocities, fuel, prograde — into a user prompt such as 'Given these observations ..., what is the best throttle to capture evader?' with step-by-step reasoning requested. Prepend a fixed system prompt describing the autonomous pursuit agent and the vessel-frame throttle axes (x right, y toward target, z up, max throttle 1). Format the paired thrust action as the assistant response ending in a perform_action(ft, rt, dt) call, delimited with HUMAN/ASSISTANT keywords. Serialize each timestep as one JSON line with the three role fields, saved as fine_tuning_data.l. The rationale is that the KSPDG loop transforms real-time observations into textual user prompts and plugs the model's action back into the environment, so fine-tuning data must mirror that exact text interface.

## Related tools

- **KSPDG** (Source of the recorded gameplay logs and observation/action schemas that the serialization pipeline consumes) — https://github.com/mit-ll/spacegym-kspdg
- **Hugging Face** (Hosts the resulting fine-tuning datasets (LLaMA 10/25/50-file dataset files) produced by the serialization workflow) — https://huggingface.co
- **openai/tiktoken** (Tokeniser used in the article's methods for measuring token counts of the formatted prompts/dataset) — https://github.com/openai/tiktoken

## Examples

```
import ; rows=[{'system':sys_prompt,'user':fmt_obs(o),'assistant':fmt_action(a)} for o,a in timesteps]; open('fine_tuning_data.l','w').write('\n'.join(.dumps(r) for r in rows))
```

## Evaluation signals

- Every output line in fine_tuning_data.l parses as valid JSON
- Each JSON line contains all three role fields: 'system', 'user', and 'assistant'
- The 'assistant' field embeds a well-formed perform_action(ft, rt, dt) thrust label
- The 'user' field reflects the logged observation dictionary (relative positions, velocities, fuel, prograde) at the 0.5 s sampling cadence
- Serialized dataset files, when used for fine-tuning, yield measurable model performance (e.g. fine-tune 50 files: mean reward 11.86 vs navball bot 34.34 as reported baseline context)

## Limitations

- All dataset files in the article's LLaMA fine-tuning experiments were collected using the navball bot, so the format encodes bot-like rather than fully human-diverse behavior; adding human gameplays improved training loss (1.3e-4) but worsened validation loss (4.4e-1)
- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are not reported in the source, limiting exact reproducibility of downstream training
- Serialization alone does not guarantee performance — dataset size matters (10 vs 25 vs 50 files showed large reward differences)

## Evidence

- [intro] Interface the KSPDG simulation environment with an LLM in a classic RL loop, transforming real-time observations into textual user prompts and plugging the model's action back into KSPDG: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [methods] Human gameplay data collected for fine-tuning: "This section houses all the data generated from human gameplay. It is crucial for understanding human strategies and behaviors"
- [methods] Dataset files collected with navball bot: [evidence span withheld — anchor status: not_found]
- [abstract] Fine-tuning techniques among the paper's approaches: "Our approach leverages prompt engineering, few-shot prompting, and fine-tuning techniques"
- [readme] KSPDG provides standardized Gym API for agent interactions: [evidence span withheld — anchor status: not_found]
