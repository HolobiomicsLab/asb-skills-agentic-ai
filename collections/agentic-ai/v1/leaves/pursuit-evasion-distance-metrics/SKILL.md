---
name: pursuit-evasion-distance-metrics
description: Use when evaluating or reproducing agent performance on orbital pursuit-evasion
  benchmarks like KSPDG/Space Gym, where results are reported as /average closest-approach
  distances (m) and failure rates rather than RL rewards.
when_to_use_negative:
- You need a training environment for large-scale RL — KSPDG explicitly provides evaluation
  environments, NOT training environments, since KSP was not designed for massively
  parallel, faster-than-real-time, headless execution
- The task requires full trajectory or fuel/delta-v metrics (e.g. utils.estimate_capture_dv)
  — the reported tables only cover closest-approach distance and failure rate
- Results are reported as RL reward curves rather than the paper's three-column distance/failure-rate
  format; converting would require re-running KSPDG episodes
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: KSPDG
  role: Differential-game (pursuit-evasion) evaluation environment whose episodes
    produce the closest-approach distance and failure-rate metrics
  repo: https://github.com/mit-ll/spacegym-kspdg
- name: Weights & Biases
  role: Experiment tracking backend holding the detailed logs and metrics from which
    results tables are validated
- name: PhysicsRangeExtender
  role: KSP mod enabling control of physically distant spacecraft so pursuit-evasion
    distances remain meaningful across extended ranges
  repo: https://github.com/jrodrigv/PhysicsRangeExtender
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.asr.2025.06.034
    title: Large language models as autonomous spacecraft operators in kerbal space
      program
  source_content_hash: sha256:0d13efd571e52484a136fc3d564252944a2e6bacf599bbe211dbf713fef936e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mit-ll/spacegym-kspdg
  tools:
  - GPT-3.5
  - KSPDG
  - PhysicsRangeExtender
  - Weights & Biases
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pursuit-evasion-distance-metrics

## Summary

Extract and compare closest-approach performance metrics (Best Distance in meters, Average Distance in meters, and Failure Rate %) for pursuit-evasion agents in the KSPDG differential-game suite, as used to rank LLM fine-tuning variants against navball bot and Lambert-MPC/PPO baselines.

## When to use

Use when evaluating or reproducing agent performance on orbital pursuit-evasion benchmarks like KSPDG/Space Gym, where results are reported as best/average closest-approach distances (m) and failure rates rather than RL rewards.

## When NOT to use

- You need a training environment for large-scale RL — KSPDG explicitly provides evaluation environments, NOT training environments, since KSP was not designed for massively parallel, faster-than-real-time, headless execution
- The task requires full trajectory or fuel/delta-v metrics (e.g. utils.estimate_capture_dv) — the reported tables only cover closest-approach distance and failure rate
- Results are reported as RL reward curves rather than the paper's three-column distance/failure-rate format; converting would require re-running KSPDG episodes

## Inputs

- KSPDG/Space Gym evaluation results tables (per-method best/average closest-approach distance and failure rate)
- Weights & Biases experiment logs for fine-tuning runs
- KSPDG mission save files (ksp_files/saves/missions) used to run evaluation episodes

## Outputs

- CSV table with columns method, best_distance_m, avg_distance_m, failure_rate (e.g. gpt_finetuning_results.csv)
- Ranked comparison of LLM fine-tuning variants against navball bot and optimal-control baselines

## How to apply

Locate the results tables comparing methods (e.g. Baseline GPT, Simple fine-tuning, + Hyperparam tuning, + System prompt, + Two train gameplays; or LLaMA fine-tune 10/25/50 files, sliding window, navball bot, Lambert-MPC, PPO, iLQGames). For each row, extract three columns: Best Distance (m), Average Distance (m), and Failure Rate, preserving threshold notation ('>') where reported (e.g. PPO >1643/2346 m, Lambert-MPC >18.24/47.94 m). Assemble the rows into a CSV with columns method, best_distance_m, avg_distance_m, failure_rate using pandas, then validate each cell against the source table value exactly (e.g. Simple fine-tuning 263.55/265.89/0.0%, fine-tune 50 files 11.86/29.76 m at 0.00%). Lower distances indicate a better pursuer, so the final GPT-3.5 config's 132.09/159.78 beats the 263.55/265.89 baseline; note that these scores are evaluation-only metrics since KSPDG provides evaluation environments, NOT training environments for reinforcement learning.

## Related tools

- **KSPDG** (Differential-game (pursuit-evasion) evaluation environment whose episodes produce the closest-approach distance and failure-rate metrics) — https://github.com/mit-ll/spacegym-kspdg
- **Weights & Biases** (Experiment tracking backend holding the detailed logs and metrics from which results tables are validated)
- **PhysicsRangeExtender** (KSP mod enabling control of physically distant spacecraft so pursuit-evasion distances remain meaningful across extended ranges) — https://github.com/jrodrigv/PhysicsRangeExtender

## Examples

```
import pandas as pd; df = pd.DataFrame([{'method':'Simple fine-tuning','best_distance_m':263.55,'avg_distance_m':265.89,'failure_rate':'0.0%'},{'method':'+ Two train gameplays','best_distance_m':132.09,'avg_distance_m':159.78,'failure_rate':'0.2%'}]); df.to_csv('gpt_finetuning_results.csv', index=False)
```

## Evaluation signals

- Every CSV cell matches the reported table value exactly (e.g. best distances 263.55, 188.90, 197.41, 132.09 for the four GPT-3.5 fine-tuning stages)
- All 9 LLaMA-comparison rows are present: 4 fine-tuned variants, baseline LLaMA, navball bot, Naive, PPO, iLQGames, Lambert-MPC
- Threshold notation ('>') is preserved for PPO (>1643/2346 m), iLQGames (>53.31/60.99 m), and Lambert-MPC (>18.24/47.94 m)
- Failure rates fall within plausible ranges (0.0%–0.2% for fine-tuned agents; 9.09% for baseline LLaMA)
- The lowest best-distance row corresponds to the method the paper identifies as best (fine-tune 50 files at 11.86 m; GPT-3.5 two gameplays at 132.09 m)

## Limitations

- KSPDG uses a non-blocking step function — game time does not pause while agents decide, penalizing slow LLM inference, so distances partly reflect decision latency
- Fine-tuning hyperparameters (learning rate, epochs, LoRA rank) are not reported, limiting exact reproduction of the numbers
- Outcomes are highly hardware-dependent for local LLaMA training/inference, and hardware specs are not specified
- KSPDG scenarios are currently tested only against pre-scripted bot adversaries, not head-to-head AI competition

## Evidence

- [methods] Simple fine-tuning 263.55 265.89 0.0%: "Simple fine-tuning 263.55 265.89 0.0%"
- [methods] + Two train gameplays 132.09 159.78 0.2%: "+ Two train gameplays 132.09 159.78 0.2%"
- [methods] fine-tune 50 files 11.86 29.76 0.00%: "fine-tune 50 files 11.86 29.76 0.00%"
- [methods] Navball (bot) 34.34 36.43 -: "Navball (bot) 34.34 36.43 -"
- [readme] KSPDG provides evaluations environments, _NOT_ training environments for reinforcement learning algorithms: [evidence span withheld — anchor status: not_found]
- [readme] In KSPDG, game time does not pause while agents make decisions: [evidence span withheld — anchor status: not_found]
