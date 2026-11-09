---
name: marl-trust-region-algorithm-training
description: Use when you need to run training loops for trust region or clipped-surrogate
  MARL algorithms on environments like SMAC (e.g. 27m_vs_30m, 3s5z_vs_3s6z, corridor)
  and log per-iteration evaluation win rates for algorithm comparison.
when_to_use_negative:
- The target environment is not covered by the repo's supported envs (SMAC, MPE via
  pettingzoo==1.22.2/supersuit==3.7.0, MuJoCo/MAMuJoCo); e.g. dexhands requires special
  handling because isaac gym does not support multiple instances and eval is disabled
- You only need a theoretical variance or convergence analysis of the clipping objective
  rather than an actual training run
- A single-agent RL benchmark is the target — the runner registry and configs are
  built for heterogeneous-agent cooperative MARL
tools:
- name: clip-HAPPO
  role: Proposed clipped sequential-update algorithm to train and evaluate against
    baselines
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HATRPO
  role: Proposed clipped trust region variant to train alongside clip-HAPPO
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HAPPO
  role: Baseline heterogeneous-agent PPO algorithm; known unstable convergence in
    27m_vs_30m
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HATRPO
  role: Baseline heterogeneous-agent trust region algorithm
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: MAPPO
  role: Baseline direct application of PPO-style trust region optimization for MARL
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: SMAC
  role: StarCraft Multi-Agent Challenge benchmark environment providing the 27m_vs_30m,
    3s5z_vs_3s6z, and corridor scenarios
  repo: https://github.com/oxwhirl/smac
- name: pip
  role: Package installer used for `pip install -e .` and pinning gym==0.21.0, pyglet==1.5.0,
    importlib-metadata==4.13.0
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:80f4662d3ce1218952bb4506c09650f900b6ffee3596fe4cc13440330912956b
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# marl-trust-region-algorithm-training

## Summary

Train trust region MARL algorithms (clip-HAPPO, clip-HATRPO and baselines HAPPO, HATRPO, MAPPO) from the Low-Variance-Trust-Region-MARL codebase on cooperative multi-agent benchmarks, recording test win rate per iteration to compare convergence stability.

## When to use

Use when you need to run training loops for trust region or clipped-surrogate MARL algorithms on environments like SMAC (e.g. 27m_vs_30m, 3s5z_vs_3s6z, corridor) and log per-iteration evaluation win rates for algorithm comparison.

## When NOT to use

- The target environment is not covered by the repo's supported envs (SMAC, MPE via pettingzoo==1.22.2/supersuit==3.7.0, MuJoCo/MAMuJoCo); e.g. dexhands requires special handling because isaac gym does not support multiple instances and eval is disabled
- You only need a theoretical variance or convergence analysis of the clipping objective rather than an actual training run
- A single-agent RL benchmark is the target — the runner registry and configs are built for heterogeneous-agent cooperative MARL

## Inputs

- run_configs config file (yaml/json) for the chosen algorithm and environment scenario (e.g. clip-HAPPO on 27m_vs_30m)
- Installed codebase (giangbang/Low-Variance-Trust-Region-MARL) with SMAC maps available under $SC2PATH/Maps
- Algorithm choice: clip-HAPPO, clip-HATRPO, HAPPO, HATRPO, or MAPPO

## Outputs

- Per-iteration test win rate logs for each algorithm-scenario pair
- Win-rate-versus-iteration curves per scenario aggregating all algorithms (e.g. figure3_win_rate_curves.png)
- Training checkpoints/logs from runner.run()

## How to apply

Install the codebase with `pip install -e .`, set up the SMAC environment following the official oxwhirl/smac instructions (StarCraft II >= 3.16.1 plus SMAC Maps extracted to $SC2PATH/Maps), then pin the remaining dependencies gym==0.21.0, pyglet==1.5.0, and importlib-metadata==4.13.0. Launch each algorithm with `python -m examples.train --load_config <CONFIG PATH>`, pointing at the run_configs entries for clip-HAPPO, clip-HATRPO, HAPPO, HATRPO, or MAPPO; the load_config path selects an existing experiment config instead of reading defaults from the yaml config file. Internally the runner is instantiated via RUNNER_REGISTRY[args["algo"]](args, algo_args, env_args) and executed with runner.run() followed by runner.close(). Periodically evaluate the policies during training to record test win rate at each iteration for every algorithm-scenario pair, then aggregate and plot win-rate-versus-iteration curves per scenario (e.g. figure3_win_rate_curves.png).

## Related tools

- **clip-HAPPO** (Proposed clipped sequential-update algorithm to train and evaluate against baselines) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HATRPO** (Proposed clipped trust region variant to train alongside clip-HAPPO) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HAPPO** (Baseline heterogeneous-agent PPO algorithm; known unstable convergence in 27m_vs_30m) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HATRPO** (Baseline heterogeneous-agent trust region algorithm) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **MAPPO** (Baseline direct application of PPO-style trust region optimization for MARL) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **SMAC** (StarCraft Multi-Agent Challenge benchmark environment providing the 27m_vs_30m, 3s5z_vs_3s6z, and corridor scenarios) — https://github.com/oxwhirl/smac
- **pip** (Package installer used for `pip install -e .` and pinning gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0)

## Examples

```
python -m examples.train --load_config run_configs/clip-HAPPO/27m_vs_30m.yaml
```

## Evaluation signals

- Reproduced win-rate curves qualitatively match the paper's Figure 3 trends: clipping-based methods outperform HAPPO/HATRPO/MAPPO in most scenarios
- HAPPO exhibits unstable convergence in the 27m_vs_30m scenario, as reported in the paper
- Training runs complete cleanly through runner.run() and runner.close() without registry/config errors for every algorithm-scenario pair
- Test win rate is recorded at each iteration for all five algorithms so per-scenario curves can be aggregated and plotted together
- Environment setup passes the SMAC sanity check (e.g. python -m smac.examples.random_agents runs after installing maps)

## Limitations

- The exponential advantage-variance growth derived in the paper is only an upper bound and is a pessimistic view; in practice variance depends on environment type and hyperparameters
- The clipping threshold is fixed (e.g. to 1 for all agents) which may be pessimistic in environments with less severe variance issues; exploration signal within the joint advantage could be dampened
- StarCraft II version matters: SMAC performance is not always comparable between SC2 versions (paper results use SC2.4.6.2.69232, not SC2.4.10)
- isaac gym does not support multiple instances, so separate evaluation is disabled for the dexhands environment
- Sample-based convergence rate analysis of the clipped method was not performed and remains future work

## Evidence
