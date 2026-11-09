---
name: structured-reasoning-chain-serialization
description: Use when an MLLM has generated multiple chain-of-thought explanations
  per multimodal example and they must be persisted in a schema-valid JSON file with
  paired hateful/benign reasoning for contrastive alignment.
when_to_use_negative:
- When only a single reasoning chain per example is needed — the EVL-MCoT design specifically
  requires n=3, m=3 multi-perspective chains; a single chain is unreliable and lacks
  multi-perspective thinking
- When the downstream encoder is standard CLIP rather than Long-CLIP — serialized
  150-word CoT sequences exceed the ~20-token input limit of the standard CLIP text
  encoder
- When inputs are not image-text meme tuples (e.g., plain-text documents) — chains
  must be grounded in the image-text relationship of M = {I, T}
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3372
- http://edamontology.org/topic_0091
tools:
- name: LLaVA-1.5-7B
  role: MLLM used to generate n=3 hateful and m=3 multi-CoT reasoning chains per Hateful
    Memes example
  repo: https://github.com/BGWH123/EVL-MCoT
- name: GPT-4
  role: MLLM used to generate the paired hateful/benign reasoning chains for the MultiOFF
    dataset (n=3, m=3)
- name: Long-CLIP
  role: Downstream frozen text encoder that consumes the serialized hateful/benign
    chain texts (248-token context supports 150-word CoT sequences)
  repo: https://github.com/BGWH123/EVL-MCoT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:e9782685fdadc014a17358366d79e2c36ea14205976ee0cb7e3518436d9223bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - GPT-4
  - LLaVA
  - LLaVA-1.5-7B
  - Long-CLIP
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3372
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# structured-reasoning-chain-serialization

## Summary

Serializes the multi-CoT reasoning chains produced by an MLLM (e.g., LLaVA-1.5-7B or GPT-4) for each harmful-meme tuple M = {I, T} into a structured JSON file (multi_cot_chains.), preserving per-meme hateful/benign chains and their n × m comparative pairings for downstream contrastive training in EVL-MCoT.

## When to use

Use when an MLLM has generated multiple chain-of-thought explanations per multimodal example and they must be persisted in a schema-valid JSON file with paired hateful/benign reasoning for contrastive alignment.

## When NOT to use

- When only a single reasoning chain per example is needed — the EVL-MCoT design specifically requires n=3, m=3 multi-perspective chains; a single chain is unreliable and lacks multi-perspective thinking
- When the downstream encoder is standard CLIP rather than Long-CLIP — serialized 150-word CoT sequences exceed the ~20-token input limit of the standard CLIP text encoder
- When inputs are not image-text meme tuples (e.g., plain-text documents) — chains must be grounded in the image-text relationship of M = {I, T}

## Inputs

- Meme tuples M = {I, T} (image plus associated text) from the Hateful Memes dataset (10,000+ examples)
- Meme tuples from the MultiOFF dataset (743 memes, offensive/non-offensive labels)
- MLLM outputs: n=3 hateful and m=3 benign reasoning chains per meme from LLaVA-1.5-7B or GPT-4

## Outputs

- multi_cot_chains. — structured JSON containing, for every meme, exactly 3 hateful chains, 3 benign chains, and their 9 hateful/benign comparative pairings

## How to apply

Load each meme tuple M = {I, T} from the Hateful Memes dataset (10,000+ examples) or MultiOFF dataset (743 memes), and prompt the MLLM with the template 'You are a professional meme explainer. Please explain why this meme is hateful/benign, using relevant background and commonsense knowledge.' using harmfulness indicators h0 (benign) and h1 (hateful). Generate n=3 hateful and m=3 benign reasoning chains per meme grounded in the image-text relationship, using LLaVA-1.5-7B for Hateful Memes and GPT-4 for MultiOFF. Pair every hateful chain with every benign chain for the same meme to form the n × m = 9 comparative combinations, then serialize all chains and pairings into a structured JSON file named multi_cot_chains.. Rationale: single-chain CoT is unreliable and susceptible to inconsistencies, so multi-perspective supportive and opposing chains provide the logical foundation the downstream Long-CLIP text encoder consumes as enhanced hateful and benign text features.

## Related tools

- **LLaVA-1.5-7B** (MLLM used to generate n=3 hateful and m=3 multi-CoT reasoning chains per Hateful Memes example) — https://github.com/BGWH123/EVL-MCoT
- **GPT-4** (MLLM used to generate the paired hateful/benign reasoning chains for the MultiOFF dataset (n=3, m=3))
- **Long-CLIP** (Downstream frozen text encoder that consumes the serialized hateful/benign chain texts (248-token context supports 150-word CoT sequences)) — https://github.com/BGWH123/EVL-MCoT

## Evaluation signals

- multi_cot_chains. exists and parses as valid JSON
- For every meme, the JSON contains exactly 3 hateful and 3 benign reasoning chains
- The per-meme pairing structure yields exactly n × m = 9 hateful/benign comparative combinations
- Each chain is grounded in the image-text relationship of its meme tuple M = {I, T}, not generic text
- Downstream compatibility check: chain texts encode successfully with the frozen Long-CLIP text encoder (context length 248 tokens handles the 150-word sequences)

## Limitations

- The repository README indicates only the train/ pipeline is currently available; models/, configs/, and scripts/ (including preprocessing/serialization utilities) are marked 'coming soon', so serialization must be implemented manually
- The authors list shortening the reasoning chains while maintaining performance as future work, so current chains are long (150-word sequences) and costly to encode
- Multi-CoT generation is heavily influenced by the prior knowledge of the underlying LLM, so chain quality varies with the chosen MLLM (LLaVA-1.5-7B vs GPT-4)

## Evidence

- [methods] multi-CoT is introduced. For each meme example M = {I, T}, a prompt q* is designed that incorporates a harmfulness indicator: [evidence span withheld — anchor status: not_found]
- [methods] The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM: "The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM"
- [methods] we first encode both the hateful text and benign text using the frozen Long-CLIP [26] text encoder: "we first encode both the hateful text and benign text using the frozen Long-CLIP [26] text encoder"
- [intro] their reasoning processes can be unreliable due to their reliance on a single reasoning chain, which makes them susceptible to inconsistencies and errors: "their reasoning processes can be unreliable due to their reliance on a single reasoning chain, which makes them susceptible to inconsistencies and errors, as well as being heavily influenced by the"
- [readme] The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates.: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
