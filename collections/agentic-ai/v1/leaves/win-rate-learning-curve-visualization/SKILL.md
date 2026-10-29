---
name: win-rate-learning-curve-visualization
description: Use when you have training logs from SMAC scenarios (e.g., 27m_vs_30m,
  3s5z_vs_3s6z, corridor) run with sequential-update trust region algorithms and need
  to compare convergence stability via test win-rate curves.
when_to_use_negative:
- The analysis target is advantage-variance estimation or ablation of the clipping
  threshold rather than end-task win-rate convergence
- The runs come from MPE or MAMuJoCo benchmarks where the primary success metric is
  not the SMAC test win rate
- Only a single algorithm/environment pair exists, so cross-algorithm comparison curves
  cannot be formed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SMAC
  role: StarCraft Multi-Agent Challenge environment whose scenarios (27m_vs_30m, 3s5z_vs_3s6z,
    corridor) supply the test win rate metric plotted in the learning curves
  repo: https://github.com/oxwhirl/smac
- name: giangbang/Low-Variance-Trust-Region-MARL
  role: Codebase providing examples.train, run_configs files, and the registered runner
    that produce the per-iteration win-rate logs to be plotted
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Installs the codebase (`pip install -e .`) and pinned dependencies (gym==0.21.0,
    pyglet==1.5.0, importlib-metadata==4.13.0) needed before training runs generate
    logs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:39c715d860659b4cb00a64204c5d5b7fca135d6e93e52b4b752efc7d494e3c6b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/oxwhirl/smac
  tools:
  - HAPPO
  - HATRPO
  - MAPPO
  - SMAC
  - clip-HAPPO
  - clip-HATRPO
  - giangbang/Low-Variance-Trust-Region-MARL
  - pip
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# win-rate-learning-curve-visualization

## Summary

Periodically evaluates MARL policies during training and aggregates per-iteration test win rates into win-rate-versus-iteration learning curves for every algorithm-scenario pair, reproducing Figure 3-style comparisons of clip-HAPPO/clip-HATRPO against HAPPO/HATRPO/MAPPO on SMAC.

## When to use

Use when you have training logs from SMAC scenarios (e.g., 27m_vs_30m, 3s5z_vs_3s6z, corridor) run with sequential-update trust region algorithms and need to compare convergence stability via test win-rate curves.

## When NOT to use

- The analysis target is advantage-variance estimation or ablation of the clipping threshold rather than end-task win-rate convergence
- The runs come from MPE or MAMuJoCo benchmarks where the primary success metric is not the SMAC test win rate
- Only a single algorithm/environment pair exists, so cross-algorithm comparison curves cannot be formed

## Inputs

- Win-rate logs recorded per iteration from periodic policy evaluation during training (one log per algorithm-scenario pair)
- Training outputs from runner.run() for clip-HAPPO, clip-HATRPO, HAPPO, HATRPO, and MAPPO on SMAC scenarios 27m_vs_30m, 3s5z_vs_3s6z, and corridor
- run_configs config files passed via --load_config for each algorithm-scenario pair

## Outputs

- figure3_win_rate_curves.png — win-rate-versus-iteration curves for all five algorithms per SMAC scenario
- Aggregated win-rate log tables per algorithm-scenario pair

## How to apply

Train each algorithm (clip-HAPPO, clip-HATRPO, HAPPO, HATRPO, MAPPO) on the target SMAC scenarios by launching `python -m examples.train --load_config <CONFIG PATH>` with the corresponding run_configs entries, so each run invokes the registered runner (`runner.run()` followed by `runner.close()`). During training, periodically evaluate the policies to record the test win rate at each training iteration for every algorithm-scenario pair. Aggregate the win-rate logs across the five algorithms per scenario and plot win-rate-versus-iteration curves, saving the combined figure as figure3_win_rate_curves.png. Judge correctness qualitatively: the clipping-based methods should outperform the baselines in most scenarios, and HAPPO should exhibit unstable (high-variance) convergence in 27m_vs_30m, consistent with the paper's variance analysis.

## Related tools

- **SMAC** (StarCraft Multi-Agent Challenge environment whose scenarios (27m_vs_30m, 3s5z_vs_3s6z, corridor) supply the test win rate metric plotted in the learning curves) — https://github.com/oxwhirl/smac
- **giangbang/Low-Variance-Trust-Region-MARL** (Codebase providing examples.train, run_configs files, and the registered runner that produce the per-iteration win-rate logs to be plotted) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Installs the codebase (`pip install -e .`) and pinned dependencies (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0) needed before training runs generate logs)

## Examples

```
python -m examples.train --load_config run_configs/smac/27m_vs_30m/clip_happo.yaml
```

## Evaluation signals

- A win-rate value is recorded at every iteration for each of the 5 algorithms x 3 SMAC scenarios (no missing algorithm-scenario pairs in the aggregated logs)
- The saved figure is figure3_win_rate_curves.png with one win-rate-versus-iteration panel/curve set per scenario covering all five algorithms
- Qualitative match to the paper's Figure 3: clip-HAPPO and clip-HATRPO outperform HAPPO, HATRPO, and MAPPO in most environments
- HAPPO's curve in 27m_vs_30m shows unstable convergence (oscillation/dropouts), consistent with the reported exponentially high advantage variance of sequential updates
- Win rates fall within the expected [0, 1] test win-rate range and training curves reach the approximate final win-rate levels shown in the reported figure

## Limitations

- StarCraft II version matters: SMAC warns performance is not always comparable between SC2 versions (the SMAC paper used SC2.4.6.2.69232, not SC2.4.10), so reproduced win-rate curves may shift across SC2 builds
- The exponential variance growth motivating unstable HAPPO curves is only an upper bound and pessimistic; in practice growth depends on environment types and hyperparameters and does not always follow the worst case
- HATRPO is less susceptible to high variance than HAPPO because training and trust region setups also mitigate variance, so baseline instability may not appear in every scenario

## Evidence

- [readme] SC2 version caveat for comparability of win rates: [evidence span withheld — anchor status: not_found]
