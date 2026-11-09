---
name: minigrid-gridworld-state-representation
description: Use when you need to convert a MiniGrid environment such as MiniGrid-UnlockPickup
  into a discrete state-action transition graph so that a graph-search oracle (e.g.,
  A*) can synthesize symbolic training trajectories.
when_to_use_negative:
- The task requires continuous state spaces or non-gridworld environments where discrete
  cell-state enumeration does not apply
- The goal is to evaluate the trained ULPS policy (Calibrated LLM + PPO blending)
  rather than to generate symbolic training data for the LLM predictor
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MiniGrid
  role: Provides the UnlockPickup gridworld environment (4x4 and 8x8) whose discrete
    cells and actions define the transition graph
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: ULPS
  role: Framework whose experiment runners (run_4x4_experiment.py, run_8x8_experiment.py)
    instantiate the environment and orchestrate trajectory generation
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:5abea3c0e429e6ebedafe5b7886fe7d70c6392cbc621d4ea50c782b4579f44bb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - MiniGrid
  - ULPS
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# minigrid-gridworld-state-representation

## Summary

Represent the MiniGrid-UnlockPickup grid world (4x4 and 8x8 layouts) as a transition graph over discrete cell states and actions, providing the structured input on which the A*-based oracle synthesizes optimal symbolic trajectories for BERT fine-tuning.

## When to use

Use when you need to convert a MiniGrid environment such as MiniGrid-UnlockPickup into a discrete state-action transition graph so that a graph-search oracle (e.g., A*) can synthesize symbolic training trajectories.

## When NOT to use

- The task requires continuous state spaces or non-gridworld environments where discrete cell-state enumeration does not apply
- The goal is to evaluate the trained ULPS policy (Calibrated LLM + PPO blending) rather than to generate symbolic training data for the LLM predictor

## Inputs

- MiniGrid-UnlockPickup environment instance (4x4 or 8x8 grid layout)
- Grid cell observations and available action set from the MiniGrid API

## Outputs

- Transition graph over discrete cell states and actions
- Structured discrete state-action space consumed by the A*-based oracle

## How to apply

Instantiate the MiniGrid-UnlockPickup environment in the desired layout using the repository's run_4x4_experiment.py or run_8x8_experiment.py utilities from the USD-AI-ResearchLab/uncertainty-aware-llm-rl codebase. Encode each grid cell as a discrete state and enumerate the available actions so that transitions between cell states form a transition graph. This graph is the substrate on which the A* oracle with Manhattan distance heuristic runs to produce structured state-action trajectories; those trajectories are later converted into textual prompts for fine-tuning bert-base-uncased as a next-action predictor. Correctness is judged by whether the resulting graph supports oracle trajectories that reach 99.17% held-out next-action prediction accuracy after BERT fine-tuning on the 4x4 environment.

## Related tools

- **MiniGrid** (Provides the UnlockPickup gridworld environment (4x4 and 8x8) whose discrete cells and actions define the transition graph) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **ULPS** (Framework whose experiment runners (run_4x4_experiment.py, run_8x8_experiment.py) instantiate the environment and orchestrate trajectory generation) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl

## Examples

```
python MainCode/run_4x4_experiment.py
```

## Evaluation signals

- The discrete state-action space covers all reachable cell states of the 4x4 or 8x8 UnlockPickup grid without unmodeled transitions
- A* with Manhattan distance heuristic returns optimal trajectories over the graph, consistent with the paper's oracle trajectory synthesis
- Downstream fine-tuned BERT predictor reaches the reported 99.17% next-action accuracy on held-out oracle trajectories for the 4x4 environment
- Episode-level CSV logs from run_4x4_experiment.py / run_8x8_experiment.py populate episode_metrics_*.csv consistent with the generated trajectories

## Limitations

- The representation is specific to discrete gridworlds like MiniGrid-UnlockPickup; the paper evaluates only 4x4 and 8x8 layouts
- Repository notes say some results were pre-generated during experimentation and not all result files are regenerated by a single script, so the graph/trajectory pipeline may require path adjustments

## Evidence

- [methods] Instantiate the MiniGrid-UnlockPickup environment (4x4 and 8x8 layouts via the repository's run_4x4_experiment.py / run_8x8_experiment.py utilities) and represent the grid world as a transition graph over discrete cell states and actions.: [evidence span withheld — anchor status: not_found]
