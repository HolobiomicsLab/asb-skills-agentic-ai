---
name: sparse-reward-rl-oracle-data-generation
description: 'Use when generating supervised training data for an LLM action prior
  in sparse-reward RL: the grid world (MiniGrid-UnlockPickup) is discrete and encodable
  as a transition graph, so optimal symbolic trajectories can be synthesized by an
  A* oracle.'
when_to_use_negative:
- The environment is not a discrete grid world representable as a transition graph
  over cell states and actions, so exhaustive A* planning cannot enumerate optimal
  paths.
- You need the full ULPS policy-shaping stage (MC-dropout uncertainty estimation,
  entropy-based blending with PPO) rather than just training-data synthesis — this
  skill stops at the fine-tuned predictor.
- You are running RL baselines (Unguided PPO, Linear Decay, Q-Learning, DQN) that
  by design receive no LLM guidance, so no oracle-generated data is used.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: A*-based oracle
  role: Plans optimal state-action paths over the transition graph using the Manhattan-distance
    heuristic to synthesize symbolic training trajectories
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: BERT (bert-base-uncased)
  role: Fine-tuned on the synthesized prompt-trajectory pairs as the next-action predictor
    (99.17% accuracy on the 4x4 grid)
- name: MiniGrid
  role: Supplies the MiniGrid-UnlockPickup benchmark (4x4 and 8x8 layouts) whose discrete
    cell states and actions define the transition graph
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:8654c968e99cccb4377205e70316c29dc18e8b2269da0b619598c05b017932e3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - A*-based oracle
  - BERT (bert-base-uncased)
  - MiniGrid
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sparse-reward-rl-oracle-data-generation

## Summary

Synthesizes the symbolic training data behind ULPS's BERT action prior: an A*-based oracle with a Manhattan-distance heuristic derives optimal state-action trajectories from a MiniGrid-UnlockPickup transition graph, converts them to textual prompts, and fine-tunes bert-base-uncased as a next-action predictor reaching 99.17% on the 4x4 grid (a >9% gain over prior calibrated-LLM work).

## When to use

Use when generating supervised training data for an LLM action prior in sparse-reward RL: the grid world (MiniGrid-UnlockPickup) is discrete and encodable as a transition graph, so optimal symbolic trajectories can be synthesized by an A* oracle.

## When NOT to use

- The environment is not a discrete grid world representable as a transition graph over cell states and actions, so exhaustive A* planning cannot enumerate optimal paths.
- You need the full ULPS policy-shaping stage (MC-dropout uncertainty estimation, entropy-based blending with PPO) rather than just training-data synthesis — this skill stops at the fine-tuned predictor.
- You are running RL baselines (Unguided PPO, Linear Decay, Q-Learning, DQN) that by design receive no LLM guidance, so no oracle-generated data is used.

## Inputs

- MiniGrid-UnlockPickup environment instance (4x4 or 8x8 grid layout)
- Transition graph over discrete cell states and actions of the grid world
- Heuristic specification for the A* oracle (Manhattan distance)

## Outputs

- Optimal symbolic state-action trajectories (structured sequences)
- Textual prompts paired with symbolic trajectories for language-model conditioning
- Fine-tuned bert-base-uncased next-action predictor checkpoint
- Training/accuracy log for the fine-tuned predictor

## How to apply

Instantiate MiniGrid-UnlockPickup (4x4 or 8x8 layout via the repository's run_4x4_experiment.py / run_8x8_experiment.py runners) and encode the grid world as a transition graph over discrete cell states and actions. Run the A* oracle over this graph with the Manhattan-distance heuristic to synthesize optimal symbolic trajectories as structured state-action sequences — optimality is the rationale, since it provides a clean, conflict-free supervision signal for the action prior. Convert each symbolic state-action trajectory into a textual prompt suitable for language-model conditioning, then fine-tune bert-base-uncased as a next-action predictor on the prompt-trajectory pairs. Evaluate on held-out oracle trajectories and save the model checkpoint plus a training/accuracy log; treat held-out next-action accuracy of 99.17% on the 4x4 environment (a >9% improvement over the 90% of prior calibrated-LLM work) as the reproduction target. Keep the state-action schema consistent with ULPS's downstream stages (MC-dropout uncertainty estimation and entropy-based blending with the PPO policy), which consume this predictor at run time.

## Related tools

- **A*-based oracle** (Plans optimal state-action paths over the transition graph using the Manhattan-distance heuristic to synthesize symbolic training trajectories) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **BERT (bert-base-uncased)** (Fine-tuned on the synthesized prompt-trajectory pairs as the next-action predictor (99.17% accuracy on the 4x4 grid))
- **MiniGrid** (Supplies the MiniGrid-UnlockPickup benchmark (4x4 and 8x8 layouts) whose discrete cell states and actions define the transition graph)

## Examples

```
python MainCode/run_4x4_experiment.py
```

## Evaluation signals

- Held-out next-action prediction accuracy of the fine-tuned bert-base-uncased reaches the reported 99.17% on the 4x4 MiniGrid environment (vs the 90% of prior calibrated-LLM work).
- Every synthesized trajectory is optimal: A* with the Manhattan-distance heuristic over the transition graph, and each state-action step is executable in MiniGrid-UnlockPickup.
- Each symbolic trajectory converts to a textual prompt that preserves the state-action structure used during fine-tuning.
- Runner scripts run_4x4_experiment.py / run_8x8_experiment.py emit episode-level CSV logs and summaries in the expected output structure (e.g., episode_metrics_*.csv under results/).
- The model checkpoint plus a training/accuracy log are saved, and the logged accuracy matches the paper's reported fine-tuning accuracy.

## Limitations

- Not all result files are regenerated automatically by a single script; some CSVs and plots were pre-generated during experimentation and shipped under results/.
- Plotting and summarization scripts assume the same output folder structure used during the experiments, and the README notes some parts may require minor path adjustments locally.
- The pipeline is demonstrated on discrete MiniGrid-UnlockPickup grids (4x4 and 8x8) represented as transition graphs; it is not validated on continuous or non-enumerable state spaces.

## Evidence
