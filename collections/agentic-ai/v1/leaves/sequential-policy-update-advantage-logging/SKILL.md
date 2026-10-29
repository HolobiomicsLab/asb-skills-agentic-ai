---
name: sequential-policy-update-advantage-logging
description: Use when reproducing or extending the advantage-variance ablation of
  sequential-update trust region MARL methods (HAPPO vs clip-HAPPO) on MAMuJoCo tasks
  such as Ant-v2-8x1, Half Cheetah-v2-6x1, Manyagent Swimmer-v2-10x2, Walker2d-v2-6x1,
  and Humanoid-v2-17x1.
when_to_use_negative:
- Studying HATRPO or non-sequential (joint/simultaneous) policy updates, where the
  advantage does not change between sub-steps and the variance-vs-position phenomenon
  does not arise.
- Only evaluating return/sample-efficiency of the algorithms on the three MARL benchmarks
  without analyzing advantage variance — the standard training runs suffice and no
  per-agent logging hooks are needed.
- Environments with less severe variance issues where a fixed clipping threshold of
  1 for all agents may be pessimistic (the paper notes the threshold could be varied
  instead).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: HAPPO
  role: Sequential-update trust region algorithm whose unclipped ratio-product advantage
    estimates are logged to expose exponential variance growth
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: clip-HAPPO
  role: Clipped variant whose min(ℜ_{1:m-1},1)*A^π(s,a) advantage estimates are logged
    to verify variance stays flat near 1 (≤ ϵ²)
  repo: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- name: pip
  role: Installs the codebase and pinned dependencies (pip install -e ., mujoco-py>=2.1.2.14,
    gym==0.21.0)
- name: MuJoCo / MAMuJoCo
  role: Benchmark environments on which the per-agent advantage logging is performed
  repo: https://github.com/google-deepmind/mujoco
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s10458-025-09695-8
    title: Low variance trust region optimization with independent actors and sequential
      updates in cooperative multi-agent reinforcement learning
  source_content_hash: sha256:5bf862791ca67d00769f8fefdc2bc138fd509c92b9102cfbd8158497184857fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/giangbang/Low-Variance-Trust-Region-MARL
  tools:
  - HAPPO
  - MuJoCo / MAMuJoCo
  - clip-HAPPO
  - pip
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sequential-policy-update-advantage-logging

## Summary

Instrument the per-agent sequential updates of HAPPO and clip-HAPPO to log the advantage estimate computed under the partially updated joint policy, quantifying how variance grows with update position (exponentially for HAPPO, up to ~1e1-1e2, while clip-HAPPO stays flat near 1).

## When to use

Use when reproducing or extending the advantage-variance ablation of sequential-update trust region MARL methods (HAPPO vs clip-HAPPO) on MAMuJoCo tasks such as Ant-v2-8x1, Half Cheetah-v2-6x1, Manyagent Swimmer-v2-10x2, Walker2d-v2-6x1, and Humanoid-v2-17x1.

## When NOT to use

- Studying HATRPO or non-sequential (joint/simultaneous) policy updates, where the advantage does not change between sub-steps and the variance-vs-position phenomenon does not arise.
- Only evaluating return/sample-efficiency of the algorithms on the three MARL benchmarks without analyzing advantage variance — the standard training runs suffice and no per-agent logging hooks are needed.
- Environments with less severe variance issues where a fixed clipping threshold of 1 for all agents may be pessimistic (the paper notes the threshold could be varied instead).

## Inputs

- Low-Variance-Trust-Region-MARL codebase installed via pip install -e .
- Matching run_configs YAML files for HAPPO and clip-HAPPO on MAMuJoCo tasks (Ant-v2-8x1, Half Cheetah-v2-6x1, Manyagent Swimmer-v2-10x2, Walker2d-v2-6x1)
- MuJoCo/MAMuJoCo environment dependencies (mujoco-py>=2.1.2.14, gym==0.21.0)
- Per-agent advantage estimates logged during sequential policy updates

## Outputs

- Per-agent mean and maximum variance statistics of the advantage estimate indexed by update order
- Variance-vs-agent-index plots comparing HAPPO and clip-HAPPO (reproducing Figures 5-6)
- Validation table confirming clip-HAPPO variance ≈ 1 (bound ϵ²) and HAPPO variance in the 1e1-1e2 range

## How to apply

Set up the Low-Variance-Trust-Region-MARL codebase (pip install -e .) plus MuJoCo/MAMuJoCo dependencies (mujoco-py>=2.1.2.14, gym==0.21.0, pyglet==1.5.0, importlib-metadata==4.13.0), then launch HAPPO and clip-HAPPO with `python -m examples.train --load_config` using the matching YAML files from the run_configs folder. During each sequential update round, hook the training loop (runner.run()) so that for agent m you log the advantage estimate computed under the partially updated joint policy: min(ℜ_{1:m-1}(π̃,π),1)*A^π(s,a) for clip-HAPPO versus the unclipped importance-ratio product for HAPPO. Aggregate the logged values into per-agent mean and maximum variance statistics over the update order for each algorithm and each MAMuJoCo environment. Plot variance versus agent index/update order to reproduce Figures 5-6, then judge correctness against the paper's invariants: clip-HAPPO variance stays approximately flat near 1 (consistent with the Var <= ϵ² bound) while HAPPO variance reaches the 1e1-1e2 range, up to ~1e6 for Humanoid-v2-17x1 during training.

## Related tools

- **HAPPO** (Sequential-update trust region algorithm whose unclipped ratio-product advantage estimates are logged to expose exponential variance growth) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **clip-HAPPO** (Clipped variant whose min(ℜ_{1:m-1},1)*A^π(s,a) advantage estimates are logged to verify variance stays flat near 1 (≤ ϵ²)) — https://github.com/giangbang/Low-Variance-Trust-Region-MARL
- **pip** (Installs the codebase and pinned dependencies (pip install -e ., mujoco-py>=2.1.2.14, gym==0.21.0))
- **MuJoCo / MAMuJoCo** (Benchmark environments on which the per-agent advantage logging is performed) — https://github.com/google-deepmind/mujoco

## Examples

```
python -m examples.train --load_config <CONFIG PATH>
```

## Evaluation signals

- clip-HAPPO per-agent variance curves remain approximately flat near 1 across all agents and environments, consistent with Var <= ϵ².
- HAPPO per-agent variance grows with update position, reaching the ~1e1-1e2 range and up to ~1e6 for Humanoid-v2-17x1 during training.
- Logged advantage values match the paper's formulas: min(ℜ_{1:m-1}(π̃,π),1)*A^π(s,a) for clip-HAPPO vs the unclipped importance-ratio product for HAPPO.
- Plots reproduce the trend of Figures 5-6 across all four MAMuJoCo tasks (Ant-v2-8x1, Half Cheetah-v2-6x1, Manyagent Swimmer-v2-10x2, Walker2d-v2-6x1).
- Training runs are launched from the exact matching run_configs YAML files for each algorithm-environment pair.

## Limitations

- The exponential variance growth derived is only an upper bound — a pessimistic view; in practice growth depends on environment types and hyperparameters and does not always follow the worst case.
- The clipped objective is not the only way to reduce variance: training and trust region setups also matter (e.g. HATRPO is less susceptible than HAPPO).
- Fixing the clipping threshold to 1 for all agents may be pessimistic in environments with less severe variance issues, and the clip could dampen the exploration signal within the joint advantage.

## Evidence

- [methods] The clipped advantage variance is bounded by epsilon squared for any state and does not grow with the number of agents: [evidence span withheld — anchor status: not_found]
