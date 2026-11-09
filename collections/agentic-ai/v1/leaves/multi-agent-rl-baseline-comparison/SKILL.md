---
name: multi-agent-rl-baseline-comparison
description: Use when a new MARL policy-optimization method must be validated against
  trust-region baselines on cooperative benchmarks, with training-convergence stability
  as a key evaluation axis.
when_to_use_negative:
- When the benchmark environments are MPE, MuJoCo, or MAMuJoCo rather than SMAC —
  the pinned dependency set (gym==0.21.0, pyglet==1.5.0) and scenario configs target
  SMAC starcraft2 maps
- When the goal is a statistical significance test of final performance rather than
  a qualitative reproduction of training-convergence curves
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: clip-HAPPO
  role: Proposed clipped sequential-update algorithm trained on the SMAC scenarios
    and compared in the win-rate curves
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HATRPO
  role: Proposed clipped trust-region variant trained alongside clip-HAPPO for comparison
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HAPPO
  role: Baseline heterogeneous-agent PPO algorithm trained on the same scenarios;
    known to exhibit unstable convergence in 27m_vs_30m
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HATRPO
  role: Baseline heterogeneous-agent trust-region algorithm trained on the same scenarios
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: MAPPO
  role: Multi-agent PPO baseline trained on the same scenarios
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Installs the codebase (`pip install -e .`), SMAC, and the pinned dependency
    versions
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:a372108310535b299c1ab350cf82014f3be287d03e130a7bbf519c148b4caaf9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - HATRPO
  - MAPPO
  - clip-HAPPO
  - clip-HATRPO
  - pip
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# multi-agent-rl-baseline-comparison

## Summary

Benchmark a proposed multi-agent RL algorithm (clip-HAPPO, clip-HATRPO) against trust-region baselines (HAPPO, HATRPO, MAPPO) on SMAC scenarios by training all algorithms from shared run_configs and comparing test win-rate-versus-iteration curves.

## When to use

Use when a new MARL policy-optimization method must be validated against trust-region baselines on cooperative benchmarks, with training-convergence stability as a key evaluation axis.

## When NOT to use

- When the benchmark environments are MPE, MuJoCo, or MAMuJoCo rather than SMAC — the pinned dependency set (gym==0.21.0, pyglet==1.5.0) and scenario configs target SMAC starcraft2 maps
- When the goal is a statistical significance test of final performance rather than a qualitative reproduction of training-convergence curves

## Inputs

- run_configs config files for each algorithm-scenario pair (clip-HAPPO, clip-HATRPO, HAPPO, HATRPO, MAPPO)
- SMAC scenarios: 27m_vs_30m, 3s5z_vs_3s6z, corridor
- Installed SMAC environment with SC2 maps in $SC2PATH/Maps

## Outputs

- Per-iteration test win-rate logs for each algorithm-scenario pair
- figure3_win_rate_curves.png: win-rate-versus-iteration curves for all five algorithms per scenario

## How to apply

Install the Low-Variance-Trust-Region-MARL codebase with `pip install -e .`, install SMAC following oxwhirl/smac instructions, and pin remaining dependencies (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0). Train each algorithm-scenario pair with `python -m examples.train --load_config <CONFIG PATH>` using the provided run_configs entries; each run instantiates RUNNER_REGISTRY[args['algo']] and calls runner.run() followed by runner.close(). Evaluate policies periodically during training to record test win rate at each iteration for clip-HAPPO, clip-HATRPO, HAPPO, HATRPO, and MAPPO on 27m_vs_30m, 3s5z_vs_3s6z, and corridor. Aggregate win-rate logs per scenario and plot win-rate-versus-iteration curves for all five algorithms to reproduce the paper's comparison figure.

## Related tools

- **clip-HAPPO** (Proposed clipped sequential-update algorithm trained on the SMAC scenarios and compared in the win-rate curves) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HATRPO** (Proposed clipped trust-region variant trained alongside clip-HAPPO for comparison) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HAPPO** (Baseline heterogeneous-agent PPO algorithm trained on the same scenarios; known to exhibit unstable convergence in 27m_vs_30m) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HATRPO** (Baseline heterogeneous-agent trust-region algorithm trained on the same scenarios) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **MAPPO** (Multi-agent PPO baseline trained on the same scenarios) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Installs the codebase (`pip install -e .`), SMAC, and the pinned dependency versions) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL

## Examples

```
python -m examples.train --load_config run_configs/smac/clip_happo_27m_vs_30m.
```

## Evaluation signals

- Reproduced win-rate curves qualitatively match the paper's Figure 3: clipping-based methods (clip-HAPPO, clip-HATRPO) outperform HAPPO, HATRPO, MAPPO in most scenarios
- HAPPO shows unstable convergence in the 27m_vs_30m scenario, consistent with the reported figure
- Test win rate is recorded at every training iteration for all five algorithm-scenario pairs (15 curves total)
- Each training run completes runner.run() and runner.close() without environment errors on the SMAC maps

## Limitations

- SC2 version matters: SMAC README warns performance is not always comparable between versions and the original SMAC paper results used SC2.4.6.2.69232, not SC2.4.10
- Exponential advantage-variance growth in sequential updates is only a pessimistic upper bound; practical variance depends on environment types and hyperparameters
- The clipping threshold of 1 for all agents may be pessimistic in environments with less severe variance issues, potentially dampening exploration signal

## Evidence
