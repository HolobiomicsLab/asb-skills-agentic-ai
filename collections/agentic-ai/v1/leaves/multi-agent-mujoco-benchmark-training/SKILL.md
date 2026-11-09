---
name: multi-agent-mujoco-benchmark-training
description: Use when you need to reproduce MAMuJoCo benchmark comparisons (e.g. Ant-v2-8x1,
  Humanoid-v2-17x1, Walker2d-v2-6x1) of trust-region MARL algorithms, or when logging
  per-agent advantage variance during sequential updates to study variance growth.
when_to_use_negative:
- The task targets single-agent MuJoCo/Gym environments or non-trust-region MARL algorithms
  not registered in RUNNER_REGISTRY
- The benchmark of interest is SMAC or MPE — those need separate environment dependencies
  (SMAC install instructions, pettingzoo==1.22.2 / supersuit==3.7.0) and this skill's
  MAMuJoCo setup does not cover them
- The goal is sample-based convergence rate analysis, which the article explicitly
  leaves as future work
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2269
tools:
- name: clip-HAPPO
  role: Proposed low-variance clipped trust-region algorithm trained on MAMuJoCo;
    its advantage uses min(ℜ_{1:m-1}, 1) * A^π(s,a) to keep variance ≤ ϵ^2
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HATRPO
  role: Proposed clipped trust-region baseline trained alongside clip-HAPPO for the
    benchmark comparison
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HAPPO
  role: Unclipped sequential-update baseline whose advantage variance is logged to
    show exponential growth with agent update position
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: HATRPO
  role: Trust-region baseline trained on MAMuJoCo; less susceptible to high variance
    than HAPPO due to its trust region setup
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: MAPPO
  role: Multi-agent PPO baseline trained on MAMuJoCo for the aggregate normalized-score
    comparison
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Installs the codebase (`pip install -e .`), MAMuJoCo dependencies, and the
    pinned gym==0.21.0 / pyglet==1.5.0 / importlib-metadata==4.13.0 set
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:cba77664ec348bac103c83b69fffe85a3b7f977482a2dc51e76cc063b4c6417d
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
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# multi-agent-mujoco-benchmark-training

## Summary

Trains trust-region MARL algorithms (clip-HAPPO, clip-HATRPO, HAPPO, HATRPO, MAPPO) on the nine MAMuJoCo heterogeneous-agent MuJoCo environments using the Low-Variance-Trust-Region-MARL codebase, with YAML configs, to benchmark performance and to log per-agent advantage variance during sequential updates.

## When to use

Use when you need to reproduce MAMuJoCo benchmark comparisons (e.g. Ant-v2-8x1, Humanoid-v2-17x1, Walker2d-v2-6x1) of trust-region MARL algorithms, or when logging per-agent advantage variance during sequential updates to study variance growth.

## When NOT to use

- The task targets single-agent MuJoCo/Gym environments or non-trust-region MARL algorithms not registered in RUNNER_REGISTRY
- The benchmark of interest is SMAC or MPE — those need separate environment dependencies (SMAC install instructions, pettingzoo==1.22.2 / supersuit==3.7.0) and this skill's MAMuJoCo setup does not cover them
- The goal is sample-based convergence rate analysis, which the article explicitly leaves as future work

## Inputs

- YAML experiment config files from the run_configs folder (per algorithm x per MAMuJoCo environment)
- Installed MAMuJoCo environments (Ant-v2-4x2, Ant-v2-8x1, HalfCheetah-v2-2x3, HalfCheetah-v2-6x1, Humanoid-v2-17x1, Walker2d-v2-3x2, Walker2d-v2-6x1, Manyagent Swimmer-v2-10x2, Manyagent Swimmer-v2-20x1)
- Random and expert reference scores per environment for return normalization

## Outputs

- Evaluation episode returns per algorithm-environment pair from training runs
- Normalized aggregate score comparison across the 9 MAMuJoCo tasks (e.g. figure2_mamujoco_normalized_scores.png)
- Per-agent mean and maximum advantage variance statistics over the update order, plot of variance vs agent index for HAPPO vs clip-HAPPO (Figures 5-6 style)

## How to apply

Install the Low-Variance-Trust-Region-MARL codebase with `pip install -e .`, set up MuJoCo (extract to ~/.mujoco, export LD_LIBRARY_PATH to the mujoco210 or mujoco-2.2.1 bin folder, apt install libosmesa6-dev libgl1-mesa-glx libglfw3, patchelf), then install MAMuJoCo dependencies (`mujoco-py>=2.1.2.14`, `Jinja2>=3.0.3`, `glfw>=2.5.1`, `Cython>=0.29.28`) and pin the dependency set with gym==0.21.0, pyglet==1.5.0, and importlib-metadata==4.13.0. Launch each experiment via `python -m examples.train --load_config <CONFIG PATH>` using the matching YAML files from the run_configs folder; the script loads the existing config when --load_config is set (otherwise reads the default yaml), builds the runner via RUNNER_REGISTRY[args["algo"]], and calls runner.run() followed by runner.close(). For the 9 MAMuJoCo environments (Ant-v2-4x2, Ant-v2-8x1, HalfCheetah-v2-2x3/6x1, Humanoid-v2-17x1, Walker2d-v2-3x2/6x1, Manyagent Swimmer-v2-10x2/20x1) train the five algorithms independently and collect evaluation episode returns; normalize them per environment by min-max normalization against random and expert reference scores before aggregating. For the variance ablation, during the sequential per-agent updates log the advantage estimate for agent m computed under the partially updated joint policy — min(ℜ_{1:m-1}, 1) * A^π(s,a) for clip-HAPPO versus the unclipped ratio product for HAPPO — and aggregate per-agent mean and maximum variance over the update order.

## Related tools

- **clip-HAPPO** (Proposed low-variance clipped trust-region algorithm trained on MAMuJoCo; its advantage uses min(ℜ_{1:m-1}, 1) * A^π(s,a) to keep variance ≤ ϵ^2) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HATRPO** (Proposed clipped trust-region baseline trained alongside clip-HAPPO for the benchmark comparison) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HAPPO** (Unclipped sequential-update baseline whose advantage variance is logged to show exponential growth with agent update position) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **HATRPO** (Trust-region baseline trained on MAMuJoCo; less susceptible to high variance than HAPPO due to its trust region setup) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **MAPPO** (Multi-agent PPO baseline trained on MAMuJoCo for the aggregate normalized-score comparison) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Installs the codebase (`pip install -e .`), MAMuJoCo dependencies, and the pinned gym==0.21.0 / pyglet==1.5.0 / importlib-metadata==4.13.0 set)

## Examples

```
python -m examples.train --load_config run_configs/mamujoco/Ant-v2-8x1/clip-happo.yaml
```

## Evaluation signals

- For the variance ablation, clip-HAPPO variance curves remain approximately flat near 1 (consistent with Var ≤ ϵ^2) while HAPPO variance reaches the ~1e1–1e2 range across Ant-v2-8x1, Half Cheetah-v2-6x1, Manyagent Swimmer-v2-10x2, and Walker2d-v2-6x1
- The aggregate normalized-score figure shows clip-HAPPO achieving the best performance across the 9 MAMuJoCo environments, matching Figure 2's reported claim
- Training completes through runner.run() and runner.close() without import/dependency errors, confirming the pinned gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0 and mujoco-py>=2.1.2.14 stack is correctly installed
- Per-agent variance statistics are logged for every position in the sequential update order (e.g. up to 17 agents for Humanoid-v2-17x1), not just the first agent
- Episode returns are min-max normalized against random and expert reference scores before cross-environment aggregation

## Limitations

- The derived exponential variance growth is only an upper bound (pessimistic); in practice growth depends on environment types and hyperparameters and does not always follow the worst case
- The clipped objective could dampen the exploration signal within the joint advantage, and fixing the clipping threshold to 1 for all agents may be pessimistic in environments with less severe variance issues
- It remains open whether the joint policy optimized by the Trust Region framework converges to the global optimum under independent actors; sample-based convergence rate analysis was not performed
- Training and trust region setups also affect variance — HATRPO is less susceptible to high variance than HAPPO even without clipping

## Evidence

pip install "Jinja2>=3.0.3"
pip install "glfw>=2.5.1"
pip install "Cython>=0.29.28""
- [methods] clipped advantage variance bounded by epsilon squared: [evidence span withheld — anchor status: not_found]
