---
name: log-scale-variance-growth-visualization
description: Use when you need to empirically demonstrate that the advantage-estimate
  variance in HAPPO's sequential update scheme grows exponentially with the agents'
  positions, i.e. reproducing Fig. 1 on MAMuJoCo Ant-v2-8x1 or Humanoid-v2-17x1.
when_to_use_negative:
- When the goal is to measure variance of the proposed clipped objective (clip-HAPPO/clip-HATRPO)
  rather than baseline HAPPO — the clipped advantage variance is bounded by ε² and
  will not show exponential growth, so a log-scale growth curve is not the target
  quantity.
- When the benchmark has fewer agents or a milder variance regime, since the article
  notes the exponential growth is only an upper bound and 'in practice growth depends
  on environment types and hyperparameters' — the ~1e6 validation signal may not appear.
- When you want the theoretical variance bound rather than its empirical measurement;
  this skill is the empirical/logged-visualization part of the analysis.
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: HAPPO
  role: Algorithm whose sequential per-agent updates produce the advantage estimates
    whose per-position variance is logged and plotted
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: MAMuJoCo
  role: Benchmark environment providing Ant-v2-8x1 and Humanoid-v2-17x1 multi-agent
    configurations for the 8- and 17-agent sequential updates
- name: MuJoCo / mujoco-py
  role: Physics engine dependency required by MAMuJoCo; mujoco-py>=2.1.2.14 is the
    pinned binding for the MAMuJoCo setup
  repo: https://github.com/openai/mujoco-py
- name: pip
  role: Installs the codebase (pip install -e .) and the pinned environment dependencies
    (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0, mujoco-py>=2.1.2.14)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:25b7b8eeb4849459fc16a2d541db9e2e59124b410bfd5c5fe1de4b503ac9d108
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - MAMuJoCo
  - MuJoCo / mujoco-py
  - pip
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# log-scale-variance-growth-visualization

## Summary

Reproduces the exponential growth of HAPPO's advantage-estimate variance with agent position in the sequential update order (Fig. 1 of the Low-Variance Trust Region MARL paper) by logging per-agent advantage variance during training on MAMuJoCo Ant-v2-8x1 and Humanoid-v2-17x1 and plotting it on a log scale.

## When to use

Use when you need to empirically demonstrate that the advantage-estimate variance in HAPPO's sequential update scheme grows exponentially with the agents' positions, i.e. reproducing Fig. 1 on MAMuJoCo Ant-v2-8x1 or Humanoid-v2-17x1.

## When NOT to use

- When the goal is to measure variance of the proposed clipped objective (clip-HAPPO/clip-HATRPO) rather than baseline HAPPO — the clipped advantage variance is bounded by ε² and will not show exponential growth, so a log-scale growth curve is not the target quantity.
- When the benchmark has fewer agents or a milder variance regime, since the article notes the exponential growth is only an upper bound and 'in practice growth depends on environment types and hyperparameters' — the ~1e6 validation signal may not appear.
- When you want the theoretical variance bound rather than its empirical measurement; this skill is the empirical/logged-visualization part of the analysis.

## Inputs

- HAPPO training run on MAMuJoCo Ant-v2-8x1 (8 agents in sequential update order)
- HAPPO training run on MAMuJoCo Humanoid-v2-17x1 (17 agents in sequential update order)
- Per-agent advantage estimates logged during the sequential update (one value per agent position m per training iteration)
- Config file path from the run_configs folder (--load_config)

## Outputs

- variance_curves.csv: table of maximum advantage-estimate variance per agent position (columns for Ant-v2-8x1 and Humanoid-v2-17x1)
- figure1_variance.png: log-scale plot of per-agent maximum variance versus agent position for both environments

## How to apply

Set up the Low-Variance-Trust-Region-MARL codebase (pip install -e .) with MuJoCo and MAMuJoCo dependencies (mujoco-py>=2.1.2.14, Jinja2>=3.0.3, glfw>=2.5.1, Cython>=0.29.28), then pin gym==0.21.0, pyglet==1.5.0 and importlib-metadata==4.13.0 to resolve remaining dependencies. Launch HAPPO training via `python -m examples.train --load_config` using the Ant-v2-8x1 (8 agents) and Humanoid-v2-17x1 (17 agents) configs from the run_configs folder, driving the runner through runner.run() and runner.close(). During the sequential per-agent updates, log the advantage estimate for each agent position m in the update order (agent 1 to agent 8 for Ant, agent 1 to agent 17 for Humanoid), compute the maximum variance of the advantage estimate per agent position across training iterations, and write the table to variance_curves.csv. Plot the per-agent maximum variance on a log-scale y-axis against agent position for both environments and save the result as figure1_variance.png.

## Related tools

- **HAPPO** (Algorithm whose sequential per-agent updates produce the advantage estimates whose per-position variance is logged and plotted) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **MAMuJoCo** (Benchmark environment providing Ant-v2-8x1 and Humanoid-v2-17x1 multi-agent configurations for the 8- and 17-agent sequential updates)
- **MuJoCo / mujoco-py** (Physics engine dependency required by MAMuJoCo; mujoco-py>=2.1.2.14 is the pinned binding for the MAMuJoCo setup) — https://github.com/openai/mujoco-py
- **pip** (Installs the codebase (pip install -e .) and the pinned environment dependencies (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0, mujoco-py>=2.1.2.14))

## Examples

```
python -m examples.train --load_config <CONFIG PATH>
```

## Evaluation signals

- The log-scale variance curve for Humanoid-v2-17x1 exhibits exponential growth with agent position, reaching approximately 1e6 by the later positions, consistent with the reported empirically high variance.
- The Ant-v2-8x1 curve increases monotonically with agent position 1..8 (variance grows with update-order index m).
- variance_curves.csv contains one maximum-variance entry per agent position (8 rows for Ant, 17 rows for Humanoid) computed across training iterations.
- figure1_variance.png uses a logarithmic y-axis; without the log scale the exponential range (up to 1e6) would visually collapse early positions.
- Training runs complete cleanly via runner.run() and runner.close() after loading the config with --load_config, confirming the logged advantage statistics came from an intact sequential-update HAPPO loop.

## Limitations

- The exponential growth derived in the work is only an upper bound of the variance ('a pessimistic point of view'); actual growth in practice depends on environment types and hyperparameters and does not always follow the worst case, so the ~1e6 magnitude is specific to Humanoid-v2-17x1.
- The measurement is tied to baseline HAPPO's unclipped sequential-update importance ratio; it does not measure the clipped objective's bounded (≤ ε²) variance.
- Environment setup is version-sensitive: mujoco-py is deprecated and does not support MuJoCo after 2.1.0, and remaining dependencies must be pinned exactly (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0) or installation fails.

## Evidence

- [abstract] the updated advantage function suffers from exponentially high variance problems, which likely results in unstable convergence: "the updated advantage function suffers from exponentially high variance problems, which likely results in unstable convergence"
- [methods] Users can reproduce our results by using `python -m examples.train --load_config <CONFIG PATH>`: "Users can reproduce our results by using `python -m examples.train --load_config <CONFIG PATH>`"
- [discussion] the exponential growth derived in this work is only the upper bound of the variance, meaning that it is a pessimistic point of view: "the exponential growth derived in this work is only the upper bound of the variance, meaning that it is a pessimistic point of view"
- [readme] pip install gym==0.21.0 / pyglet==1.5.0 / importlib-metadata==4.13.0 dependency pins: "run the following commands to solve dependencies.

shell
pip install gym==0.21.0
pip install pyglet==1.5.0
pip install importlib-metadata==4.13.0"
- [readme] MAMuJoCo dependency pins for the training environments: "pip install "mujoco-py>=2.1.2.14"
pip install "Jinja2>=3.0.3"
pip install "glfw>=2.5.1"
pip install "Cython>=0.29.28""
