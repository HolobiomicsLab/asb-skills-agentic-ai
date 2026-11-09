---
name: harmful-meme-multimodal-classification
description: Use when classifying meme images with embedded text (e.g., Hateful Memes
  Test Seen/Unseen or MultiOFF) as harmful or benign and you need CoT-grounded multimodal
  fusion with ACC/AUROC/F1 outputs.
when_to_use_negative:
- 'Inputs exceed the encoder''s context budget in a different regime: standard CLIP
  cannot handle the 150-word CoT sequences, and the skill depends on Long-CLIP''s
  248-token context — do not substitute shorter-context encoders'
- 'Single-chain CoT is sufficient: the paper argues a simple CoT approach lacks multi-perspective
  thinking, so if one reasoning chain is reliable for your task, the multi-CoT pipeline
  adds cost without clear benefit'
- Task is not binary harmful-meme classification (e.g., open-ended meme explanation);
  EVL-MCoT outputs alignment scores for hateful/benign classes, not explanatory captions
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3377
tools:
- name: Long-CLIP
  role: Frozen visual and text encoder whose 248-token context supports the 150-word
    CoT sequences; standard CLIP struggles beyond ~20 tokens
- name: LLaVA-1.5-7B
  role: MLLM backbone for multi-CoT generation on the Hateful Memes benchmark under
    n=3, m=3
- name: GPT-4
  role: MLLM backbone for multi-CoT generation on the MultiOFF benchmark
- name: AdamW
  role: Optimizer with learning rate 5e-5, weight decay 0.01, and cosine learning
    rate scheduler
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:263fbb104a5a324a5c89089010f034ca05dd88c73df71d3679aeadbd021725ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AdamW
  - GPT-4
  - LLaVA-1.5-7B
  - Long-CLIP
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3377
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# harmful-meme-multimodal-classification

## Summary

Classifies memes as hateful/offensive using EVL-MCoT: multi-chain-of-thought (multi-CoT) reasoning from an MLLM backbone fused with vision-language features via a prototype-guided and context-guided decoder over Long-CLIP encodings, evaluated on Hateful Memes and MultiOFF.

## When to use

Use when classifying meme images with embedded text (e.g., Hateful Memes Test Seen/Unseen or MultiOFF) as harmful or benign and you need CoT-grounded multimodal fusion with ACC/AUROC/F1 outputs.

## When NOT to use

- Inputs exceed the encoder's context budget in a different regime: standard CLIP cannot handle the 150-word CoT sequences, and the skill depends on Long-CLIP's 248-token context — do not substitute shorter-context encoders
- Single-chain CoT is sufficient: the paper argues a simple CoT approach lacks multi-perspective thinking, so if one reasoning chain is reliable for your task, the multi-CoT pipeline adds cost without clear benefit
- Task is not binary harmful-meme classification (e.g., open-ended meme explanation); EVL-MCoT outputs alignment scores for hateful/benign classes, not explanatory captions

## Inputs

- Meme dataset with image + embedded text pairs and binary labels (Hateful Memes: 10,000+ examples with test seen/unseen splits; MultiOFF: 743 memes, offensive/non-offensive)
- Benign/hateful CoT prompt with harmfulness indicator (n=3, m=3 chain setting)
- Frozen Long-CLIP visual and text encoders
- EVL-MCoT training pipeline from the repository's train/ directory

## Outputs

- Binary harmful/non-harmful meme predictions
- ACC and AUROC on Hateful Memes Test Seen (expected 75.88 / 79.25) and Test Unseen (expected 75.57 / 79.40)
- ACC and F1 on MultiOFF (expected 70.0 / 63.8)
- Benchmark comparison table against baselines (Image-Grid, MMBT-Region, ViLBERT, VisualBERTCOCO, LLaVA-1.5-7B, GPT-4o, MOMENTA, RGCL, SFT variants)

## How to apply

For each meme M = {I, T}, generate multi-CoT chains by prompting an MLLM (LLaVA-1.5-7B for Hateful Memes, GPT-4 for MultiOFF) with a benign/hateful prompt and harmfulness indicator, pairing m benign with n hateful chains under the n=3, m=3 setting. Encode visual inputs and the ~150-word hateful/benign CoT texts with frozen Long-CLIP encoders (248-token context; standard CLIP fails beyond ~20 tokens). Apply the prototype-guided token fusion (learnable prototypes Pr as queries over visual tokens, then attention-based fusion with learnable weights W_a, W_b, W_v, W_c) to obtain visual features S, then a context-guided attention layer (prototypes Pr~ as queries, text embeddings T_emb as keys/values) to yield enhanced text features T_embE. Compute alignment scores with cross-entropy loss using avg-pooled hateful/benign text features concatenated against S and a learnable temperature tau; train with AdamW (lr 5e-5, weight decay 0.01, cosine scheduler). Report ACC and AUROC on Hateful Memes Test Seen/Unseen and ACC and F1 on MultiOFF, comparing against baselines from Tables 1 and 2.

## Related tools

- **Long-CLIP** (Frozen visual and text encoder whose 248-token context supports the 150-word CoT sequences; standard CLIP struggles beyond ~20 tokens)
- **LLaVA-1.5-7B** (MLLM backbone for multi-CoT generation on the Hateful Memes benchmark under n=3, m=3)
- **GPT-4** (MLLM backbone for multi-CoT generation on the MultiOFF benchmark)
- **AdamW** (Optimizer with learning rate 5e-5, weight decay 0.01, and cosine learning rate scheduler)

## Evaluation signals

- Reproduced Hateful Memes Test Seen row matches Table 1: ACC 75.88, AUROC 79.25
- Reproduced Test Unseen row matches Table 1: ACC 75.57, AUROC 79.40
- Reproduced MultiOFF row matches Table 2: ACC 70.0, F1 63.8
- Ablation sanity check: removing Long-CLIP should significantly reduce ACC and AUROC per the paper's results section
- Compiled benchmark_results.csv contains EVL-MCoT row plus all baseline rows from Tables 1-2

## Limitations

- Repository README notes only the train/ pipeline is released; models/, configs/, and scripts/ are 'coming soon', with no requirements.txt, pretrained weights, or dataset demos yet
- Backbone choice is benchmark-specific: LLaVA-1.5-7B for Hateful Memes, GPT-4 for MultiOFF, which may limit transferability
- Future work per the paper: shortening the reasoning chains while maintaining performance and extending to other multimodal tasks

## Evidence

- [methods] multi-CoT with Long-CLIP encoders: "we use Long-CLIP [26], which extends the context length to 248 tokens, enabling more effective reasoning over longer texts."
- [methods] n=3, m=3 evaluation setting: "The proposed EVL-MCoT is evaluated with the setting n=3, m=3 using LLaVA as the MLLM"
- [methods] CLIP token limitation: "The standard CLIP model struggles with inputs exceeding 20 tokens, while our CoT approach requires processing 150-word sequences."
- [results] headline results: "it reaches an accuracy of 75.88 and AUROC of 79.25 on Test Seen"
- [results] MultiOFF results: [evidence span withheld — anchor status: not_found]
- [methods] optimizer settings: "The optimizer is set to AdamW [13], with a learning rate of 5e-5, a weight decay of 0.01, and a cosine learning rate scheduler"
- [results] ablation evidence: [evidence span withheld — anchor status: not_found]
- [readme] repo status: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
