---
name: mlm-inference-pipeline-construction
description: 'Use when interpreting harmful memes with an MLLM requires multi-perspective
  reasoning: you have image-text meme tuples M = {I, T} and need paired supportive/opposing
  CoT chains rather than one unreliable reasoning chain.'
when_to_use_negative:
- When the downstream task needs fine-grained visual-textual feature fusion (the prototype-guided
  / context-guided decoder with Long-CLIP embeddings) rather than reasoning-chain
  generation
- When a single CoT chain is sufficient and generation cost of 9 pairings per meme
  is unjustified
- When no MLLM (LLaVA-1.5-7B, GPT-4) inference access is available, since chain generation
  depends entirely on MLLM calls
tools:
- name: LLaVA-1.5-7B
  role: MLLM run with n=3, m=3 to generate hateful and benign multi-CoT reasoning
    chains for the Hateful Memes dataset
  repo: https://github.com/BGWH123/EVL-MCoT
- name: GPT-4
  role: MLLM run with n=3, m=3 to generate the multi-CoT chains for the MultiOFF dataset
  repo: https://github.com/BGWH123/EVL-MCoT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:539790ae8730a9e7360e5ee1f475f4f5d2846bba1c203d3cfc7a52c4d7de7e00
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# mlm-inference-pipeline-construction

## Summary

Constructs an MLLM inference pipeline that produces multiple chain-of-thought (multi-CoT) reasoning chains per harmful meme — n=3 hateful and m=3 benign chains per meme — to overcome the unreliability of single-chain CoT reasoning. As used in EVL-MCoT with LLaVA-1.5-7B (Hateful Memes) and GPT-4 (MultiOFF).

## When to use

Use when interpreting harmful memes with an MLLM requires multi-perspective reasoning: you have image-text meme tuples M = {I, T} and need paired supportive/opposing CoT chains rather than one unreliable reasoning chain.

## When NOT to use

- When the downstream task needs fine-grained visual-textual feature fusion (the prototype-guided / context-guided decoder with Long-CLIP embeddings) rather than reasoning-chain generation
- When a single CoT chain is sufficient and generation cost of 9 pairings per meme is unjustified
- When no MLLM (LLaVA-1.5-7B, GPT-4) inference access is available, since chain generation depends entirely on MLLM calls

## Inputs

- Meme tuples M = {I, T} (visual image plus associated text) from the Hateful Memes dataset (10,000+ examples)
- Meme tuples from the MultiOFF dataset (743 memes)
- Prompt template q* with harmfulness indicators h0 (benign) and h1 (hateful)

## Outputs

- multi_cot_chains.: structured JSON with, for every meme, exactly 3 hateful and 3 benign reasoning chains
- 9 (n × m) hateful/benign comparative chain pairings per meme

## How to apply

For each meme tuple M = {I, T} from the Hateful Memes dataset (10,000+ examples) or MultiOFF (743 memes), build a prompt q* that incorporates a harmfulness indicator h0 (benign) and h1 (hateful) using the template 'You are a professional meme explainer. Please explain why this meme is hateful/benign, using relevant background and commonsense knowledge.' Run the MLLM — LLaVA-1.5-7B for Hateful Memes, GPT-4 for MultiOFF — with the setting n=3, m=3 to generate 3 hateful and 3 benign reasoning chains per meme, each grounded in the image-text relationship. Pair each hateful chain with each benign chain for the same meme to form the n × m = 9 comparative combinations. Serialize all per-meme chains and pairings into a structured JSON file (multi_cot_chains.). The rationale: single reasoning chains are susceptible to inconsistencies and heavily influenced by LLM prior knowledge, whereas multi-CoT with opposing perspectives enhances reasoning reliability and consistency.

## Related tools

- **LLaVA-1.5-7B** (MLLM run with n=3, m=3 to generate hateful and benign multi-CoT reasoning chains for the Hateful Memes dataset) — https://github.com/BGWH123/EVL-MCoT
- **GPT-4** (MLLM run with n=3, m=3 to generate the multi-CoT chains for the MultiOFF dataset) — https://github.com/BGWH123/EVL-MCoT

## Evaluation signals

- multi_cot_chains. exists and parses as valid JSON
- Every meme in the output has exactly 3 hateful and 3 benign reasoning chains
- Each meme yields exactly n × m = 9 hateful/benign comparative pairings
- Generated chains are grounded in the image-text relationship of the meme, not generic prior-knowledge text
- Downstream validation: EVL-MCoT should reach ~75.88 accuracy / 79.25 AUROC on Hateful Memes Test Seen and ~70.0 accuracy / 63.8 F1 on MultiOFF when these chains feed the decoders

## Limitations

- Reasoning quality is heavily influenced by the prior knowledge of the underlying LLM; single chains are unreliable, which is why multi-CoT with opposing chains is required
- The reasoning chains are long (the CoT approach requires processing 150-word sequences, beyond the 20-token comfort zone of standard CLIP); the authors list shortening chains while maintaining performance as future work
- The public repository is incomplete: only the train/ pipeline is present; models/, configs/, scripts/, pretrained weights, and usage instructions are listed as upcoming

## Evidence

- [intro] single-chain unreliability motivation: "their reasoning processes can be unreliable due to their reliance on a single reasoning chain, which makes them susceptible to inconsistencies and errors"
- [intro] multi-CoT objective: "To improve reasoning reliability, multi-CoT are generated, which enhances the consistency of the reasoning process."
- [methods] multi-CoT generation setup: [evidence span withheld — anchor status: not_found]
- [methods] chain length constraint: "The standard CLIP model struggles with inputs exceeding 20 tokens, while our CoT approach requires processing 150-word sequences."
- [readme] repository completeness: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
