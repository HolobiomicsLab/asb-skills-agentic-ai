---
name: llm-prompt-context-engineering
description: 'Use when you have raw LLM chain-of-thought outputs from prediction runs
  (here: 1748 high-risk option combinations across 52 programs) and want to extract,
  rank, and generalize the implicit knowledge the model used into a compact, evidence-backed
  knowledge table.'
when_to_use_negative:
- You need actual vulnerability detection — this skill only distills knowledge from
  existing prediction traces; use the full ProphetFuzz pipeline (constraint extraction
  → prediction → command assembly → fuzzing) instead
- The CoT logs lack the fourth-step prediction content, since the extraction targets
  only that step's reasoning
- The goal is per-option documentation parsing (Groff man pages), which is handled
  by ProphetFuzz's manpage_parser module, not this knowledge-distillation step
edam_operation: http://edamontology.org/operation_0306
tools:
- name: GPT-4 Turbo
  role: LLM used both to summarize per-program knowledge from parsed CoT traces and
    to synthesize and rank the 15 knowledge items under the engineered audience context
- name: Python
  role: Text parsing to extract prediction-related content (fourth prompt step) from
    the CoT outputs before summarization
  repo: https://github.com/NASP-THU/ProphetFuzz
- name: ProphetFuzz
  role: Source of the 1748 predicted high-risk option combinations and recorded chain-of-thought
    outputs across the 52 evaluation programs
  repo: https://github.com/NASP-THU/ProphetFuzz
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1145/3658644.3690231
    title: 'ProphetFuzz: Fully Automated Prediction and Fuzzing of High-Risk Option
      Combinations with Only Documentation via Large Language Model'
  source_content_hash: sha256:ec4deb926e7424f3ab37e0e3a596c1395c8063c160f6d521010aa27f7b98d9c8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/NASP-THU/ProphetFuzz
  tools:
  - GPT-4 Turbo
  - ProphetFuzz
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# llm-prompt-context-engineering

## Summary

Distills the 15 key pieces of knowledge that ProphetFuzz's LLM implicitly uses to predict high-risk option combinations by feeding recorded chain-of-thought (CoT) traces back into GPT-4 Turbo with an engineered, audience-specific prompt context. This converts opaque LLM reasoning into reusable, ranked security-testing knowledge for non-expert testers.

## When to use

Use when you have raw LLM chain-of-thought outputs from prediction runs (here: 1748 high-risk option combinations across 52 programs) and want to extract, rank, and generalize the implicit knowledge the model used into a compact, evidence-backed knowledge table.

## When NOT to use

- You need actual vulnerability detection — this skill only distills knowledge from existing prediction traces; use the full ProphetFuzz pipeline (constraint extraction → prediction → command assembly → fuzzing) instead
- The CoT logs lack the fourth-step prediction content, since the extraction targets only that step's reasoning
- The goal is per-option documentation parsing (Groff man pages), which is handled by ProphetFuzz's manpage_parser module, not this knowledge-distillation step

## Inputs

- Chain-of-thought (CoT) LLM output logs from prediction runs covering 1748 predicted high-risk option combinations across 52 programs
- Prediction prompt structure (knowledge corresponds to the fourth step of the prompt)
- Python text-parsing script to isolate prediction-related CoT content

## Outputs

- knowledge_table.csv with exactly 15 ranked rows of (knowledge item, supporting option-combination evidence)
- Ranked knowledge list headed by: Resource Management and Limits; Complex Data Processing; Output and Format Manipulation

## How to apply

Load the complete CoT outputs recorded by the LLM during each prediction run (from ProphetFuzz's predictCombinations runs over the 52 evaluation programs), and parse out only the content directly related to the fourth step of the prediction prompt using Python-based text parsing. For each program, prompt GPT-4 Turbo to 'summarize the knowledge of the categories of program option combinations that may have potential vulnerabilities'. Compile the per-program summaries and have GPT-4 Turbo synthesize them, ranking knowledge items from most to least common. Critically, set the prompt context to the target audience — 'For software testers who do not understand security, they hope to have easy-to-understand knowledge to conduct efficient security testing' — so the synthesized knowledge is actionable rather than technical. Capture 15 key knowledge pieces, each paired with supporting option-combination evidence from the traces, and write them to knowledge_table.csv.

## Related tools

- **GPT-4 Turbo** (LLM used both to summarize per-program knowledge from parsed CoT traces and to synthesize and rank the 15 knowledge items under the engineered audience context)
- **Python** (Text parsing to extract prediction-related content (fourth prompt step) from the CoT outputs before summarization) — https://github.com/NASP-THU/ProphetFuzz
- **ProphetFuzz** (Source of the 1748 predicted high-risk option combinations and recorded chain-of-thought outputs across the 52 evaluation programs) — https://github.com/NASP-THU/ProphetFuzz

## Examples

```
client.chat.completions.create(model="gpt-4-turbo", messages=[{"role":"user","content":"For software testers who do not understand security, summarize the knowledge of the categories of program option combinations that may have potential vulnerabilities"}])
```

## Evaluation signals

- knowledge_table.csv contains exactly 15 ranked rows
- Top three ranked knowledge items match the paper's reported categories: Resource Management and Limits; Complex Data Processing; Output and Format Manipulation
- Every knowledge row carries supporting option-combination evidence traceable to the CoT traces of the 1748 predictions
- Summaries only use CoT content from the fourth step of the prediction prompt (no contamination from constraint-extraction or command-assembly reasoning)
- Synthesized knowledge is phrased for non-security testers per the audience context, i.e., understandable without domain expertise

## Limitations

- Knowledge is distilled from one LLM (GPT-4 Turbo) on 52 programs from 40 packages, so the 15 items may not generalize to other models, targets, or documentation formats
- Quality depends on the CoT traces faithfully reflecting the fourth prediction step; hallucinated or truncated traces propagate into the distilled knowledge
- LLM summarization is itself non-deterministic, so re-running the synthesis can reorder or rephrase the ranked knowledge items

## Evidence

- [methods] For each program, instruct GPT-4 Turbo to 'summarize the knowledge of the categories of program option combinations that may have potential vulnerabilities': [evidence span withheld — anchor status: not_found]
- [methods] Set the prompt context to 'For software testers who do not understand security, they hope to have easy-to-understand knowledge to conduct efficient security testing' and capture 15 key pieces of knowledge: [evidence span withheld — anchor status: not_found]
- [supplementary] 1. Resource Management and Limits. Options that affect re-source allocation and limits can lead to vulnerabilities: [evidence span withheld — anchor status: not_found]
