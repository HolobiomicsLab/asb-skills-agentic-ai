---
name: trust-region-sequential-update-execution
description: Use when a task requires running cooperative multi-agent RL training
  with trust region methods (HAPPO/HATRPO/clip-HAPPO/clip-HATRPO) on SMAC, MPE, or
  MAMuJoCo benchmarks, or when measuring per-agent advantage variance across sequential
  update order.
when_to_use_negative:
- When the task needs single-agent PPO/TRPO rather than sequential multi-agent updates
  with independent actors
- When a shared-critic method with centralized value functions (e.g. MAPPO-style joint
  critics) is required, since this skill models agents as independent actors
- When environments outside SMAC/MPE/MuJoCo-family (e.g. Isaac Gym dexhands with separate
  evaluation) are targeted — note the codebase disables separate evaluation for dexhands
  because isaac gym does not support multiple instances
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: HAPPO
  role: Sequential-update proximal policy optimization algorithm with independent
    actors being executed/reproduced
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HAPPO
  role: Clipped variant whose clipping objective bounds advantage fluctuation in sequential
    updates
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Installs the codebase ('pip install -e .') and the pinned dependency versions
    (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0)
- name: MAMuJoCo
  role: Heterogeneous-agent MuJoCo benchmark (Ant-v2-8x1, Humanoid-v2-17x1) used for
    variance growth experiments
  repo: https://github.com/openai/mujoco-py
- name: MuJoCo
  role: Physics engine dependency, installed to ~/.mujoco with LD_LIBRARY_PATH set
    before mujoco-py
  repo: https://github.com/deepmind/mujoco
- name: SMAC
  role: StarCraft multi-agent benchmark installed per oxwhirl/smac instructions
  repo: https://github.com/oxwhirl/smac
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:304e8d4bca63b39a14da7b9b1bd8655641373a38ed9764c72d5b6e3f004ccda2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - MAMuJoCo
  - MuJoCo
  - SMAC
  - clip-HAPPO
  - pip
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# trust-region-sequential-update-execution

## Summary

Executes trust region policy optimization training with sequential per-agent updates (HAPPO, HATRPO, and their clipped variants) in the giangbang/Low-Variance-Trust-Region-MARL codebase, using MAMuJoCo, SMAC, and MPE environments to reproduce benchmark results and measure advantage-estimate variance.

## When to use

Use when a task requires running cooperative multi-agent RL training with trust region methods (HAPPO/HATRPO/clip-HAPPO/clip-HATRPO) on SMAC, MPE, or MAMuJoCo benchmarks, or when measuring per-agent advantage variance across sequential update order.

## When NOT to use

- When the task needs single-agent PPO/TRPO rather than sequential multi-agent updates with independent actors
- When a shared-critic method with centralized value functions (e.g. MAPPO-style joint critics) is required, since this skill models agents as independent actors
- When environments outside SMAC/MPE/MuJoCo-family (e.g. Isaac Gym dexhands with separate evaluation) are targeted — note the codebase disables separate evaluation for dexhands because isaac gym does not support multiple instances

## Inputs

- Algorithm config file (yaml) from the run_configs folder specifying algo, environment, and hyperparameters
- Environment installation: SMAC (StarCraft II + SMAC_Maps in $SC2PATH/Maps), MPE (pettingzoo==1.22.2, supersuit==3.7.0), or MAMuJoCo (MuJoCo 2.1+ in ~/.mujoco)
- Choice of algorithm: HAPPO, HATRPO, clip-HAPPO, or clip-HATRPO via RUNNER_REGISTRY

## Outputs

- Trained multi-agent policies per benchmark environment (SMAC, MPE, MAMuJoCo Ant-v2-8x1, Humanoid-v2-17x1)
- Per-iteration training logs including advantage estimates per agent position m in the sequential update order
- Variance tables (e.g. variance_curves.csv) and log-scale plots of maximum advantage-estimate variance versus agent position

## How to apply

Install the codebase with 'pip install -e .', then install the target environment's dependencies: SMAC per oxwhirl instructions, MPE via pettingzoo==1.22.2 and supersuit==3.7.0, or MuJoCo by extracting mujoco to ~/.mujoco, exporting LD_LIBRARY_PATH, and installing mujoco-py>=2.1.2.14, Jinja2>=3.0.3, glfw>=2.5.1, Cython>=0.29.28 for MAMuJoCo. Resolve remaining dependency conflicts with pinned gym==0.21.0, pyglet==1.5.0, and importlib-metadata==4.13.0. Launch training with 'python -m examples.train --load_config <CONFIG PATH>' using a config file from the run_configs folder; the runner is instantiated via RUNNER_REGISTRY[args['algo']] and driven by runner.run() followed by runner.close(). If --load_config is set, an existing experiment config is loaded, otherwise defaults are read from the yaml config file. During training, the sequential update order processes each agent in turn, so instrument the per-agent update loop to log advantage estimates by agent position if variance-by-position curves are needed.

## Related tools

- **HAPPO** (Sequential-update proximal policy optimization algorithm with independent actors being executed/reproduced) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HAPPO** (Clipped variant whose clipping objective bounds advantage fluctuation in sequential updates) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Installs the codebase ('pip install -e .') and the pinned dependency versions (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0))
- **MAMuJoCo** (Heterogeneous-agent MuJoCo benchmark (Ant-v2-8x1, Humanoid-v2-17x1) used for variance growth experiments) — https://github.com/openai/mujoco-py
- **MuJoCo** (Physics engine dependency, installed to ~/.mujoco with LD_LIBRARY_PATH set before mujoco-py) — https://github.com/deepmind/mujoco
- **SMAC** (StarCraft multi-agent benchmark installed per oxwhirl/smac instructions) — https://github.com/oxwhirl/smac

## Examples

```
python -m examples.train --load_config run_configs/mamujoco/happo_ant_8x1.yaml
```

## Evaluation signals

- Training runs terminate cleanly after runner.run() returns and runner.close() is invoked without MuJoCo/pip dependency errors
- On Humanoid-v2-17x1, the logged maximum advantage-estimate variance grows exponentially with agent position, reaching approximately 1e6 at high agent positions (agent 17)
- The variance curve on a log scale is monotonic increasing in agent position m (1 to 8 for Ant-v2-8x1, 1 to 17 for Humanoid-v2-17x1)
- Config loading path: with --load_config set, hyperparameters match the existing experiment config file rather than yaml defaults
- Clipped variants (clip-HAPPO/clip-HATRPO) show advantage variance bounded near the clipping threshold (epsilon^2 per state) instead of growing with the number of agents

## Limitations

- The derived exponential variance growth is only an upper bound and is pessimistic; in practice growth depends on environment types and hyperparameters and does not always follow the worst case
- HATRPO is less susceptible to high variance than HAPPO because training and trust region setups also matter, not just the clipping mechanism
- The clipping objective could potentially dampen the exploration signal within the joint advantage, and fixing the clipping threshold to 1 for all agents may be pessimistic in environments with less severe variance issues
- mujoco-py is deprecated and does not support MuJoCo versions after 2.1.0; new users should use official DeepMind Python bindings

## Evidence

pip install "Jinja2>=3.0.3"
pip install "glfw>=2.5.1"
pip install "Cython>=0.29.28""
