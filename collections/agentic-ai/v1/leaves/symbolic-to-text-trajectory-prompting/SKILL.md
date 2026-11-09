---
name: symbolic-to-text-trajectory-prompting
description: Use when you need to synthesize symbolic expert trajectories (e.g., via
  an A* oracle over a grid transition graph) and render them as prompt-action pairs
  to fine-tune a language model as a next-action predictor.
when_to_use_negative:
- Input is not a discrete symbolic planning domain with a deterministic transition
  graph over grid states and actions
- You need the downstream RL policy itself (PPO) or the uncertainty blending mechanism
  (MC dropout, entropy-based blending) rather than trajectory-to-prompt data generation
- A meaningful admissible heuristic (e.g., Manhattan distance) cannot be defined,
  making A* oracle synthesis infeasible
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3372
tools:
- name: BERT (bert-base-uncased)
  role: Fine-tuned as the next-action predictor on the textual prompt–trajectory pairs
    generated from the A* oracle
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: MiniGrid
  role: Source environment (UnlockPickup benchmark) whose grid states and actions
    are serialized into symbolic trajectories and prompts
- name: ULPS
  role: Framework consuming the generated prompts; main implementation and experiment
    runners live under MainCode/
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:e893cdacaeddc149c6c3b9172afa77c3d957c711e9dd5a28e4db3b6e7e6d9836
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - BERT (bert-base-uncased)
  - MiniGrid
  - ULPS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# symbolic-to-text-trajectory-prompting

## Summary

Converts A*-synthesized optimal symbolic state-action trajectories from a MiniGrid transition graph into textual prompts used to fine-tune a BERT-based next-action predictor. This bridges discrete symbolic planning with language-model conditioning, enabling the ULPS LLM guidance component.

## When to use

Use when you need to synthesize symbolic expert trajectories (e.g., via an A* oracle over a grid transition graph) and render them as prompt-action pairs to fine-tune a language model as a next-action predictor.

## When NOT to use

- Input is not a discrete symbolic planning domain with a deterministic transition graph over grid states and actions
- You need the downstream RL policy itself (PPO) or the uncertainty blending mechanism (MC dropout, entropy-based blending) rather than trajectory-to-prompt data generation
- A meaningful admissible heuristic (e.g., Manhattan distance) cannot be defined, making A* oracle synthesis infeasible

## Inputs

- MiniGrid-UnlockPickup environment (4x4 and 8x8 grid layouts)
- Transition graph over discrete grid cell states and actions
- A* oracle with Manhattan distance heuristic (optimal symbolic state-action sequences)

## Outputs

- Textual prompt–trajectory pairs derived from symbolic state-action sequences
- Fine-tuned bert-base-uncased next-action predictor checkpoint
- Training/accuracy log (held-out prediction accuracy, e.g., 99.17%)

## How to apply

Instantiate the MiniGrid-UnlockPickup environment (4x4 or 8x8 layouts via run_4x4_experiment.py / run_8x8_experiment.py utilities) and represent the grid world as a transition graph over discrete cell states and actions. Run an A* oracle with the Manhattan distance heuristic over this graph to synthesize optimal symbolic trajectories as structured state-action sequences. Convert each symbolic state-action trajectory into a textual prompt suitable for language-model conditioning, then fine-tune bert-base-uncased as a next-action predictor on the generated prompt-trajectory pairs. Evaluate on held-out oracle trajectories; success is validated when held-out next-action prediction accuracy reaches the reported 99.17% on the 4x4 environment, a >9% improvement over prior calibrated-LLM work.

## Related tools

- **BERT (bert-base-uncased)** (Fine-tuned as the next-action predictor on the textual prompt–trajectory pairs generated from the A* oracle) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **MiniGrid** (Source environment (UnlockPickup benchmark) whose grid states and actions are serialized into symbolic trajectories and prompts)
- **ULPS** (Framework consuming the generated prompts; main implementation and experiment runners live under MainCode/) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl

## Examples

```
python MainCode/run_4x4_experiment.py
```

## Evaluation signals

- Held-out next-action prediction accuracy of the fine-tuned BERT model reaches 99.17% on the 4x4 MiniGrid environment
- Fine-tuned model shows >9% improvement in execution accuracy over prior calibrated-LLM work (90%)
- Generated prompts correctly render the A* oracle's optimal state-action sequences (structured state-action pairs, Manhattan-distance-optimal)
- Fine-tuned predictor is usable downstream in ULPS, where it combines with MC dropout uncertainty and entropy-based blending with PPO

## Limitations

- Not all result files are regenerated automatically by a single script; some CSV/plots were pre-generated during experimentation and included under results/
- Some plotting and summarization scripts assume the same output folder structure used during the experiments; minor path adjustments may be required
- Only a subset of development plots was selected for the final paper for compactness; no changelog found in the repository

## Evidence

- [methods] Environment instantiation and trajectory-to-prompt conversion workflow: [evidence span withheld — anchor status: not_found]
- [results] Fine-tuned predictor validated at 99.17% accuracy: [evidence span withheld — anchor status: not_found]
