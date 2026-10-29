---
name: harmful-meme-implicit-meaning-explanation
description: Use when interpreting harmful memes requires grounding implicit meanings
  in background/commonsense knowledge and a single CoT chain is too unreliable or
  inconsistent to trust.
when_to_use_negative:
- The task is fine-grained visual-textual fusion or classification decoding rather
  than generating explanatory reasoning chains (use the prototype-guided/context-guided
  decoder stages instead)
- Shorter reasoning chains are required — the paper's future work explicitly targets
  shortening chains while maintaining performance
- The input is not a multimodal meme (image + embedded text) but plain text or unimodal
  imagery, where the {I, T} tuple structure does not apply
edam_operation: http://edamontology.org/operation_0305
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LLaVA-1.5-7B
  role: MLLM used to generate the n=3 hateful and m=3 reasoning chains for the Hateful
    Memes dataset
  repo: https://github.com/BGWH123/EVL-MCoT
- name: GPT-4
  role: MLLM used as the chain generator for the MultiOFF dataset under the n=3, m=3
    setting
  repo: https://github.com/BGWH123/EVL-MCoT
- name: LLaVA
  role: Guiding MLLM paradigm for generating multi-step chain-of-thought reasoning
    processes
  repo: https://github.com/BGWH123/EVL-MCoT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:df666d81c6a8897a93a3c9f18e31fd6b120ef6cace984c80dff3293593b0f697
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - GPT-4
  - LLaVA
  - LLaVA-1.5-7B
  edam_operation: http://edamontology.org/operation_0305
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# harmful-meme-implicit-meaning-explanation

## Summary

Reconstructs EVL-MCoT's multi-CoT generation phase, in which an MLLM (LLaVA-1.5-7B or GPT-4) produces n=3 hateful and m=3 benign reasoning chains per meme M = {I, T}, whose n × m = 9 comparative pairings expose the implicit, multi-perspective semantics behind harmful memes.

## When to use

Use when interpreting harmful memes requires grounding implicit meanings in background/commonsense knowledge and a single CoT chain is too unreliable or inconsistent to trust.

## When NOT to use

- The task is fine-grained visual-textual fusion or classification decoding rather than generating explanatory reasoning chains (use the prototype-guided/context-guided decoder stages instead)
- Shorter reasoning chains are required — the paper's future work explicitly targets shortening chains while maintaining performance
- The input is not a multimodal meme (image + embedded text) but plain text or unimodal imagery, where the {I, T} tuple structure does not apply

## Inputs

- Meme tuples M = {I, T} (visual image plus associated text) from the Hateful Memes dataset (10,000+ multimodal examples)
- Meme tuples from the MultiOFF dataset (743 memes, offensive/non-offensive labels)
- Prompt template with harmfulness indicators h0 (benign) and h1 (hateful)

## Outputs

- Per-meme set of n=3 hateful reasoning chains and m=3 benign reasoning chains grounded in the image-text relationship
- n × m = 9 hateful-vs-benign comparative chain pairings per meme
- Structured JSON file multi_cot_chains. containing all serialized chains and pairings

## How to apply

Load meme tuples M = {I, T} from the Hateful Memes dataset (10,000+ examples) or MultiOFF dataset (743 memes). Build a prompt q* incorporating a harmfulness indicator h0 (benign) and h1 (hateful) using the template 'You are a professional meme explainer. Please explain why this meme is hateful/benign, using relevant background and commonsense knowledge.' Run the MLLM — LLaVA-1.5-7B for Hateful Memes, GPT-4 for MultiOFF — with the evaluated setting n=3, m=3 to generate 3 hateful and 3 benign reasoning chains per meme, grounded in the image-text relationship. Pair each hateful chain with each benign chain (n × m = 9 combinations) so explanations are contrastively validated rather than drawn from a single reasoning chain, which the paper identifies as susceptible to inconsistencies and prior-knowledge bias. Serialize all chains and pairings into a structured JSON file (multi_cot_chains.) for downstream prototype-guided and context-guided decoding.

## Related tools

- **LLaVA-1.5-7B** (MLLM used to generate the n=3 hateful and m=3 reasoning chains for the Hateful Memes dataset) — https://github.com/BGWH123/EVL-MCoT
- **GPT-4** (MLLM used as the chain generator for the MultiOFF dataset under the n=3, m=3 setting) — https://github.com/BGWH123/EVL-MCoT
- **LLaVA** (Guiding MLLM paradigm for generating multi-step chain-of-thought reasoning processes) — https://github.com/BGWH123/EVL-MCoT

## Evaluation signals

- multi_cot_chains. exists and parses as valid JSON
- Every meme in multi_cot_chains. contains exactly 3 hateful and 3 benign reasoning chains
- Every meme yields exactly 9 hateful/benign pairings (n × m = 3 × 3)
- Each chain explicitly invokes background and commonsense knowledge about the image-text relationship, per the prompt template
- Downstream EVL-MCoT results reproduce the reported metrics (e.g., 75.88 accuracy / 79.25 AUROC on Hateful Memes Test Seen; 70.0 accuracy / 63.8 F1 on MultiOFF)

## Limitations

- Reasoning chains are long (~150-word sequences); the standard CLIP model struggles with inputs exceeding 20 tokens, which motivates Long-CLIP but adds encoder constraints
- Single-chain CoT is unreliable and heavily influenced by the prior knowledge of large language models; multi-CoT mitigates but does not eliminate this dependence on the chosen MLLM (LLaVA-1.5-7B vs GPT-4)
- Repository is incomplete: only train/ is available; models/, configs/, scripts/, pretrained weights, and usage instructions are marked as upcoming, limiting reproducibility

## Evidence

- [methods] multi-CoT generation prompt design: [evidence span withheld — anchor status: not_found]
- [methods] n=3 m=3 MLLM setting: "The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM"
- [intro] unreliable single-chain CoT: "their reasoning processes can be unreliable due to their reliance on a single reasoning chain, which makes them susceptible to inconsistencies and errors"
- [methods] datasets: "The Hateful Memes dataset contains over 10,000 multimodal examples for detecting hate speech in memes, labeled as hateful or non-hateful."
- [readme] repository status: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
