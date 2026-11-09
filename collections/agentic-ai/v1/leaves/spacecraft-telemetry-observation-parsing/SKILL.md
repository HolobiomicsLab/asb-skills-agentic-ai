---
name: spacecraft-telemetry-observation-parsing
description: Use when raw kspdg episode logs (human gameplay or navball-bot data)
  must be rendered into natural-language observation/action pairs to interface the
  KSPDG simulation with an LLM in a classic RL loop.
when_to_use_negative:
- When you need a large-scale RL training environment — KSPDG explicitly provides
  evaluation environments, NOT training environments, since KSP was not designed for
  massively parallel, faster-than-real-time, headless execution
- When the input is already a fine-tuning JSONL with system/user/assistant roles and
  perform_action labels
- When observations are not KSPDG differential-game telemetry (e.g. multi-agent LBG1
  or sun-blocking scenarios with different observation spaces) without adapting the
  prompt template
tools:
- name: KSPDG
  role: Source of the simulation environment and episode logs whose real-time observations
    are transformed into textual user prompts
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: tiktoken
  role: OpenAI BPE tokeniser used to measure/validate the token length of the rendered
    observation prompts before fine-tuning
  repo: https://github.com/openai/tiktoken
- name: Hugging Face
  role: Hosts the resulting fine-tuning datasets built from the parsed telemetry
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:d9f9a8e5e38a2510a359e40fbbfcd83a4803b08b43ebc98751feb4676b6015bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - Hugging Face
  - KSPDG
  - tiktoken
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spacecraft-telemetry-observation-parsing

## Summary

Converts real-time KSPDG spacecraft telemetry (relative positions, velocities, fuel, prograde) into textual user prompts and paired vessel-frame thrust actions (perform_action calls), serialized as OpenAI-style JSONL with system/user/assistant roles for LLM prompt engineering or fine-tuning.

## When to use

Use when raw kspdg episode logs (human gameplay or navball-bot data) must be rendered into natural-language observation/action pairs to interface the KSPDG simulation with an LLM in a classic RL loop.

## When NOT to use

- When you need a large-scale RL training environment — KSPDG explicitly provides evaluation environments, NOT training environments, since KSP was not designed for massively parallel, faster-than-real-time, headless execution
- When the input is already a fine-tuning JSONL with system/user/assistant roles and perform_action labels
- When observations are not KSPDG differential-game telemetry (e.g. multi-agent LBG1 or sun-blocking scenarios with different observation spaces) without adapting the prompt template

## Inputs

- kspdg episode logs from the agents_data folder (human gameplay and navball-bot data)
- Observation dictionaries containing relative positions, velocities, fuel, and prograde values sampled every 0.5 seconds
- Discrete thrust action labels relative to the spacecraft's reference frame
- System prompt describing the autonomous pursuit agent and vessel-frame throttle axes (x right, y toward target, z up, max throttle 1)

## Outputs

- JSONL file (fine_tuning_data.l) with one JSON line per timestep, each containing 'system', 'user', and 'assistant' role fields
- Assistant responses embedding well-formed perform_action(ft, rt, dt) thrust calls

## How to apply

Load the recorded gameplay episode logs from the kspdg agents_data folder, which consist of observations and thrust actions recorded every 0.5 seconds during gameplay. For each logged timestep, render the observation dictionary (relative positions, velocities, fuel, prograde) into the user prompt template 'Given these observations ..., what is the best throttle to capture evader?' with step-by-step reasoning requested, and prepend the fixed system prompt describing the autonomous pursuit agent and the vessel-frame throttle axes (x right, y toward target, z up, max throttle 1). Format the paired discrete thrust label — logged relative to the spacecraft's reference frame — as the assistant response ending in a perform_action(ft, rt, dt) call, delimited with the HUMAN/ASSISTANT keywords shown in the prompt templates. Serialize each timestep as one JSON line containing the 'system', 'user', and 'assistant' role fields; use tiktoken to keep prompts within model token budgets. The rationale is that KSPDG's non-blocking step function penalizes long computation, so observation text must be compact and pre-templated rather than generated ad hoc at inference time.

## Related tools

- **KSPDG** (Source of the simulation environment and episode logs whose real-time observations are transformed into textual user prompts) — https://github.com/mit-ll/spacegym-kspdg
- **tiktoken** (OpenAI BPE tokeniser used to measure/validate the token length of the rendered observation prompts before fine-tuning) — https://github.com/openai/tiktoken
- **Hugging Face** (Hosts the resulting fine-tuning datasets built from the parsed telemetry)

## Examples

```
import tiktoken; enc = tiktoken.encoding_for_model("gpt-3.5-turbo"); n_tokens = len(enc.encode(system_prompt + user_prompt))
```

## Evaluation signals

- Every output line of fine_tuning_data.l parses as valid JSON
- Each line contains all three role fields: 'system', 'user', and 'assistant'
- The assistant field embeds a well-formed perform_action(ft, rt, dt) thrust label
- Sampling cadence of records matches the 0.5-second gameplay logging interval
- Throttle labels respect the vessel-frame axes convention (x right, y toward target, z up) and max throttle of 1

## Limitations

- KSP game time does not pause while agents make decisions (non-blocking step function), so slow prompt rendering directly penalizes the control policy
- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are not reported for the underlying conversion/training pipeline
- Few-shot agent example was marked '[TO BE DONE]' in the paper, so no fully worked few-shot prompt template is documented
- Physical-range extension via PhysicsRangeExtender can introduce vessel shaking, phantom forces, and landed-vessel collisions, which may corrupt telemetry at ranges > 100 km

## Evidence

- [intro] Observation-to-prompt transformation: "we design the classic RL loop by interfacing the simulation environment (KSDPG) with a LLM, transforming the real-time observations (or state) of the mission as textual user prompts"
- [methods] 0.5 s logging cadence: [evidence span withheld — anchor status: not_found]
- [readme] Non-blocking step function penalty: [evidence span withheld — anchor status: not_found]
- [readme] Evaluation-not-training scope: [evidence span withheld — anchor status: not_found]
- [readme] Tokeniser role: [evidence span withheld — anchor status: not_found]
