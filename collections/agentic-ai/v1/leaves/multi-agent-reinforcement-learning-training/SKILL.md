---
name: multi-agent-reinforcement-learning-training
description: Use when a task requires reproducing MARL benchmark training runs (e.g.,
  Ant-v2-8x1, Humanoid-v2-17x1, MPE Spread/Reference/Speaker-Listener) for HAPPO/clip-HAPPO-family
  algorithms and logging per-iteration reward or advantage-variance statistics.
when_to_use_negative:
- The task targets single-agent RL or non-cooperative (competitive/mixed) MARL settings
  — the article's methods assume independent actors in fully cooperative tasks
- Only the SMAC StarCraft II benchmark is required and the MAMuJoCo/MuJoCo dependency
  stack is unnecessary — install SMAC directly instead of the full pinned dependency
  set
- The analysis goal is sample-based convergence-rate estimation, which the article
  explicitly leaves as unperformed future work
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: HAPPO
  role: Sequential-update trust region baseline whose training runs and advantage-variance
    growth are reproduced
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HAPPO
  role: Clipped variant whose bounded advantage variance (Var <= epsilon^2) is validated
    against HAPPO
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Installs the codebase ('pip install -e .') and pinned dependencies (gym==0.21.0,
    pyglet==1.5.0, importlib-metadata==4.13.0)
- name: pettingzoo
  role: Provides MPE environment dependencies for fully-cooperative MPE task training
- name: supersuit
  role: MPE environment wrapper dependency pinned at 3.7.0
- name: MuJoCo
  role: Physics engine for MAMuJoCo environments; downloaded, extracted to ~/.mujoco
    with LD_LIBRARY_PATH set
  repo: https://github.com/google-deepmind/mujoco
- name: SMAC
  role: StarCraft Multi-Agent Challenge benchmark installed per official instructions
  repo: https://github.com/oxwhirl/smac
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_007
  - task_008
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:81d590509107ea5946d3cdeafffc4342487f62fa538cc88c7a89688608555bc3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - HATRPO
  - MAMuJoCo
  - MAPPO
  - MuJoCo
  - SMAC
  - clip-HAPPO
  - clip-HATRPO
  - pettingzoo
  - pip
  - supersuit
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# multi-agent-reinforcement-learning-training

## Summary

Launches training runs of sequential-update trust region MARL algorithms (MAPPO, HATRPO, HAPPO, clip-HATRPO, clip-HAPPO) on cooperative benchmarks (MPE, SMAC, MuJoCo/MAMuJoCo) using the Low-Variance-Trust-Region-MARL codebase, to reproduce reward curves and advantage-variance measurements.

## When to use

Use when a task requires reproducing MARL benchmark training runs (e.g., Ant-v2-8x1, Humanoid-v2-17x1, MPE Spread/Reference/Speaker-Listener) for HAPPO/clip-HAPPO-family algorithms and logging per-iteration reward or advantage-variance statistics.

## When NOT to use

- The task targets single-agent RL or non-cooperative (competitive/mixed) MARL settings — the article's methods assume independent actors in fully cooperative tasks
- Only the SMAC StarCraft II benchmark is required and the MAMuJoCo/MuJoCo dependency stack is unnecessary — install SMAC directly instead of the full pinned dependency set
- The analysis goal is sample-based convergence-rate estimation, which the article explicitly leaves as unperformed future work

## Inputs

- Experiment configuration YAML from the run_configs folder (algorithm, environment, thread budget)
- Algorithm choice among MAPPO, HATRPO, HAPPO, clip-HATRPO, clip-HAPPO
- Environment specification (MPE: simple_spread_v2, simple_reference_v2, simple_speaker_listener_v3; MAMuJoCo: Ant-v2-8x1, Humanoid-v2-17x1, Walker2d-v2-6x1; SMAC)
- Number of parallel environment threads (e.g., reduced to 10 or 5 for ablations)

## Outputs

- Episode-reward-versus-iteration training curves aggregated across seeds per algorithm-task pair
- Per-agent-position advantage-estimate variance logs (e.g., variance_curves.csv, maximum variance per agent position across iterations)
- Figure files such as figure1_variance.png (log-scale variance vs agent position) and mujoco_reward_variance.png (reward and variance panels)

## How to apply

Install the cloned repository with 'pip install -e .' and set up the target environment: MPE via pettingzoo==1.22.2 and supersuit==3.7.0, MuJoCo by extracting the binary to ~/.mujoco, exporting LD_LIBRARY_PATH and installing MAMuJoCo dependencies (mujoco-py>=2.1.2.14, Jinja2>=3.0.3, glfw>=2.5.1, Cython>=0.29.28), then resolve pinned dependencies with gym==0.21.0, pyglet==1.5.0, and importlib-metadata==4.13.0. Launch each run via 'python -m examples.train --load_config <CONFIG PATH>' with a config file from the run_configs folder; the runner is constructed through RUNNER_REGISTRY[args['algo']](args, algo_args, env_args), executed with runner.run() until the training horizon completes, and flushed with runner.close(). During sequential per-agent updates, log the episodic reward and, for variance studies, the advantage estimate per agent position m in the update order (e.g., agent 1 to 8 for Ant-v2-8x1, 1 to 17 for Humanoid-v2-17x1). Decide between the unclipped HAPPO baseline and the clipped variant, whose surrogate replaces the importance ratio with min(R_1:m-1, 1) and adds the reverse-order KL penalty term in the discounted marginal state distribution. Aggregate logged curves across seeds and plot reward-versus-iteration and variance-versus-agent-index panels; expect clipped methods to stay bounded (Var <= epsilon^2) while HAPPO's advantage variance grows exponentially with agent position, reaching up to 1e6 on Humanoid-v2-17x1.

## Related tools

- **HAPPO** (Sequential-update trust region baseline whose training runs and advantage-variance growth are reproduced) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HAPPO** (Clipped variant whose bounded advantage variance (Var <= epsilon^2) is validated against HAPPO) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Installs the codebase ('pip install -e .') and pinned dependencies (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0))
- **pettingzoo** (Provides MPE environment dependencies for fully-cooperative MPE task training)
- **supersuit** (MPE environment wrapper dependency pinned at 3.7.0)
- **MuJoCo** (Physics engine for MAMuJoCo environments; downloaded, extracted to ~/.mujoco with LD_LIBRARY_PATH set) — https://github.com/google-deepmind/mujoco
- **SMAC** (StarCraft Multi-Agent Challenge benchmark installed per official instructions) — https://github.com/oxwhirl/smac

## Examples

```
python -m examples.train --load_config run_configs/mujoco/happo_humanoid_17x1.yaml
```

## Evaluation signals

- The log-scale advantage-variance curve for Humanoid-v2-17x1 under HAPPO grows exponentially with agent position and reaches approximately 1e6
- clip-HAPPO's advantage-variance estimate stays bounded (Var <= epsilon^2) and does not grow as the number of agents increases
- Reward-versus-iteration curves are produced for all five algorithms per MPE task with the small performance spread reported for these simple tasks and stable convergence of clipped methods
- Thread-reduction ablation confirms clip-HAPPO's reported win fractions: 5/6 environments at 10 threads and 4/6 at 5 threads
- The runner lifecycle completes as runner.run() followed by runner.close(), flushing episode-reward logs

## Limitations

- The exponential variance growth analysis is a pessimistic upper bound; in practice growth depends on environment types and hyperparameters and does not always follow the worst case
- Fixing the clipping threshold to 1 for all agents may appear pessimistic in environments with less severe variance issues, and the clipped objective could dampen the exploration signal within the joint advantage
- Sample-based convergence rate analysis was not performed and is left as possible future work; whether the joint policy converges to the global optimum under independent actors remains open

## Evidence

- [methods] Clipped variance bound: [evidence span withheld — anchor status: not_found]
