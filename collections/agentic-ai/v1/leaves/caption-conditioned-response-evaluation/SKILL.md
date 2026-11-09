---
name: caption-conditioned-response-evaluation
description: Use when benchmarking VLMs on image-inclusive MCQs and you need to isolate
  how image access mode (none, LLM caption, human caption, raw image) changes accuracy,
  with paired significance tests against a no-image baseline.
when_to_use_negative:
- The question set contains no image-inclusive items (images excluded, as in the 455-question
  text-only ACG board set in the README's literature table)
- You are evaluating text-only LLM performance on the 162 text-only questions — no
  caption or image condition applies
- No answer key or baseline run is available, so paired scenario-vs-baseline scoring
  is impossible
edam_topics:
- http://edamontology.org/topic_0218
tools:
- name: LangChain (0.1.17 and 0.2.4)
  role: Standardizes API interactions with each VLM across the four input-scenario
    conditions
- name: OpenAI Python package
  role: Runs the GPT-3.5 pipeline that extracts the selected MCQ option from each
    VLM's textual response
- name: Streamlit (1.33)
  role: Hosts the human evaluation app used to review the semi-automatically extracted
    answers
- name: Sdamirsa/LLM-VLM-in-Gastroenterology
  role: Source repository for the 2022 ACG-SA questions, answer images, and human-generated
    captions
  repo: github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41746-025-02174-0
    title: Benchmarking proprietary and open-source language and vision-language models
      for gastroenterology clinical reasoning
  source_content_hash: sha256:c9c26efd75846e971084fd6c59d9d933266115bff64d2335819b29da12797728
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Sdamirsa/LLM-VLM-in-Gastroenterology
  tools:
  - Claude 3.5 Sonnet
  - LangChain (0.1.17 and 0.2.4)
  - Llama 3.2-11b (8-bit)
  - OpenAI Python package
  - Python
  - Sdamirsa/LLM-VLM-in-Gastroenterology
  - Streamlit (1.33)
  - o1-preview
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# caption-conditioned-response-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A four-scenario VLM benchmarking method that scores vision-language model accuracy on the 138 image-inclusive 2022 ACG-SA gastroenterology MCQs under a no-image baseline, LLM-generated captions, human-generated captions, and direct image input, then paired-tests each scenario against the baseline.

## When to use

Use when benchmarking VLMs on image-inclusive MCQs and you need to isolate how image access mode (none, LLM caption, human caption, raw image) changes accuracy, with paired significance tests against a no-image baseline.

## When NOT to use

- The question set contains no image-inclusive items (images excluded, as in the 455-question text-only ACG board set in the README's literature table)
- You are evaluating text-only LLM performance on the 162 text-only questions — no caption or image condition applies
- No answer key or baseline run is available, so paired scenario-vs-baseline scoring is impossible

## Inputs

- 138 image-inclusive ACG Self-Assessment 2022 MCQs with answer images and answer key
- Human-generated captions for the answer images
- LLM-generated (VLM) captions of the answer images
- API/local access to the VLMs under test (e.g., Claude3-Sonnet, Llama 3.2-11b 8-bit)

## Outputs

- Per-VLM, per-scenario accuracy table over the 138 image-based questions (correct/incorrect per response)
- Paired-test p-values comparing each image condition to the no-image baseline
- Human-reviewed extracted answer options (GPT-3.5 extraction output)

## How to apply

Load the 138 image-inclusive questions, their answer images, answer key, and human-generated captions for the 2022 ACG-SA from the github.com/Sdamirsa/LLM-VLM-in-Gastroenterology repository. Query each VLM (e.g., Claude3-Sonnet, 8-bit Llama 3.2-11b) under the no-image baseline using the benchmark QA prompt via Python 3.11 with LangChain (0.1.17 and 0.2.4) to standardize API interactions, then run the three image-informed conditions: LLM-generated captions, human-generated captions, and direct image input. Extract the selected multiple-choice option from each textual response using the semi-automated GPT-3.5 extraction pipeline (OpenAI Python package) followed by human review in the Streamlit (1.33) evaluation app. Score each response as correct/incorrect against the answer key, compute accuracy per VLM per scenario over the 138 questions, and run paired statistical tests of each image condition against the no-image baseline, reporting p-values (direct images: Claude3-Sonnet +20.2%, p=0.001; Llama3.2-11b +8.7%, p=0.047; human captions ~10% average improvement).

## Related tools

- **LangChain (0.1.17 and 0.2.4)** (Standardizes API interactions with each VLM across the four input-scenario conditions)
- **OpenAI Python package** (Runs the GPT-3.5 pipeline that extracts the selected MCQ option from each VLM's textual response)
- **Streamlit (1.33)** (Hosts the human evaluation app used to review the semi-automatically extracted answers)
- **Sdamirsa/LLM-VLM-in-Gastroenterology** (Source repository for the 2022 ACG-SA questions, answer images, and human-generated captions) — github.com/Sdamirsa/LLM-VLM-in-Gastroenterology

## Examples

```
from langchain.chains import LLMChain; pred = LLMChain(llm=vlm, prompt=qa_prompt).run({"question": q, "caption": human_caption})  # one run per scenario
```

## Evaluation signals

- The per-scenario accuracy table over all 138 image-based questions reproduces the reported pattern: ~10% improvement with human captions, unchanged with original images, decline with LLM-generated captions
- Paired tests show direct-image gains of +20.2% (p=0.001) for Claude3-Sonnet and +8.7% (p=0.047) for Llama3.2-11b over the no-image baseline
- Every model response yields an extracted single option that passes human review in the Streamlit (1.33) app; incoherent-response models (like Meditron-7b-Q16, excluded for >50% incoherence) are flagged and dropped
- Each of the four scenarios is run under identical prompt settings, so any accuracy difference is attributable to image-access mode rather than prompt changes

## Limitations

- Prompt/settings were optimized only for OpenAI (GPT-3.5) and reused for other LLMs; per-model prompt configuration was suggested as more effective
- Limited number of experimental runs — no mean and SD were reported to assess model consistency
- Binary correct/incorrect evaluation loses information; a correct option selection may still contain hallucinations
- Access to the ACG-SA examinations is restricted by a paywall, and commercial models' knowledge cutoffs changed during experimentation without documented notification

## Evidence

- [abstract] VLM accuracy on image-containing questions improved (~10%) when given human-generated captions, remained unchanged with original images, and declined with LLM-generated captions.: "VLM accuracy on image-containing questions improved (~10%) when given human-generated captions, remained unchanged with original images, and declined with LLM-generated captions."
- [discussion] We compared baseline performance (without image information) to performance with accompanying LLM-generated captions or human-generated captions: "We compared baseline performance (without image information) to performance with accompanying LLM-generated captions or human-generated captions"
- [discussion] we used the GPT-3.5 API to extract the selected option from the LLM textual response: "we used the GPT-3.5 API to extract the selected option from the LLM textual response"
