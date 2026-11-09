---
name: latex-math-to-markdown-transcription
description: Use when a source (textbook chapter, code like masked_mean/masked_sum
  utilities, or paper) defines optimization losses in prose or Python and you must
  produce a single markdown cheatsheet with LaTeX equations per algorithm.
when_to_use_negative:
- The source is already structured markdown/LaTeX with rendered equations — re-transcription
  adds no value
- You need numerical verification of the equations (e.g., reproducing training curves)
  rather than faithful symbolic transcription — substring validation cannot certify
  mathematical correctness
- The task targets a different algorithm family (e.g., reward-model or rejection-sampling
  stages) whose losses are not part of this RL loss-function cheatsheet
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Python
  role: Script that authors rl_cheatsheet.md and runs the substring-based validation
    checks over the written file
  repo: https://github.com/natolambert/rlhf-book
- name: PyTorch
  role: Source of the reference training-loop code (masked_mean, per_token_loss) whose
    notation the LaTeX transcription must match
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:0d8d04023a3de51cd4ca5d5e88e6a4f34196ff5e946ac45bd1574cf32565b4c4
  source_papers:
  - doi: 10.48550/arxiv.2504.12501
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/natolambert/rlhf-book
  tools:
  - PyTorch
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# latex-math-to-markdown-transcription

## Summary

Transcribes mathematical loss functions (here the RLHF Book's RL optimization losses: REINFORCE, RLOO, PPO, GRPO, GSPO, CISPO, DPO) from prose/code sources into LaTeX inside a one-page markdown reference (rl_cheatsheet.md), preserving each algorithm's advantage estimator, clipping style, and importance-sampling granularity.

## When to use

Use when a source (textbook chapter, code like masked_mean/masked_sum utilities, or paper) defines optimization losses in prose or Python and you must produce a single markdown cheatsheet with LaTeX equations per algorithm.

## When NOT to use

- The source is already structured markdown/LaTeX with rendered equations — re-transcription adds no value
- You need numerical verification of the equations (e.g., reproducing training curves) rather than faithful symbolic transcription — substring validation cannot certify mathematical correctness
- The task targets a different algorithm family (e.g., reward-model or rejection-sampling stages) whose losses are not part of this RL loss-function cheatsheet

## Inputs

- Prose/tables from the RLHF Book Chapter 6 policy-gradient methods describing IS granularity, clipping style, and advantage estimator per algorithm
- Python training-loop code snippets (e.g., masked_mean/masked_sum utilities, per_token_loss construction) from the book's methods sections
- Target algorithm list (REINFORCE, RLOO, PPO, GRPO, GSPO, CISPO, and optionally DPO)

## Outputs

- rl_cheatsheet.md — a one-page markdown file: comparison table followed by per-algorithm LaTeX loss equations

## How to apply

Read the source chapter (e.g., the RLHF Book's Chapter 6 policy-gradient material) and extract a comparison table over IS granularity, clipping style, and advantage estimator for REINFORCE, RLOO, PPO, GRPO, GSPO, and CISPO. Transcribe each core loss L(theta) into LaTeX, e.g. REINFORCE as -(1/T) * sum_t log pi_theta(a_t|s_t)(G_t - b(s_t)), RLOO with the leave-one-out baseline R_i - (1/(K-1)) * sum_{j != i} R_j, and CISPO as -sum sg(rho_hat) A log pi with rho_hat = clip(rho, 1-eps, 1+eps). For PPO write the pessimistic surrogate min(rho_t A_t, clip(rho_t, 1-eps, 1+eps) A_t) with rho_t = pi_theta/pi_theta_old, plus the value loss 0.5*(returns - values)^2 combined via vf_coef into per_token_loss; for GRPO use the group-relative advantage A_i = (r_i - mean(r_1:G))/std(r_1:G) with the KL penalty added to the loss (L = L_pg + beta * D_KL); include GSPO's sequence-level ratio rho_i = (pi_theta/pi_theta_old)^(1/|a_i|) and DPO's Bradley-Terry loss -log sigma(beta[Delta log pi_theta - Delta log pi_ref]). Save as rl_cheatsheet.md and validate by substring checks (see evaluation_signals).

## Related tools

- **Python** (Script that authors rl_cheatsheet.md and runs the substring-based validation checks over the written file) — https://github.com/natolambert/rlhf-book
- **PyTorch** (Source of the reference training-loop code (masked_mean, per_token_loss) whose notation the LaTeX transcription must match)

## Examples

```
open('rl_cheatsheet.md','w').write(body); txt=open('rl_cheatsheet.md').read(); assert all(s in txt for s in ['clip(..., 1-eps, 1+eps)','std(r_1:G)','min(...)'])
```

## Evaluation signals

- rl_cheatsheet.md exists and is valid markdown
- File contains, for every listed algorithm, the advantage/baseline term and the clipped surrogate: substring 'clip(..., 1-eps, 1+eps)' present
- Group-relative normalization present as substring 'std(r_1:G)' (GRPO/GSPO/CISPO entries)
- PPO entry contains the pessimistic objective 'min(...)' with rho_t = pi_theta(a_t|s_t)/pi_theta_old(a_t|s_t)
- Comparison table covers all six algorithms with columns for IS granularity, clipping style, and advantage estimator

## Limitations

- Validation is substring-based (e.g., 'clip(..., 1-eps, 1+eps)'), so a typo elsewhere in an equation can pass while the check still succeeds
- Transcription fidelity depends on the source chapter's notation; the book's code (masked_mean, masked_sum) may use variable names that differ from the cheatsheet LaTeX
- The cheatsheet is tied to the RLHF Book's specific framing (e.g., KL penalty added to loss as L = L_pg + beta * D_KL vs. reward-side penalty r - beta*D_KL); other implementations may place penalties differently

## Evidence

- [other] Transcribe each core loss L(theta) into LaTeX: REINFORCE as -(1/T) * sum_t log pi_theta(a_t|s_t)(G_t - b(s_t)): [evidence span withheld — anchor status: not_found]
- [methods] def masked_mean(values: torch.Tensor, mask: torch.Tensor, axis: Optional[int] = None) -> torch.Tensor:: [evidence span withheld — anchor status: not_found]
