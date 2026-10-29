---
name: policy-gradient-objective-reconstruction
description: Use when the task requires comparing or reconstructing policy-gradient
  / RLHF loss functions (REINFORCE, RLOO, PPO, GRPO, GSPO, CISPO, DPO) and their advantage
  estimators from source text into a written reference.
when_to_use_negative:
- The goal is to run RL training rather than document its objectives — use an implementation
  framework (e.g. TRL or VeRL) instead of authoring a cheatsheet
- The question concerns RLHF pipeline stages outside policy-gradient losses (reward
  model training, rejection sampling, on-policy distillation, direct alignment beyond
  DPO)
- You need runnable training code with hyperparameters; the book does not release
  full RL training configs (e.g. no training details for the instruction-tuning stage
  of rejection sampling)
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Authoring the markdown cheatsheet file and running substring validation checks
    on its contents
- name: PyTorch
  role: Source of the reference per-token loss implementation style (masked_mean over
    token masks) the cheatsheet equations mirror
- name: TRL
  role: Framework context for how these losses are used in practice; TIS learner–sampler
    correction is adopted in TRL
  repo: https://github.com/huggingface/trl
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:86415af608e438cce06cfe4430170c33d0b8566a1246bd51233b00aeebafb5fc
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/huggingface/trl
  tools:
  - PyTorch
  - Python
  - TRL
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# policy-gradient-objective-reconstruction

## Summary

Reconstructs the one-page RL loss-function cheatsheet (rlhfbook.com/rl-cheatsheet) from the RLHF book's Chapter 6 policy-gradient material, transcribing the core objectives L(theta) for REINFORCE, RLOO, PPO, GRPO, GSPO, and CISPO into LaTeX with their IS granularity, clipping style, and advantage estimators.

## When to use

Use when the task requires comparing or reconstructing policy-gradient / RLHF loss functions (REINFORCE, RLOO, PPO, GRPO, GSPO, CISPO, DPO) and their advantage estimators from source text into a written reference.

## When NOT to use

- The goal is to run RL training rather than document its objectives — use an implementation framework (e.g. TRL or VeRL) instead of authoring a cheatsheet
- The question concerns RLHF pipeline stages outside policy-gradient losses (reward model training, rejection sampling, on-policy distillation, direct alignment beyond DPO)
- You need runnable training code with hyperparameters; the book does not release full RL training configs (e.g. no training details for the instruction-tuning stage of rejection sampling)

## Inputs

- Chapter 6 policy-gradient methods text from the RLHF book (natolambert/rlhf-book)
- Task source card with research question, finding, and workflow steps
- Table of contents / cheatsheet page at rlhfbook.com/rl-cheatsheet

## Outputs

- rl_cheatsheet.md — one-page markdown reference: comparison table (IS granularity, clipping style, advantage estimator) followed by per-algorithm LaTeX loss functions
- Transcribed core losses in LaTeX for REINFORCE, RLOO, PPO, GRPO, GSPO, CISPO (and optionally DPO)

## How to apply

Read the Chapter 6 policy-gradient methods text and first extract the comparison table of IS granularity (token vs sequence), clipping style (bilateral vs stop-grad), and advantage estimator. Then transcribe each core loss into LaTeX: REINFORCE as -(1/T) * sum_t log pi_theta(a_t|s_t)(G_t - b(s_t)); RLOO with the leave-one-out baseline R_i - (1/(K-1)) * sum_{j != i} R_j; PPO's pessimistic surrogate min(rho_t A_t, clip(rho_t, 1-eps, 1+eps) A_t) plus its value-function loss 0.5*(returns - values)^2 combined via vf_coef into per_token_loss; GRPO with group-relative advantage A_i = (r_i - mean(r_1:G))/std(r_1:G) and the KL penalty added directly to the loss (L = L_pg + beta * D_KL) rather than the reward-side penalty r - beta*D_KL; GSPO's sequence-level ratio rho_i = (pi_theta/pi_theta_old)^(1/|a_i|); and CISPO as -sum sg(rho_hat) A log pi with rho_hat = clip(rho, 1-eps, 1+eps). Optionally include DPO's Bradley-Terry sigmoid loss -log sigma(beta[Delta log pi_theta - Delta log pi_ref]). Author the markdown reference (rl_cheatsheet.md) with the comparison table followed by per-algorithm equations.

## Related tools

- **Python** (Authoring the markdown cheatsheet file and running substring validation checks on its contents)
- **PyTorch** (Source of the reference per-token loss implementation style (masked_mean over token masks) the cheatsheet equations mirror)
- **TRL** (Framework context for how these losses are used in practice; TIS learner–sampler correction is adopted in TRL) — https://github.com/huggingface/trl

## Examples

```
python -c "open('rl_cheatsheet.md','w').write(md); assert 'clip(..., 1-eps, 1+eps)' in md and 'std(r_1:G)' in md and 'min(' in md"
```

## Evaluation signals

- rl_cheatsheet.md exists and is valid markdown
- For every listed algorithm the advantage/baseline term and the clipped surrogate are present: substring 'clip(..., 1-eps, 1+eps)' found
- GRPO entry contains the group-relative normalization substring 'std(r_1:G)'
- PPO entry contains the pessimistic 'min(...)' objective min(rho_t A_t, clip(rho_t, 1-eps, 1+eps) A_t)
- RLOO entry shows the leave-one-out baseline R_i - (1/(K-1)) * sum_{j != i} R_j

## Limitations

- GRPO's advantage estimate assigns the same value to every token in the completion rather than discounted rewards from a learned value function, so it does not estimate per-token advantages
- The book describes GRPO's KL penalty as added directly to the loss (L = L_pg + beta * D_KL), which differs from the standard reward-side penalty r - beta*D_KL — mixing these conventions is an error mode
- No software version numbers are stated for the referenced frameworks (vLLM, Ray, PyTorch), so the cheatsheet's framing-code claims cannot be pinned to versions

## Evidence

- [full_text] GRPO simplifies value estimation by assigning the same value to every token in the episode: [evidence span withheld — anchor status: not_found]
- [methods] def masked_mean(values: torch.Tensor, mask: torch.Tensor, axis: Optional[int] = None): [evidence span withheld — anchor status: not_found]
