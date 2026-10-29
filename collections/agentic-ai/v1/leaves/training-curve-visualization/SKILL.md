---
name: training-curve-visualization
description: Use when you have per-iteration episode-reward logs (and optionally advantage-variance-per-agent
  logs) from multiple seeds of MARL algorithm-task runs and need to produce comparison
  panels across algorithms such as MAPPO, HAPPO, HATRPO, clip-HAPPO, clip-HATRPO.
when_to_use_negative:
- You only have a single training run without multiple seeds, so seed-aggregated mean
  curves would misrepresent run-to-run variability
- The evaluation metric is not a per-iteration training signal (e.g. you need offline
  policy evaluation or final-policy benchmarks rather than reward-versus-iteration
  curves)
- The environment logs do not include per-agent advantage-variance estimates, so the
  variance-versus-agent-index panel (the key artifact for the Fig. 8 ablation) cannot
  be produced
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: HAPPO
  role: Baseline algorithm whose reward and advantage-variance training curves are
    plotted for comparison against the clipped variant
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HAPPO
  role: Proposed algorithm whose low-variance, stable-convergence curves are visualized
    side by side with HAPPO
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Installs the Low-Variance-Trust-Region-MARL codebase ('pip install -e .')
    and pinned dependencies (gym==0.21.0, pettingzoo==1.22.2, supersuit==3.7.0) needed
    to generate the logged curves
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_007
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:cb57c31008d99ceb0398f655bace06cd55307da33d7d6c9d9cd1c86718dd1a6c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - HATRPO
  - MAMuJoCo
  - MAPPO
  - clip-HAPPO
  - clip-HATRPO
  - pettingzoo
  - pip
  - supersuit
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# training-curve-visualization

## Summary

Aggregate logged episodic rewards and advantage-variance statistics across seeds and render mean reward-versus-iteration curves and variance-versus-agent-index panels in the style of Figures 4, 7, and 8 of the Low-Variance-Trust-Region-MARL paper, enabling visual comparison of HAPPO/HATRPO baselines against the clipped variants.

## When to use

Use when you have per-iteration episode-reward logs (and optionally advantage-variance-per-agent logs) from multiple seeds of MARL algorithm-task runs and need to produce comparison panels across algorithms such as MAPPO, HAPPO, HATRPO, clip-HAPPO, clip-HATRPO.

## When NOT to use

- You only have a single training run without multiple seeds, so seed-aggregated mean curves would misrepresent run-to-run variability
- The evaluation metric is not a per-iteration training signal (e.g. you need offline policy evaluation or final-policy benchmarks rather than reward-versus-iteration curves)
- The environment logs do not include per-agent advantage-variance estimates, so the variance-versus-agent-index panel (the key artifact for the Fig. 8 ablation) cannot be produced

## Inputs

- Logged episodic reward time series per iteration, per algorithm, per task, per seed (emitted by runner.run()/runner.close())
- Average estimated advantage variance per agent index, logged over training iterations (for HAPPO and clip-HAPPO on MAMuJoCo tasks such as Ant-v2-8x1 and Humanoid-v2-17x1)
- Experiment configuration paths from the run_configs folder identifying the algorithm-task-seed combinations

## Outputs

- Figure-4-style panel per MPE task with five mean reward-versus-iteration curves (one per algorithm)
- mujoco_reward_variance.png with reward-versus-iteration and average-variance-versus-agent-index curves for HAPPO and clip-HAPPO side by side
- Figure-7/C1-C2-style comparison panels of reward and variance curves under 10-thread and 5-thread budgets
- Aggregated (seed-averaged) training statistics per algorithm-task pair

## How to apply

After training each algorithm-task pair with 'python -m examples.train --load_config <CONFIG PATH>' and flushing logs via runner.close(), aggregate the logged episode rewards across seeds for each algorithm-task pair. For MPE tasks (simple_spread_v2, simple_reference_v2, simple_speaker_listener_v3), plot the mean reward-versus-iteration curve for all five algorithms (MAPPO, HATRPO, HAPPO, clip-HATRPO, clip-HAPPO) into a single Figure-4-style panel per task. For MAMuJoCo ablations, additionally plot average-estimated-advantage-variance-versus-agent-index curves for HAPPO vs clip-HAPPO side by side with the reward curves, saving the output as mujoco_reward_variance.png for the fixed-order-vs-random-order ablation (Fig. 8) and rendering per-thread-budget panels for the 10- and 5-environment ablations (Fig. 7, C1-C2). The rationale is that the paper's central claims — stable convergence, low advantage variance (Var ≤ ε^2), and win fractions such as 5/6 at 10 threads — are all communicated through these comparative curve panels, so correct aggregation and averaging across seeds is essential for faithful reproduction.

## Related tools

- **HAPPO** (Baseline algorithm whose reward and advantage-variance training curves are plotted for comparison against the clipped variant) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HAPPO** (Proposed algorithm whose low-variance, stable-convergence curves are visualized side by side with HAPPO) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Installs the Low-Variance-Trust-Region-MARL codebase ('pip install -e .') and pinned dependencies (gym==0.21.0, pettingzoo==1.22.2, supersuit==3.7.0) needed to generate the logged curves) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL

## Examples

```
python -m examples.train --load_config run_configs/mpe/simple_spread_v2/clip_happo.yaml  # then aggregate logged rewards across seeds and plot the Figure-4-style panel
```

## Evaluation signals

- Five distinct reward curves per MPE task are present in the Figure-4-style panel, one for each of MAPPO, HATRPO, HAPPO, clip-HATRPO, and clip-HAPPO
- The plotted curves exhibit the small performance spread reported for the simple MPE tasks, with clipped-objective methods showing stable convergence
- The variance panel shows HAPPO's advantage-variance estimate growing with agent index (up to ~1e6 on Humanoid-v2-17x1) while clip-HAPPO's stays bounded (Var ≤ ε^2) and does not grow with agent count
- Win-fraction checks against reported results: clip-HAPPO outperforms HAPPO in 5/6 MAMuJoCo environments at 10 threads and 4/6 at 5 threads
- Reward curves are seed-aggregated means, not single-seed traces, and runner.close() was called so the final episode-reward logs were flushed

## Limitations

- The exponential advantage-variance growth is an upper-bound (pessimistic) analysis; in practice growth depends on environment type and hyperparameters and does not always follow the worst case, so curve shapes may differ across environments
- HATRPO is less susceptible to high variance than HAPPO due to training and trust-region setups, so the variance panel may not separate the two TRPO-style baselines as sharply
- The fixed clipping threshold of 1 for all agents may appear pessimistic in environments with less severe variance issues, which can flatten the visible advantage of clip-HAPPO in low-variance tasks

## Evidence

- [other] Aggregate the logged episode rewards across seeds for each algorithm-task pair and plot the mean reward-versus-iteration curves per MPE task: [evidence span withheld — anchor status: not_found]
- [results] variance grows with agent index: [evidence span withheld — anchor status: not_found]
- [methods] variance bound: [evidence span withheld — anchor status: not_found]
