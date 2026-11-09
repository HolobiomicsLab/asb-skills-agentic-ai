---
name: advantage-variance-estimation
description: Use when training or analyzing sequential-update trust region MARL algorithms
  (HAPPO, clip-HAPPO, HATRPO) on multi-agent benchmarks and you need to quantify whether
  the advantage estimate variance grows with the agents' position in the update order.
when_to_use_negative:
- The environment or algorithm does not use a sequential update scheme with independent
  actors (e.g. centralized-critic methods like MAPPO without ratio-product advantages),
  so the per-position variance analysis does not apply
- You only need final reward performance comparison rather than diagnosing advantage-estimate
  variance; the paper notes the exponential growth is an upper bound (pessimistic
  view) and HATRPO is less susceptible to high variance than HAPPO
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: HAPPO
  role: Baseline sequential-update algorithm whose unclipped importance-ratio-product
    advantage variance is measured per agent position
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HAPPO
  role: Clipped alternative using min(ℜ_{1:m-1}, 1) in the advantage, used as the
    low-variance comparison arm
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: MAMuJoCo
  role: Multi-agent MuJoCo benchmark environments on which the variance is estimated
- name: pip
  role: Installs the codebase and pinned dependencies (mujoco-py>=2.1.2.14, gym==0.21.0,
    pyglet==1.5.0, importlib-metadata==4.13.0)
provenance:
  source_task_ids:
  - task_006
  - task_007
  - task_008
  - task_001
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:df845b44e9184e1b3c600930395489588ac6507181513568ea4c7e568d4f4d87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - MAMuJoCo
  - clip-HAPPO
  - pip
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# advantage-variance-estimation

## Summary

Estimates and tracks the variance of the per-agent advantage estimate under sequential-update trust region methods (HAPPO) versus clipped variants (clip-HAPPO) in cooperative MARL, demonstrating exponential variance growth with agent update position for unclipped ratio products and a bounded variance (≤ ε²) for the clipped surrogate min(ℜ_{1:m-1}, 1).

## When to use

Use when training or analyzing sequential-update trust region MARL algorithms (HAPPO, clip-HAPPO, HATRPO) on multi-agent benchmarks and you need to quantify whether the advantage estimate variance grows with the agents' position in the update order.

## When NOT to use

- The environment or algorithm does not use a sequential update scheme with independent actors (e.g. centralized-critic methods like MAPPO without ratio-product advantages), so the per-position variance analysis does not apply
- You only need final reward performance comparison rather than diagnosing advantage-estimate variance; the paper notes the exponential growth is an upper bound (pessimistic view) and HATRPO is less susceptible to high variance than HAPPO

## Inputs

- HAPPO and clip-HAPPO training runs on MAMuJoCo scenarios (Ant-v2-8x1, Humanoid-v2-17x1, Half Cheetah-v2-6x1, Manyagent Swimmer-v2-10x2, Walker2d-v2-6x1)
- YAML experiment configs from the run_configs folder
- Per-iteration logged advantage estimates for each agent position m in the sequential update order

## Outputs

- Per-agent mean and maximum variance statistics over update order for each algorithm/environment
- Log-scale variance-versus-agent-position curves (e.g. figure1_variance.png, mujoco_reward_variance.png) reproducing Figs. 1, 5-6, 8
- Variance table (e.g. variance_curves.csv)

## How to apply

During the sequential per-agent updates in the Low-Variance-Trust-Region-MARL codebase, log the advantage estimate for each agent position m computed under the partially updated joint policy: the unclipped ratio product ℜ_{1:m-1} * A^π(s,a) for HAPPO versus min(ℜ_{1:m-1}, 1) * A^π(s,a) for clip-HAPPO. Launch training via 'python -m examples.train --load_config' with the matching YAML config from run_configs for environments such as Ant-v2-8x1, Humanoid-v2-17x1, Half Cheetah-v2-6x1, Manyagent Swimmer-v2-10x2, and Walker2d-v2-6x1. Aggregate logged values into per-agent mean and maximum variance statistics over the update order and training iterations. Plot variance versus agent index on a log scale for both algorithms; the theoretical clipped bound is Var ≤ ε² for any state, so clip-HAPPO curves should stay approximately flat near ~1 while HAPPO variance grows exponentially with agent position, reaching the ~1e1–1e2 range (up to ~1e6 on Humanoid-v2-17x1).

## Related tools

- **HAPPO** (Baseline sequential-update algorithm whose unclipped importance-ratio-product advantage variance is measured per agent position) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HAPPO** (Clipped alternative using min(ℜ_{1:m-1}, 1) in the advantage, used as the low-variance comparison arm) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **MAMuJoCo** (Multi-agent MuJoCo benchmark environments on which the variance is estimated)
- **pip** (Installs the codebase and pinned dependencies (mujoco-py>=2.1.2.14, gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0))

## Examples

```
python -m examples.train --load_config run_configs/mamujoco/humanoid_v2_17x1/clip_happo.yaml
```

## Evaluation signals

- Clip-HAPPO variance curves remain approximately flat near ~1, consistent with the bound Var ≤ ε² that does not grow with agent count
- HAPPO variance grows exponentially with agent update position, reaching the ~1e1–1e2 range across environments (up to ~1e6 on Humanoid-v2-17x1)
- Variance is dominated by later agents in the update order (agents near position N in Ant-v2-8x1 / Humanoid-v2-17x1)
- Log-scale per-agent maximum variance curves match the qualitative trends of the paper's Figures 1 and 5-6 across seeds

## Limitations

- The derived exponential growth is only an upper bound of the variance (a pessimistic point of view); in practice growth depends on environment types and hyperparameters and does not always follow the worst case
- The clipping objective is not the only way to reduce variance — training and trust region setups also matter, which is why HATRPO is less susceptible to high variance than HAPPO
- Fixing the clipping threshold to 1 for all agents may be pessimistic in environments with less severe variance issues, and the clipped objective could dampen the exploration signal within the joint advantage

## Evidence

- [abstract] exponential variance growth: "the updated advantage function suffers from exponentially high variance problems, which likely results in unstable convergence"
- [methods] clipped advantage variance bound: [evidence span withheld — anchor status: not_found]
- [discussion] variance bound is pessimistic upper bound: "the exponential growth derived in this work is only the upper bound of the variance, meaning that it is a pessimistic point of view"
- [methods] training entry point: "Users can reproduce our results by using `python -m examples.train --load_config <CONFIG PATH>`"
- [results] runner loop: "runner = RUNNER_REGISTRY[args["algo"]](args, algo_args, env_args)
    runner.run()
    runner.close()"
- [readme] MAMuJoCo dependency setup: "pip install "mujoco-py>=2.1.2.14"
pip install "Jinja2>=3.0.3""
