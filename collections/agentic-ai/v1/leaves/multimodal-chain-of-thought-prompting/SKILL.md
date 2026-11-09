---
name: multimodal-chain-of-thought-prompting
description: 'Use when interpreting harmful memes requires background knowledge and
  multi-perspective reasoning: for each meme M = {I, T}, generate paired benign/hateful
  CoT chains to feed a downstream fusion-and-alignment model.'
when_to_use_negative:
- 'The downstream encoder is standard CLIP rather than Long-CLIP: CoT sequences of
  ~150 words exceed the ~20-token comfort zone of standard CLIP, so shortening or
  replacing chains is required first (the paper lists chain-shortening as future work).'
- The task is plain image classification without meme text or implicit/hateful semantics
  — the skill's value comes from explaining image-text relations via benign/hateful
  comparative reasoning.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: LLaVA-1.5-7B
  role: MLLM backbone that generates the n=3 hateful and m=3 benign CoT chains for
    the Hateful Memes dataset
  repo: https://github.com/BGWH123/EVL-MCoT
- name: GPT-4
  role: MLLM backbone that generates the multi-CoT chains for the MultiOFF dataset
    (n=3, m=3 setting)
- name: Long-CLIP
  role: Frozen text encoder (248-token context) that encodes the ~150-word benign/hateful
    CoT texts into T_emb for the context-guided decoder
  repo: https://github.com/BGWH123/EVL-MCoT
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:69f1bd6a828e816451c70253cb123cef9bca13b3ce3550a41dc5970cd3b89659
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - AdamW
  - GPT-4
  - LLaVA
  - LLaVA-1.5-7B
  - Long-CLIP
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multimodal-chain-of-thought-prompting

## Summary

Generates multiple benign and hateful chain-of-thought (CoT) reasoning chains per meme with an MLLM (LLaVA-1.5-7B or GPT-4) so that comparative multi-CoT evidence, rather than a single unreliable chain, grounds harmful meme detection. This is the Multi-CoT Generation phase (Section 3.1) of the EVL-MCoT framework.

## When to use

Use when interpreting harmful memes requires background knowledge and multi-perspective reasoning: for each meme M = {I, T}, generate paired benign/hateful CoT chains to feed a downstream fusion-and-alignment model.

## When NOT to use

- The downstream encoder is standard CLIP rather than Long-CLIP: CoT sequences of ~150 words exceed the ~20-token comfort zone of standard CLIP, so shortening or replacing chains is required first (the paper lists chain-shortening as future work).
- The task is plain image classification without meme text or implicit/hateful semantics — the skill's value comes from explaining image-text relations via benign/hateful comparative reasoning.

## Inputs

- Meme tuples M = {I, T} (image plus associated text) from the Hateful Memes dataset (10,000+ examples, Test Seen and Test Unseen splits)
- Meme tuples from the MultiOFF dataset (743 memes, offensive/non-offensive labels)
- Prompt template q* with harmfulness indicators h0 (benign) and h1 (hateful)
- MLLM choice: LLaVA-1.5-7B (Hateful Memes) or GPT-4 (MultiOFF)

## Outputs

- multi_cot_chains. containing, for every meme, exactly 3 hateful and 3 benign reasoning chains forming 9 benign-hateful pairs
- Long-CLIP text embeddings of the hateful and benign CoT sequences (T_emb) consumed by the context-guided decoder and alignment scoring

## How to apply

For each meme M = {I, T}, construct a prompt q* that incorporates a harmfulness indicator (h0 = benign, h1 = hateful) using the template 'You are a professional meme explainer. Please explain why this meme is hateful/benign, using relevant background and commonsense knowledge.' Run the MLLM — LLaVA-1.5-7B for Hateful Memes, GPT-4 for MultiOFF — to produce n=3 hateful and m=3 benign reasoning chains per meme, grounded in the image-text relationship. Pair each hateful chain with each benign chain for the same meme to form n × m = 9 comparative combinations, and serialize all chains and pairings into a structured JSON file (multi_cot_chains.). Downstream, encode the hateful/benign CoT texts with the frozen Long-CLIP text encoder (248-token context, needed because standard CLIP struggles beyond ~20 tokens while CoT sequences run ~150 words). The rationale is that single-chain CoT is unreliable and heavily influenced by LLM prior knowledge, whereas multi-CoT decomposes meme semantics from multiple perspectives and improves reasoning consistency.

## Related tools

- **LLaVA-1.5-7B** (MLLM backbone that generates the n=3 hateful and m=3 benign CoT chains for the Hateful Memes dataset) — https://github.com/BGWH123/EVL-MCoT
- **GPT-4** (MLLM backbone that generates the multi-CoT chains for the MultiOFF dataset (n=3, m=3 setting))
- **Long-CLIP** (Frozen text encoder (248-token context) that encodes the ~150-word benign/hateful CoT texts into T_emb for the context-guided decoder) — https://github.com/BGWH123/EVL-MCoT

## Examples

```
python train/run_multi_cot.py --mllm llava-1.5-7b --n_hateful 3 --n_benign 3 --prompt 'You are a professional meme explainer. Please explain why this meme is hateful/benign, using relevant background and commonsense knowledge.'
```

## Evaluation signals

- multi_cot_chains. exists, parses as valid JSON, and contains for every meme exactly 3 hateful and 3 benign chains forming 9 (n × m) pairs.
- Downstream EVL-MCoT metrics on Hateful Memes match the reported Table 1 values: Test Seen ACC 75.88 / AUROC 79.25; Test Unseen ACC 75.57 / AUROC 79.40.
- MultiOFF results match Table 2: ACC 70.0 and F1 63.8.
- Ablation sanity: removing Long-CLIP encoding of the CoT texts significantly reduces ACC and AUROC, confirming the CoT embeddings carry the signal.
- Each generated chain explicitly explains why the meme is hateful or benign with background/commonsense knowledge, per the prompt template.

## Limitations

- Single-chain CoT reasoning is unreliable, susceptible to inconsistencies and errors, and heavily influenced by the prior knowledge of the underlying LLM — hence the n=3, m=3 multi-CoT setting, which multiplies MLLM inference cost per meme.
- The paper's future work notes the need for strategies for shortening the reasoning chains while maintaining performance, and for extending the application to other multimodal tasks.
- The public repository (github.com/BGWH123/EVL-MCoT) currently only ships the train/ pipeline; model definitions, configs, sample data, and pretrained weights are listed as upcoming, so full reproduction requires assembling components.

## Evidence

- [methods] For each meme example M = {I, T}, a prompt q* is designed that incorporates a harmfulness indicator: [evidence span withheld — anchor status: not_found]
- [methods] n=3, m=3 with LLaVA / GPT-4: "The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM"
- [methods] Long-CLIP 248-token context rationale: "The standard CLIP model struggles with inputs exceeding 20 tokens, while our CoT approach requires processing 150-word sequences."
- [intro] Single-chain CoT unreliability: "their reasoning processes can be unreliable due to their reliance on a single reasoning chain, which makes them susceptible to inconsistencies and errors"
- [readme] Repository only ships training pipeline: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
