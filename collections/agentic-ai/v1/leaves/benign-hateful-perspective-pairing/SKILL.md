---
name: benign-hateful-perspective-pairing
description: Use when building multi-CoT training/analysis data for harmful meme detection
  and you need, per meme tuple M = {I, T}, paired hateful and benign reasoning chains
  (e.g., 3×3 = 9 pairs) serialized for downstream alignment or classification.
when_to_use_negative:
- The downstream task is final hateful/non-hateful classification via the prototype-guided
  and context-guided decoders rather than generating paired multi-CoT explanation
  data
- The input already contains a populated multi_cot_chains.json with 3 hateful and
  3 benign chains per meme, so regeneration would duplicate existing reasoning data
- Inputs are unimodal (text-only or image-only) and lack the paired image-text meme
  tuples M = {I, T} the chains must be grounded in
tools:
- name: LLaVA-1.5-7B
  role: MLLM used to generate the n=3 hateful and m=3 benign CoT chains per meme for
    the Hateful Memes dataset under the n=3, m=3 setting
  repo: https://github.com/BGWH123/EVL-MCoT
- name: GPT-4
  role: MLLM used to generate the paired hateful/benign CoT chains for the MultiOFF
    dataset under the n=3, m=3 setting
  repo: https://github.com/BGWH123/EVL-MCoT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:6c0eff75fc745efe2cd3cca113b8f63adb3d9ad7df7e53b14c784896150a0493
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - GPT-4
  - LLaVA
  - LLaVA-1.5-7B
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# benign-hateful-perspective-pairing

## Summary

Generates n=3 hateful and m=3 benign chain-of-thought reasoning chains per meme with an MLLM and pairs them into n × m = 9 comparative combinations, so harmful meme interpretation is grounded in both supportive and opposing arguments rather than a single unreliable CoT chain.

## When to use

Use when building multi-CoT training/analysis data for harmful meme detection and you need, per meme tuple M = {I, T}, paired hateful and benign reasoning chains (e.g., 3×3 = 9 pairs) serialized for downstream alignment or classification.

## When NOT to use

- The downstream task is final hateful/non-hateful classification via the prototype-guided and context-guided decoders rather than generating paired multi-CoT explanation data
- The input already contains a populated multi_cot_chains.json with 3 hateful and 3 benign chains per meme, so regeneration would duplicate existing reasoning data
- Inputs are unimodal (text-only or image-only) and lack the paired image-text meme tuples M = {I, T} the chains must be grounded in

## Inputs

- Meme tuples M = {I, T} (visual image plus associated text) from the Hateful Memes dataset (10,000+ examples) and/or MultiOFF dataset (743 memes)
- Prompt template q* with harmfulness indicators h0 (benign) and h1 (hateful)
- MLLM for chain generation: LLaVA-1.5-7B (Hateful Memes) or GPT-4 (MultiOFF)

## Outputs

- Per-meme set of exactly 3 hateful reasoning chains and 3 benign reasoning chains grounded in the image-text relationship
- 9 comparative hateful × benign pairings per meme (n × m = 9)
- Structured JSON file multi_cot_chains.json containing all per-meme chains and their pairings

## How to apply

For each meme tuple M = {I, T} from Hateful Memes (10,000+ examples) or MultiOFF (743 memes), build a prompt q* incorporating a harmfulness indicator h0 (benign) and h1 (hateful) with the template 'You are a professional meme explainer. Please explain why this meme is hateful/benign, using relevant background and commonsense knowledge.' Run the MLLM — LLaVA-1.5-7B for Hateful Memes, GPT-4 for MultiOFF — with the setting n=3, m=3, grounding each chain in the image-text relationship. Pair every hateful chain with every benign chain for the same meme to form 9 comparative combinations. Serialize all per-meme chains and pairings into a structured JSON file (multi_cot_chains.json). The rationale is that single-chain CoT is 'susceptible to inconsistencies and errors', while multi-CoT with opposing perspectives enhances reasoning reliability and consistency.

## Related tools

- **LLaVA-1.5-7B** (MLLM used to generate the n=3 hateful and m=3 benign CoT chains per meme for the Hateful Memes dataset under the n=3, m=3 setting) — https://github.com/BGWH123/EVL-MCoT
- **GPT-4** (MLLM used to generate the paired hateful/benign CoT chains for the MultiOFF dataset under the n=3, m=3 setting) — https://github.com/BGWH123/EVL-MCoT

## Examples

```
pairs = [(h, b) for h in hateful_cots for b in benign_cots]  # n=3 hateful x m=3 benign -> 9 pairs per meme, serialized to multi_cot_chains.json
```

## Evaluation signals

- multi_cot_chains.json exists and parses as valid JSON
- Every meme in multi_cot_chains.json contains exactly 3 hateful and 3 benign reasoning chains
- The 3 hateful and 3 benign chains per meme form exactly n × m = 9 comparative pairings
- Each generated chain is grounded in the image-text relationship and includes relevant background/commonsense knowledge per the 'professional meme explainer' prompt template
- Downstream consistency improves versus single-chain CoT, since multi-CoT generation is intended to 'enhance the consistency of the reasoning process'

## Limitations

- Generated reasoning chains are long — 'the standard CLIP model struggles with inputs exceeding 20 tokens, while our CoT approach requires processing 150-word sequences' — which is why Long-CLIP's 248-token context is used downstream
- Single-chain CoT outputs are unreliable and heavily influenced by the MLLM's prior knowledge, which is the failure mode this pairing skill mitigates
- Future work explicitly targets shortening the reasoning chains while maintaining performance, implying current chains are longer than ideal

## Evidence

- [methods] The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM: "The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM"
- [intro] To improve reasoning reliability, multi-CoT are generated, which enhances the consistency of the reasoning process.: "To improve reasoning reliability, multi-CoT are generated, which enhances the consistency of the reasoning process."
- [readme] aims to enhance multi-modal reasoning by integrating visual information with structured, multi-step chain-of-thought prompts: "The **EVL-MCoT** project aims to enhance multi-modal reasoning by integrating visual information with structured, multi-step chain-of-thought prompts."
- [methods] the standard CLIP model struggles with inputs exceeding 20 tokens, while our CoT approach requires processing 150-word sequences.: "The standard CLIP model struggles with inputs exceeding 20 tokens, while our CoT approach requires processing 150-word sequences."
