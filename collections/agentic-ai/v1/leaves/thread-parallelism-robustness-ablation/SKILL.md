---
name: thread-parallelism-robustness-ablation
description: Use when validating that a variance-reduction MARL method (clip-HAPPO
  vs HAPPO) retains its performance advantage and stable convergence when the number
  of parallel environments/threads is halved and quartered.
when_to_use_negative:
- When the goal is a full head-to-head benchmark against all baselines (HATRPO, MAPPO)
  across SMAC, MPE, and MAMuJoCo at default thread settings rather than the reduced-parallelism
  ablation
- When the environment is dexhands with separate evaluation enabled, since isaac gym
  does not support multiple instances
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: HAPPO
  role: Sequential-update trust region baseline trained under reduced thread budgets
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HAPPO
  role: Proposed clipped variant whose advantage-variance and performance robustness
    is being ablated
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: MAMuJoCo
  role: Multi-agent MuJoCo benchmark environment providing the 6 scenarios used in
    the ablation
  repo: https://github.com/deepmind/mujoco
- name: pip
  role: Installs the codebase ('pip install -e .') and pinned dependencies (mujoco-py>=2.1.2.14,
    gym==0.21.0, etc.)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:00cabdf88927e9651515e452b6f5e47727dd1b2aae30d46a5ba2003df653f77f
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
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# thread-parallelism-robustness-ablation

## Summary

An ablation skill that re-trains HAPPO and clip-HAPPO on MAMuJoCo scenarios with the thread/parallel-environment budget reduced (to 1/2 and 1/4 of the default), logging episodic reward and advantage-estimate variance to test whether clip-HAPPO's low-variance advantage claim and its performance win over HAPPO hold under reduced parallelism (Fig. 7, Appendix C1-C2).

## When to use

Use when validating that a variance-reduction MARL method (clip-HAPPO vs HAPPO) retains its performance advantage and stable convergence when the number of parallel environments/threads is halved and quartered.

## When NOT to use

- When the goal is a full head-to-head benchmark against all baselines (HATRPO, MAPPO) across SMAC, MPE, and MAMuJoCo at default thread settings rather than the reduced-parallelism ablation
- When the environment is dexhands with separate evaluation enabled, since isaac gym does not support multiple instances

## Inputs

- run_configs config files for HAPPO and clip-HAPPO in 6 MAMuJoCo scenarios
- Installed MuJoCo/MAMuJoCo environment (mujoco210 or mujoco-2.2.1 under ~/.mujoco, LD_LIBRARY_PATH set)
- Thread budget settings: 10 parallel environments (1/2) and 5 parallel environments (1/4)

## Outputs

- Logged episodic reward curves per algorithm/scenario/thread combination
- Logged advantage-estimate variance curves across update iterations
- Reward and variance comparison panels replicating Figure 7 and Appendix Figures C1-C2
- Win fractions of clip-HAPPO over HAPPO per thread budget (expected 5/6 at 10 threads, 4/6 at 5 threads)

## How to apply

First install the cloned repository with 'pip install -e .' and set up MuJoCo by extracting the downloaded archive to ~/.mujoco, exporting LD_LIBRARY_PATH to its bin folder, and installing the MAMuJoCo dependencies (mujoco-py>=2.1.2.14, Jinja2>=3.0.3, glfw>=2.5.1, Cython>=0.29.28), then resolve pinned dependencies (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0). For each of the 6 MAMuJoCo scenarios, launch HAPPO and clip-HAPPO training via 'python -m examples.train --load_config <CONFIG PATH>' using the run_configs configs, once with threads reduced to 10 (1/2) and once to 5 (1/4). During and after training, log episodic reward and the advantage-estimate variance across update iterations for every algorithm/scenario/thread combination. Aggregate the logged curves into reward and variance comparison panels replicating Figure 7 and Appendix Figures C1-C2, showing HAPPO vs clip-HAPPO under each thread budget. Validate success by confirming clip-HAPPO achieves more stable, lower-variance convergence and wins in 5/6 environments at 10 threads and 4/6 environments at 5 threads.

## Related tools

- **HAPPO** (Sequential-update trust region baseline trained under reduced thread budgets) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HAPPO** (Proposed clipped variant whose advantage-variance and performance robustness is being ablated) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **MAMuJoCo** (Multi-agent MuJoCo benchmark environment providing the 6 scenarios used in the ablation) — https://github.com/deepmind/mujoco
- **pip** (Installs the codebase ('pip install -e .') and pinned dependencies (mujoco-py>=2.1.2.14, gym==0.21.0, etc.))

## Examples

```
python -m examples.train --load_config run_configs/mamujoco/half_cheetah/clip_happo_5threads.yaml
```

## Evaluation signals

- clip-HAPPO shows more stable convergence and lower advantage-estimate variance than HAPPO in the variance panels under both 10-thread and 5-thread settings
- Win fraction of clip-HAPPO over HAPPO matches 5/6 environments at 10 threads and 4/6 environments at 5 threads
- Reward and variance panels reproduce the layout of Figure 7 and Appendix Figures C1-C2 across all 6 MAMuJoCo scenarios
- Training runs complete without dependency errors after the pinned pip installs (gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0)
- Logged curves exist for every algorithm x scenario x thread-budget combination (2 algorithms x 6 scenarios x 2 thread budgets)

## Limitations

- The exponential variance growth analysis is only an upper bound and is pessimistic; in practice variance growth depends on environment types and hyperparameters and does not always follow the worst case
- The clipping objective is not the only way to reduce variance — training and trust region setups also matter, which is why HATRPO is less susceptible to high variance than HAPPO
- Fixing the clipping threshold to 1 for all agents may appear pessimistic in environments with less severe variance issues
- The clipped objective could potentially dampen the exploration signal within the joint advantage

## Evidence

pip install "Jinja2>=3.0.3"
pip install "glfw>=2.5.1"
pip install "Cython>=0.29.28""
