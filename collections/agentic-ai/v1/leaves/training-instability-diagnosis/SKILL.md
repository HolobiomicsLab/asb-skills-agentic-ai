---
name: training-instability-diagnosis
description: Use when multi-agent trust region training curves (e.g., HAPPO on SMAC
  scenarios like 27m_vs_30m) show erratic or non-convergent win rates, and you need
  to attribute the instability to advantage-estimate variance and test clipping as
  a fix.
when_to_use_negative:
- When instability is observed in single-agent PPO/TRPO training, not in multi-agent
  sequential-update trust region methods with independent actors
- When the environment's variance issues are mild — the paper notes fixing the clipping
  threshold to 1 for all agents may be pessimistic in such settings
- When the goal is hyperparameter tuning for a non-trust-region MARL baseline (e.g.,
  QMIX/COMA from PyMARL) rather than diagnosing advantage-variance-driven instability
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: HAPPO
  role: Suspect baseline algorithm whose unstable convergence in 27m_vs_30m is diagnosed
    via win-rate curves
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HAPPO
  role: Clipped sequential-update algorithm used to test whether clipping the advantage
    objective stabilizes training
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HATRPO
  role: Clipped trust region variant evaluated for stable convergence against HATRPO
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HATRPO
  role: Trust region baseline; less susceptible to high variance than HAPPO, used
    as a stability comparison
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: SMAC
  role: Benchmark environment (27m_vs_30m, 3s5z_vs_3s6z, corridor) on which convergence
    stability is measured
  repo: https://github.com/oxwhirl/smac
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:3de5e9b1b8afe4e7c9807b9911b401286d0e1d49158467ed96f58ce7b30ce7aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - HATRPO
  - MAPPO
  - SMAC
  - clip-HAPPO
  - clip-HATRPO
  - pip
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# training-instability-diagnosis

## Summary

Diagnoses and mitigates training instability in sequential-update trust region MARL algorithms (HAPPO/HATRPO) by attributing it to exponentially high variance in the updated advantage function, and by evaluating whether a clipped advantage objective (clip-HAPPO/clip-HATRPO) restores stable convergence.

## When to use

Use when multi-agent trust region training curves (e.g., HAPPO on SMAC scenarios like 27m_vs_30m) show erratic or non-convergent win rates, and you need to attribute the instability to advantage-estimate variance and test clipping as a fix.

## When NOT to use

- When instability is observed in single-agent PPO/TRPO training, not in multi-agent sequential-update trust region methods with independent actors
- When the environment's variance issues are mild — the paper notes fixing the clipping threshold to 1 for all agents may be pessimistic in such settings
- When the goal is hyperparameter tuning for a non-trust-region MARL baseline (e.g., QMIX/COMA from PyMARL) rather than diagnosing advantage-variance-driven instability

## Inputs

- Training run configs (run_configs entries) for HAPPO, HATRPO, MAPPO, clip-HAPPO, clip-HATRPO on SMAC scenarios 27m_vs_30m, 3s5z_vs_3s6z, corridor
- Per-iteration test win-rate logs recorded from periodic policy evaluation during training
- Advantage-function estimates from sequential updates with independent actors

## Outputs

- Win-rate-versus-iteration convergence curves per algorithm-scenario pair (e.g., figure3_win_rate_curves.png)
- Diagnosis of unstable convergence attributable to high advantage-estimate variance
- Comparison of training stability between clipped and unclipped sequential-update algorithms

## How to apply

First, train the suspect sequential-update algorithm (HAPPO or HATRPO) alongside candidate fixes (clip-HAPPO, clip-HATRPO, MAPPO) on the same SMAC scenarios (27m_vs_30m, 3s5z_vs_3s6z, corridor) using `python -m examples.train --load_config <CONFIG PATH>` with the repo's run_configs entries, invoking runner.run() and runner.close() for each. Periodically evaluate policies during training to record test win rate at each iteration for every algorithm-scenario pair. Aggregate the win-rate logs and compare win-rate-versus-iteration curves: instability manifests as oscillating or non-monotonic test win rate, which the paper attributes to exponentially growing variance in the updated advantage function under importance sampling with independent actors. The rationale for the fix is that the clipping objective bounds advantage fluctuation, keeping the clipped advantage variance at most epsilon-squared for any state regardless of agent count, yielding a monotonic bound with sub-linear convergence to epsilon-Nash equilibria. Judge success by stable, monotonic convergence of the clipped variant and qualitatively matching the paper's Figure 3 trends.

## Related tools

- **HAPPO** (Suspect baseline algorithm whose unstable convergence in 27m_vs_30m is diagnosed via win-rate curves) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HAPPO** (Clipped sequential-update algorithm used to test whether clipping the advantage objective stabilizes training) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HATRPO** (Clipped trust region variant evaluated for stable convergence against HATRPO) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HATRPO** (Trust region baseline; less susceptible to high variance than HAPPO, used as a stability comparison) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **SMAC** (Benchmark environment (27m_vs_30m, 3s5z_vs_3s6z, corridor) on which convergence stability is measured) — https://github.com/oxwhirl/smac

## Examples

```
python -m examples.train --load_config run_configs/smac/27m_vs_30m/clip_happo.yaml
```

## Evaluation signals

- Reproduced win-rate curves qualitatively match Figure 3 trends: clipping-based methods outperform baselines in most scenarios (expert_review/llm_judge)
- HAPPO exhibits unstable (oscillating/non-monotonic) convergence in 27m_vs_30m, while clip-HAPPO converges stably
- Advantage variance of the clipped estimate stays bounded by epsilon-squared across states and does not grow with the number of agents
- All five algorithms (HAPPO, HATRPO, MAPPO, clip-HAPPO, clip-HATRPO) are trained on all three SMAC scenarios with per-iteration test win rates logged
- Training runs completed via the registered runner (runner.run() followed by runner.close()) without environment or dependency errors (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0)

## Limitations

- The derived exponential variance growth is only an upper bound (pessimistic); in practice growth depends on environment types and hyperparameters and does not always follow the worst case
- The clipping objective is not the only way to reduce variance — training and trust region setups also matter, which is why HATRPO is less susceptible to high variance than HAPPO
- Fixing the clipping threshold to 1 for all agents may be pessimistic in environments with less severe variance issues, and the clipped objective could dampen the exploration signal within the joint advantage
- SMAC results are sensitive to the StarCraft II version; performance is not always comparable between SC2 versions (e.g., SC2.4.6.2.69232 vs SC2.4.10)

## Evidence

- [methods] Variance bound: [evidence span withheld — anchor status: not_found]
