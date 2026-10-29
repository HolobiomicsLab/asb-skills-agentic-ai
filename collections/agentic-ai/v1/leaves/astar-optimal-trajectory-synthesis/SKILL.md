---
name: astar-optimal-trajectory-synthesis
description: Use when a sparse-reward RL pipeline needs symbolic expert demonstration
  data to fine-tune a language-model action predictor, and the environment is a discrete
  grid world (e.g. MiniGrid 4x4/8x8 UnlockPickup) representable as a transition graph.
when_to_use_negative:
- The task environment is not a discrete grid world representable as a transition
  graph over cell states and actions (A* over a Manhattan heuristic will not yield
  meaningful optimal trajectories).
- The goal is to evaluate the full ULPS policy against RL baselines (Unguided PPO,
  Linear Decay, Q-Learning, DQN) rather than to generate fine-tuning data for the
  action predictor.
- Environment dynamics are stochastic or continuous, so a single optimal symbolic
  trajectory per start-goal pair is not well defined.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: BERT (bert-base-uncased)
  role: Language model fine-tuned as a next-action predictor on the A*-synthesized
    symbolic prompt trajectories
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: MiniGrid
  role: Grid-world benchmark (MiniGrid-UnlockPickup, 4x4 and 8x8) whose transition
    graph the A* oracle plans over
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: ULPS
  role: Framework whose downstream BERT + PPO pipeline consumes the synthesized trajectories;
    experiment runners live in MainCode/
  repo: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:aba33eadefd63aa208aa42df7a84000a61d552f5a06c51d33df69b83111de972
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
  tools:
  - BERT (bert-base-uncased)
  - MiniGrid
  - ULPS
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# astar-optimal-trajectory-synthesis

## Summary

Synthesizes optimal symbolic state-action trajectories for the MiniGrid-UnlockPickup environment using an A* oracle with a Manhattan distance heuristic, and converts them into textual prompts used to fine-tune a bert-base-uncased next-action predictor. This is the data-generation stage of ULPS that replaces costly environment interactions with oracle-derived supervision.

## When to use

Use when a sparse-reward RL pipeline needs symbolic expert demonstration data to fine-tune a language-model action predictor, and the environment is a discrete grid world (e.g. MiniGrid 4x4/8x8 UnlockPickup) representable as a transition graph.

## When NOT to use

- The task environment is not a discrete grid world representable as a transition graph over cell states and actions (A* over a Manhattan heuristic will not yield meaningful optimal trajectories).
- The goal is to evaluate the full ULPS policy against RL baselines (Unguided PPO, Linear Decay, Q-Learning, DQN) rather than to generate fine-tuning data for the action predictor.
- Environment dynamics are stochastic or continuous, so a single optimal symbolic trajectory per start-goal pair is not well defined.

## Inputs

- MiniGrid-UnlockPickup environment instance (4x4 and 8x8 layouts)
- Grid world represented as a transition graph over discrete cell states and actions
- Manhattan distance heuristic configuration for the A* oracle

## Outputs

- Optimal symbolic trajectories as structured state-action sequences
- Textual prompt-trajectory pairs suitable for BERT fine-tuning
- Fine-tuned bert-base-uncased next-action predictor checkpoint
- Training/accuracy log (held-out prediction accuracy)

## How to apply

Instantiate the MiniGrid-UnlockPickup environment (4x4 or 8x8 layout via MainCode/run_4x4_experiment.py or run_8x8_experiment.py utilities) and represent the grid as a transition graph over discrete cell states and actions. Run the A* oracle with the Manhattan distance heuristic over this graph to synthesize optimal symbolic trajectories as structured state-action sequences. Convert each symbolic state-action trajectory into a textual prompt suitable for language-model conditioning. Fine-tune bert-base-uncased as a next-action predictor on the prompt-trajectory pairs, then evaluate on held-out oracle trajectories, saving the checkpoint plus a training/accuracy log. Validation: held-out next-action prediction accuracy reaches 99.17% on the 4x4 MiniGrid environment, a more than 9% improvement over the 90% accuracy reported by prior calibrated-LLM work.

## Related tools

- **BERT (bert-base-uncased)** (Language model fine-tuned as a next-action predictor on the A*-synthesized symbolic prompt trajectories) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **MiniGrid** (Grid-world benchmark (MiniGrid-UnlockPickup, 4x4 and 8x8) whose transition graph the A* oracle plans over) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **ULPS** (Framework whose downstream BERT + PPO pipeline consumes the synthesized trajectories; experiment runners live in MainCode/) — https://github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl

## Examples

```
python MainCode/run_4x4_experiment.py  # runs the A* oracle trajectory synthesis + BERT fine-tuning pipeline and emits episode-level CSV logs and summaries
```

## Evaluation signals

- Held-out next-action prediction accuracy of the fine-tuned BERT reaches approximately 99.17% on the 4x4 MiniGrid environment
- Reported accuracy exceeds the 90% of prior calibrated-LLM work by more than 9 percentage points
- Each synthesized trajectory is an optimal A* solution under the Manhattan distance heuristic on the grid transition graph
- Every symbolic state-action sequence is convertible into a textual prompt without losing the state-action structure
- run_4x4_experiment.py / run_8x8_experiment.py generate episode-level CSV logs and summaries without errors

## Limitations

- Not all result files are regenerated automatically by a single script; some results (CSV files and plots) were pre-generated during experimentation and are included under results/.
- Some plotting and summarization scripts assume the same output folder structure used during the experiments and may require minor path adjustments.
- Synthesis is demonstrated only on the MiniGrid-UnlockPickup benchmark (4x4 and 8x8); transfer to other environments is not established.
- No changelog found in the repository.

## Evidence

- [abstract] A* oracle synthesizes symbolic trajectories for BERT fine-tuning: "ULPS employs an A*-based oracle to synthesize optimal symbolic trajectories, which are used to fine-tune a BERT-based language model"
- [abstract] Accuracy of the fine-tuned predictor: "ULPS achieves more than 9% improvement in execution accuracy after fine-tuning, requires fewer environment interactions, and yields higher reward AUC"
- [methods] Experiment runner scripts: "Example scripts:
- `MainCode/run_4x4_experiment.py`
- `MainCode/run_8x8_experiment.py`

These scripts generate episode-level CSV logs and summaries."
- [intro] Framework components consuming the trajectories: "The project implements ULPS, a framework that combines:
- A fine-tuned BERT-based LLM (action predictor)
- Monte Carlo Dropout for uncertainty estimation"
- [readme] Reproducibility caveats for the pipeline outputs: "Some results (CSV files and plots) were generated during experimentation and are included under `results/`.
- Not all result files are regenerated automatically by a single script."
