---
name: trust-region-clipped-objective-implementation
description: Use when sequential-update trust region MARL (HAPPO/HATRPO) shows advantage-variance
  growth with agent index on benchmarks like MAMuJoCo Ant-v2-8x1 or Humanoid-v2-17x1,
  and a low-variance clipped alternative must be trained and compared.
when_to_use_negative:
- The environment is a single-agent setting or a non-sequential joint-update MARL
  method (e.g. MAPPO-style simultaneous updates), where the exponential advantage-variance
  growth with agent position does not arise.
- Variance growth is not the diagnosis — the article notes HATRPO is less susceptible
  to high variance than HAPPO because training and trust region setups also mitigate
  variance, so clipping may be unnecessary there.
- Fixed clipping threshold 1 for all agents would be pessimistic in environments with
  less severe variance issues and could dampen the exploration signal within the joint
  advantage.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: HAPPO
  role: Sequential-update baseline whose advantage variance grows exponentially with
    agent position in the update order
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HAPPO
  role: Clipped variant using min(ℜ_1:m-1, 1) in the advantage and β_k D_KL(π̃_m ||
    π_m) penalty, with variance bounded by ε^2
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: MuJoCo
  role: Physics engine dependency for the MAMuJoCo benchmark environments
  repo: https://github.com/deepmind/mujoco
- name: MAMuJoCo
  role: Multi-agent MuJoCo benchmark (Ant-v2-8x1, Humanoid-v2-17x1, Walker2d-v2-6x1)
    used for training and variance logging
- name: pip
  role: Installs the codebase ('pip install -e .') and pinned dependencies (mujoco-py>=2.1.2.14,
    gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:d6be4ab4880b7c92b7d4642a77e3a502c78014cc484a518236d94f91a425d512
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - MAMuJoCo
  - MuJoCo
  - clip-HAPPO
  - pip
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# trust-region-clipped-objective-implementation

## Summary

Implement and evaluate the clipped trust region objective (clip-HAPPO/clip-HATRPO) that replaces the importance ratio with min(ℜ_1:m-1, 1) in the advantage estimate and penalizes with a reverse-order KL term, bounding advantage variance by ε^2 during sequential multi-agent updates.

## When to use

Use when sequential-update trust region MARL (HAPPO/HATRPO) shows advantage-variance growth with agent index on benchmarks like MAMuJoCo Ant-v2-8x1 or Humanoid-v2-17x1, and a low-variance clipped alternative must be trained and compared.

## When NOT to use

- The environment is a single-agent setting or a non-sequential joint-update MARL method (e.g. MAPPO-style simultaneous updates), where the exponential advantage-variance growth with agent position does not arise.
- Variance growth is not the diagnosis — the article notes HATRPO is less susceptible to high variance than HAPPO because training and trust region setups also mitigate variance, so clipping may be unnecessary there.
- Fixed clipping threshold 1 for all agents would be pessimistic in environments with less severe variance issues and could dampen the exploration signal within the joint advantage.

## Inputs

- MAMuJoCo task configuration files from the run_configs folder (e.g. Ant-v2-8x1, Humanoid-v2-17x1, Walker2d-v2-6x1)
- Algorithm selection argument for HAPPO vs clip-HAPPO passed to RUNNER_REGISTRY
- Multiple training seeds for aggregating reward and advantage-variance statistics

## Outputs

- Logged episode reward versus training iteration curves for HAPPO and clip-HAPPO
- Average estimated advantage variance per agent position in the update sequence, per iteration
- Side-by-side figure (mujoco_reward_variance.png) of reward-versus-iteration and variance-versus-agent-index curves

## How to apply

Install the giangbang/Low-Variance-Trust-Region-MARL codebase with 'pip install -e .' and set up MuJoCo/MAMuJoCo dependencies (MuJoCo extracted to ~/.mujoco with LD_LIBRARY_PATH, then mujoco-py>=2.1.2.14, gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0). Load an experiment config from the run_configs folder via 'python -m examples.train --load_config <CONFIG PATH>', which routes into RUNNER_REGISTRY[args['algo']] and runs the training loop through runner.run() and runner.close(). Train the HAPPO baseline with the sequential update order fixed to 1..N alongside the clipped variant that uses the surrogate min(ℜ_1:m-1, 1) in the advantage and the reverse-order KL penalty β_k D_KL(π̃_m || π_m) in the discounted marginal state distribution, logging episode reward and the average estimated advantage variance per agent across iterations and seeds. Judge success by reproducing the qualitative pattern that HAPPO's advantage-variance estimate grows with the number of agents while the clipped objective's variance stays bounded (Var ≤ ε^2) and does not grow with agent count.

## Related tools

- **HAPPO** (Sequential-update baseline whose advantage variance grows exponentially with agent position in the update order) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HAPPO** (Clipped variant using min(ℜ_1:m-1, 1) in the advantage and β_k D_KL(π̃_m || π_m) penalty, with variance bounded by ε^2) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **MuJoCo** (Physics engine dependency for the MAMuJoCo benchmark environments) — https://github.com/deepmind/mujoco
- **MAMuJoCo** (Multi-agent MuJoCo benchmark (Ant-v2-8x1, Humanoid-v2-17x1, Walker2d-v2-6x1) used for training and variance logging)
- **pip** (Installs the codebase ('pip install -e .') and pinned dependencies (mujoco-py>=2.1.2.14, gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0))

## Examples

```
python -m examples.train --load_config run_configs/mamujoco/Humanoid-v2-17x1/clip_happo.yaml
```

## Evaluation signals

- HAPPO's average estimated advantage variance grows with the agents' position in the update sequence (up to ~1e6 on Humanoid-v2-17x1).
- The clipped objective's advantage variance stays bounded by ε^2 and does not grow as the number of agents increases.
- Reward-versus-iteration curves for both algorithms are aggregated across seeds and plotted side by side (mujoco_reward_variance.png) showing the qualitative pattern of Fig. 8.
- The training loop executes through runner.run() and runner.close() with the config loaded via --load_config from run_configs.
- Both algorithms run under identical settings (fixed update order 1..N, same environment and hyperparameters) so the comparison isolates the clipping mechanism.

## Limitations

- The derived exponential variance growth is only an upper bound and is pessimistic; in practice growth depends on environment types and hyperparameters and does not always follow the worst case.
- Fixing the clipping threshold to 1 for all agents may be pessimistic in environments with less severe variance issues and could potentially dampen the exploration signal within the joint advantage.
- Clipping is not the only way to reduce variance — training and trust region setups also matter, which is why HATRPO is less susceptible to high variance than HAPPO.

## Evidence

- [methods] Clipped advantage variance bounded by epsilon squared: [evidence span withheld — anchor status: not_found]
pip install "Jinja2>=3.0.3""
