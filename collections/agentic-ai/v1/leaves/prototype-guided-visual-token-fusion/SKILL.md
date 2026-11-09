---
name: prototype-guided-visual-token-fusion
description: Use when you have tokenized visual representations from a frozen CLIP-style
  encoder and separate hateful/benign text embeddings, and need fine-grained cross-modal
  fusion beyond shallow concatenation before a similarity-based classifier.
when_to_use_negative:
- Input text exceeds ~20 tokens and a standard (non-Long) CLIP text encoder is used
  — the paper notes the standard CLIP model struggles with such inputs, so Long-CLIP
  is required for 150-word CoT sequences
- The task requires a single-image classifier with no paired textual hypotheses (hateful/benign
  CoT texts), since the alignment-score supervision depends on concatenated pooled
  text features
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Long-CLIP
  role: Frozen visual and text encoder producing visual tokens H and text embeddings
    T_emb (extended to 248 tokens) that feed the prototype-guided and context-guided
    attention layers
  repo: https://github.com/BGWH123/EVL-MCoT
- name: AdamW
  role: Optimizer for the fusion module with learning rate 5e-5, weight decay 0.01,
    and cosine learning rate scheduler
  repo: https://github.com/BGWH123/EVL-MCoT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/978-981-95-3349-7_31
    title: 'EVL-MCoT: Enhanced Vision-Language Multi-CoT for Harmful Meme Detection'
  source_content_hash: sha256:3c6a660568f2361adb8c43c9d2248579d6141bc4334c5928bfa3c4487b2d4051
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BGWH123/EVL-MCoT
  tools:
  - AdamW
  - Long-CLIP
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# prototype-guided-visual-token-fusion

## Summary

A decoder fusion skill from EVL-MCoT that aggregates Long-CLIP visual tokens into refined visual features S via learnable prototype cross-attention, then uses the resulting prototype features as queries against textual key-value pairs to enrich hateful/benign CoT text embeddings for fine-grained visual-textual alignment.

## When to use

Use when you have tokenized visual representations from a frozen CLIP-style encoder and separate hateful/benign text embeddings, and need fine-grained cross-modal fusion beyond shallow concatenation before a similarity-based classifier.

## When NOT to use

- Input text exceeds ~20 tokens and a standard (non-Long) CLIP text encoder is used — the paper notes the standard CLIP model struggles with such inputs, so Long-CLIP is required for 150-word CoT sequences
- The task requires a single-image classifier with no paired textual hypotheses (hateful/benign CoT texts), since the alignment-score supervision depends on concatenated pooled text features

## Inputs

- Visual tokens H in R^{N x M x D} from the frozen Long-CLIP visual encoder
- Text embeddings T_emb for hateful and benign CoT texts from the frozen Long-CLIP text encoder
- P learnable prototype vectors Pr in R^{P x D}
- Trainable attention matrices W_a, W_b, W_c, W_v and learnable temperature tau

## Outputs

- Refined visual features S from attention-based feature fusion over prototype tokens
- Enriched hateful and benign text representations Th_embE and Tb_embE
- Classification logits from the temperature-scaled alignment score between pooled text features and S (Eq. 7)

## How to apply

Load visual tokens H (N x M x D) from the frozen Long-CLIP visual encoder and text embeddings T_emb from the frozen Long-CLIP text encoder for each meme's hateful and benign CoT texts. Initialize P learnable prototypes Pr in R^{P x D} and run prototype-guided token fusion via cross-attention with prototypes as queries and visual tokens as keys/values, Softmax(Pr H^T / sqrt(D))H, followed by layer normalization and a residual connection (Eq. 2). Derive refined visual features S with attention-based feature fusion using tanh-softmax attention weights computed from trainable matrices W_a, W_b, W_c, W_v (Eqs. 3-5). Then apply the context-guided attention layer with prototype features Pr~ as queries and T_emb as keys and values (Eq. 6) to produce enriched text representations Th_embE and Tb_embE. Train with AdamW (learning rate 5e-5, weight decay 0.01, cosine scheduler, batch size 8 with gradient accumulation every 2 steps) on the Hateful Memes dataset.

## Related tools

- **Long-CLIP** (Frozen visual and text encoder producing visual tokens H and text embeddings T_emb (extended to 248 tokens) that feed the prototype-guided and context-guided attention layers) — https://github.com/BGWH123/EVL-MCoT
- **AdamW** (Optimizer for the fusion module with learning rate 5e-5, weight decay 0.01, and cosine learning rate scheduler) — https://github.com/BGWH123/EVL-MCoT

## Examples

```
import torch; A = torch.softmax(Pr @ H.transpose(-1, -2) / (D ** 0.5), dim=-1) @ H; Pr = Pr + layer_norm(A)
```

## Evaluation signals

- The cross-attention output shape matches Softmax(Pr H^T / sqrt(D))H with prototypes as queries and visual tokens as keys/values, followed by layer norm and residual connection
- Refined visual features S are produced with tanh-softmax attention weights from W_a, W_b, W_c, W_v (Eqs. 3-5)
- Context-guided layer output uses prototype features Pr~ as queries and text features T_emb as keys and values (Eq. 6)
- End-to-end accuracy on Hateful Memes Test Seen approaches 75.88% with AUROC near 79.25 (75.57/79.40 on Test Unseen)
- Removing Long-CLIP from the pipeline significantly reduces ACC and AUROC, confirming the encoder tokens drive the fusion

## Limitations

- Reasoning chains from the MLLM can be long; future work targets shortening chains while maintaining performance
- Standard CLIP cannot handle the 150-word CoT inputs this fusion expects; Long-CLIP's 248-token context is required
- Repository is incomplete: model definitions, configs, and pretrained weights are listed as 'coming soon' in the README, so reproduction depends on reimplementing from the equations

## Evidence

- [abstract] we design a prototype-guided and context-guided decoding framework, which incorporates visual prototypes to guide the fusion process: "we design a prototype-guided and context-guided decoding framework, which incorporates visual prototypes to guide the fusion process"
- [methods] prototype-guided token fusion mechanism aggregates visual tokens: "A prototype-guided token fusion mechanism is introduced to aggregate these visual tokens into a final representation suitable for similarity computation."
- [methods] context-guided attention layer queries and keys/values: "we employ a context-guided attention layer, where the visual prototype features Pr ∼ serve as the queries Q, while the text features T emb serve as both the keys K and values V"
- [methods] optimizer settings for training the module: "The optimizer is set to AdamW [13], with a learning rate of 5e-5, a weight decay of 0.01, and a cosine learning rate scheduler"
- [readme] README status of released code: "The core training code is available in the `train/` directory. Other components including model definitions, sample data, and configuration files will be added in upcoming updates."
