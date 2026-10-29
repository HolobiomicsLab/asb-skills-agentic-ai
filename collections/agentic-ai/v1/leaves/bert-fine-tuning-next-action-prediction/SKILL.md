---
name: bert-fine-tuning-next-action-prediction
description: Use when you have discrete symbolic optimal trajectories (state–action
  sequences from a graph search oracle) and need a language model to imitate them
  for downstream policy guidance in RL.
when_to_use_negative:
- The environment is continuous or partially observable rather than a discrete-grid
  transition graph — the A* oracle and Manhattan heuristic require a discrete state/action
  graph
- Optimal trajectories cannot be synthesized (reward/cost function undefined for graph
  search) — BERT fine-tuning here is supervised on oracle-generated data, not on raw
  environment interactions
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: BERT (bert-base-uncased)
  role: Language model fine-tuned as the next-action predictor on oracle prompt–trajectory
    pairs
  repo: github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: MiniGrid
  role: Grid-world benchmark (MiniGrid-UnlockPickup) whose discrete transition graph
    supplies the state–action training data
  repo: github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- name: ULPS
  role: Downstream consumer of the fine-tuned predictor, whose influence is conditioned
    on MC-dropout epistemic uncertainty and entropy-based blending with PPO
  repo: github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1109/cai68641.2026.11536354
    title: Uncertainty-Aware LLM-Guided Policy Shaping for Sparse-Reward Reinforcement
      Learning
  source_content_hash: sha256:c8a91a7d503a715a690b381410471d55f881b7bba317f29c4fce492c31fcefaa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BERT (bert-base-uncased)
  - MiniGrid
  - ULPS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# bert-fine-tuning-next-action-prediction

## Summary

Fine-tunes bert-base-uncased as a next-action predictor on symbolic state–action trajectories synthesized by an A*-based oracle over a MiniGrid-UnlockPickup transition graph, reaching 99.17% held-out prediction accuracy on the 4×4 environment (>9% above prior calibrated-LLM work's 90%).

## When to use

Use when you have discrete symbolic optimal trajectories (state–action sequences from a graph search oracle) and need a language model to imitate them for downstream policy guidance in RL.

## When NOT to use

- The environment is continuous or partially observable rather than a discrete-grid transition graph — the A* oracle and Manhattan heuristic require a discrete state/action graph
- Optimal trajectories cannot be synthesized (reward/cost function undefined for graph search) — BERT fine-tuning here is supervised on oracle-generated data, not on raw environment interactions

## Inputs

- Transition graph over discrete MiniGrid cell states and actions (4x4 and 8x8 MiniGrid-UnlockPickup layouts)
- Symbolic state–action trajectories synthesized by an A*-based oracle with Manhattan distance heuristic
- Textual prompt versions of each symbolic state–action trajectory (language-model conditioning format)

## Outputs

- Fine-tuned bert-base-uncased next-action predictor checkpoint
- Training/accuracy log for the fine-tuning run
- Held-out next-action prediction accuracy (99.17% expected on the 4×4 MiniGrid environment)

## How to apply

Represent the MiniGrid-UnlockPickup grid (4×4 or 8×8 via run_4x4_experiment.py / run_8x8_experiment.py utilities) as a transition graph over discrete cell states and actions. Run the A* oracle with a Manhattan distance heuristic over this graph to synthesize optimal symbolic trajectories as structured state–action sequences. Convert each symbolic trajectory into a textual prompt suitable for language-model conditioning, then fine-tune bert-base-uncased on the generated prompt–trajectory pairs with the correct next action as the label. Evaluate on held-out oracle trajectories and persist the model checkpoint plus a training/accuracy log.

## Related tools

- **BERT (bert-base-uncased)** (Language model fine-tuned as the next-action predictor on oracle prompt–trajectory pairs) — github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **MiniGrid** (Grid-world benchmark (MiniGrid-UnlockPickup) whose discrete transition graph supplies the state–action training data) — github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl
- **ULPS** (Downstream consumer of the fine-tuned predictor, whose influence is conditioned on MC-dropout epistemic uncertainty and entropy-based blending with PPO) — github.com/USD-AI-ResearchLab/uncertainty-aware-llm-rl

## Examples

```
python MainCode/run_4x4_experiment.py
```

## Evaluation signals

- Held-out next-action prediction accuracy reaches 99.17% on the 4×4 MiniGrid environment
- Fine-tuned predictor outperforms the 90% accuracy of prior calibrated-LLM work by more than 9%
- Each symbolic trajectory converts losslessly into a prompt with a matching next-action label (state–action sequence integrity preserved)
- A model checkpoint plus a training/accuracy log are saved after the fine-tuning run

## Limitations

- Only a subset of development plots was selected for the final paper; not all result files are regenerated automatically by a single script
- Some results (CSV files and plots) were pre-generated during experimentation and included under results/ rather than reproduced on demand
- Some parts of the codebase may require minor path adjustments depending on the local environment

## Evidence

- [results] 99.17% accuracy finding: [evidence span withheld — anchor status: not_found]
- [full_text] Fine-tuning procedure from source card: [evidence span withheld — anchor status: not_found]
