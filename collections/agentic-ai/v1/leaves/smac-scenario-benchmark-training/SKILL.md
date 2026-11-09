---
name: smac-scenario-benchmark-training
description: Use when reproducing SMAC super-hard scenario comparisons of clipped
  sequential-update trust region methods against HAPPO/HATRPO/MAPPO baselines, or
  when win-rate-vs-iteration curves are needed for cooperative MARL benchmarking.
when_to_use_negative:
- The benchmark of interest is MPE or (MA)MuJoCo rather than SMAC scenarios — use
  the corresponding pettingzoo/supersuit (pettingzoo==1.22.2, supersuit==3.7.0) or
  MuJoCo install path instead
- Only a single algorithm needs a smoke test without baseline comparison — full five-algorithm
  training across three scenarios is costly and unnecessary
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: clip-HAPPO
  role: Proposed clipped sequential-update algorithm trained on the SMAC scenarios
    and compared against baselines
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HATRPO
  role: Proposed clipped trust-region algorithm trained on the SMAC scenarios and
    compared against baselines
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HAPPO
  role: Baseline heterogeneous-agent PPO algorithm; expected to exhibit unstable convergence
    in 27m_vs_30m
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HATRPO
  role: Baseline heterogeneous-agent TRPO algorithm trained on the same scenarios
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: MAPPO
  role: Baseline multi-agent PPO algorithm trained on the same scenarios
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Installs the codebase (pip install -e .) and pins gym==0.21.0, pyglet==1.5.0,
    importlib-metadata==4.13.0
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:afb9af43bb5ea37145cc8f807b6b66f124122eb638c2d7174164f35ab05c0cdc
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# smac-scenario-benchmark-training

## Summary

Train MARL trust-region algorithms (clip-HAPPO, clip-HATRPO, HAPPO, HATRPO, MAPPO) on SMAC StarCraft II scenarios such as 27m_vs_30m, 3s5z_vs_3s6z, and corridor, recording test win rate per iteration to reproduce benchmark convergence curves (Figure 3).

## When to use

Use when reproducing SMAC super-hard scenario comparisons of clipped sequential-update trust region methods against HAPPO/HATRPO/MAPPO baselines, or when win-rate-vs-iteration curves are needed for cooperative MARL benchmarking.

## When NOT to use

- The benchmark of interest is MPE or (MA)MuJoCo rather than SMAC scenarios — use the corresponding pettingzoo/supersuit (pettingzoo==1.22.2, supersuit==3.7.0) or MuJoCo install path instead
- Only a single algorithm needs a smoke test without baseline comparison — full five-algorithm training across three scenarios is costly and unnecessary

## Inputs

- run_configs config files specifying algorithm, hyperparameters, and SMAC scenario (27m_vs_30m, 3s5z_vs_3s6z, corridor)
- Installed SMAC environment with StarCraft II (SC2PATH set) and SMAC Maps
- Codebase at giangbang/Low-Variance-Trust-Region-MARL installed via pip install -e .

## Outputs

- Per-iteration test win rate logs for each algorithm-scenario pair
- figure3_win_rate_curves.png: win-rate-versus-iteration curves for clip-HAPPO, clip-HATRPO, HAPPO, HATRPO, and MAPPO per scenario

## How to apply

Install the Low-Variance-Trust-Region-MARL codebase with `pip install -e .`, install SMAC following the official oxwhirl/smac instructions (including downloading SC2 and SMAC Maps), then pin the remaining dependencies gym==0.21.0, pyglet==1.5.0, and importlib-metadata==4.13.0. Launch each algorithm-scenario pair via `python -m examples.train --load_config <CONFIG PATH>` with the provided run_configs entries, which instantiates RUNNER_REGISTRY[algo] and calls runner.run() followed by runner.close(). Evaluate the policies periodically during training to record the test win rate at each iteration. Aggregate win-rate logs and plot win-rate-versus-iteration curves for all five algorithms per scenario (27m_vs_30m, 3s5z_vs_3s6z, corridor) to reproduce Figure 3. Judge correctness by whether the clipped methods (clip-HAPPO, clip-HATRPO) outperform HAPPO, HATRPO, and MAPPO in most scenarios and whether HAPPO shows the characteristic unstable convergence in 27m_vs_30m.

## Related tools

- **clip-HAPPO** (Proposed clipped sequential-update algorithm trained on the SMAC scenarios and compared against baselines) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HATRPO** (Proposed clipped trust-region algorithm trained on the SMAC scenarios and compared against baselines) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HAPPO** (Baseline heterogeneous-agent PPO algorithm; expected to exhibit unstable convergence in 27m_vs_30m) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HATRPO** (Baseline heterogeneous-agent TRPO algorithm trained on the same scenarios) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **MAPPO** (Baseline multi-agent PPO algorithm trained on the same scenarios) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Installs the codebase (pip install -e .) and pins gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0)

## Examples

```
pip install -e . && pip install gym==0.21.0 pyglet==1.5.0 importlib-metadata==4.13.0 && python -m examples.train --load_config run_configs/smac/clip_happo_27m_vs_30m.yaml
```

## Evaluation signals

- Clipped methods (clip-HAPPO, clip-HATRPO) achieve higher final test win rate than HAPPO, HATRPO, and MAPPO in most of the three scenarios
- HAPPO curves show unstable convergence (variance spikes) in 27m_vs_30m, matching the paper's Figure 3 trend
- Training runs complete via runner.run() followed by runner.close() for each registered algorithm
- Win rate is logged at every training iteration for every algorithm-scenario pair, enabling per-iteration curve plotting
- Reproduced curves qualitatively match the paper's Figure 3 (verified via expert_review/llm_judge against the reported figure)

## Limitations

- The derived exponential advantage-variance growth is only an upper bound (pessimistic); practical variance depends on environment type and hyperparameters and does not always follow the worst case
- The clipping threshold fixed at 1 for all agents may be pessimistic in environments with less severe variance issues
- SMAC performance is not always comparable across StarCraft II versions; the SMAC paper's results use SC2.4.6.2.69232, not SC2.4.10
- The clipped objective could dampen the exploration signal within the joint advantage; varying the clipping threshold is a natural extension

## Evidence

- [other] Train clip-HAPPO and clip-HATRPO on the 27m_vs_30m, 3s5z_vs_3s6z, and corridor scenarios: [evidence span withheld — anchor status: not_found]
- [readme] SMAC SC2 version warning: [evidence span withheld — anchor status: not_found]
