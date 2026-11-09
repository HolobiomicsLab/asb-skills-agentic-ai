---
name: survey-framework-information-extraction
description: Use when a survey article explicitly defines a numbered or named framework
  (e.g. a lifecycle taxonomy) and the task is to reproduce it as a structured stage-to-method
  mapping keyed by the paper's own stage definitions.
when_to_use_negative:
- The article proposes no explicitly named/numbered framework and stages would have
  to be inferred — extraction would be speculative rather than grounded in the paper's
  own definitions
- The target is a single primary-study method description rather than a survey's cross-cutting
  taxonomy; stage-to-method mapping would not apply
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: rongzhou7/Awesome-Digital-Twin-AI
  role: Companion curated repository whose README restates the four-stage framework
    and per-stage methodology labels, used as a cross-check for the extracted mapping
  repo: https://github.com/rongzhou7/Awesome-Digital-Twin-AI
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:373e59a8bfd76305586e470ade7d35a1cedab745153f5e2f916966bfcd132cfe
  source_papers:
  - doi: 10.48550/arxiv.2601.01321
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/rongzhou7/Awesome-Digital-Twin-AI
  tools:
  - rongzhou7/Awesome-Digital-Twin-AI
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# survey-framework-information-extraction

## Summary

Extracts a survey paper's canonical framework as a structured stage-to-methodology mapping, e.g. the four-stage digital-twin lifecycle (Modeling, Mirroring, Intervening, Autonomous Management) with the AI methodology class assigned to each stage.

## When to use

Use when a survey article explicitly defines a numbered or named framework (e.g. a lifecycle taxonomy) and the task is to reproduce it as a structured stage-to-method mapping keyed by the paper's own stage definitions.

## When NOT to use

- The article proposes no explicitly named/numbered framework and stages would have to be inferred — extraction would be speculative rather than grounded in the paper's own definitions
- The target is a single primary-study method description rather than a survey's cross-cutting taxonomy; stage-to-method mapping would not apply

## Inputs

- Survey paper text (arXiv:2601.01321) — abstract, introduction, and methods sections as plain text
- Companion repository README (rongzhou7/Awesome-Digital-Twin-AI) describing the framework sections

## Outputs

- CSV with exactly four stage rows, keyed by stage name, with a methodology column (e.g. 'Autonomous Management' -> 'LLMs, foundation models, and agentic AI')

## How to apply

Retrieve the survey (here arXiv:2601.01321, 'Digital Twin AI: Opportunities and Challenges from Large Language Models to World Models') and load its abstract, introduction, and methods sections as plain text. Extract the canonical stage entities using the paper's own stage definitions — Modeling, Mirroring, Intervening, Autonomous Management — rather than paraphrasing stage names. For each stage, record the AI methodology class the paper assigns: physics-informed AI for Modeling, generative AI for Mirroring, predictive AI for Intervening, and LLMs/foundation models/agentic AI for Autonomous Management. Cross-check the assignments against the repository README's framework overview, which lists the same four stages with the same methodology labels. Assemble the mapping as a structured CSV keyed by stage name with a methodology column, using manual expert review to validate each row against the paper's stated assignment.

## Related tools

- **rongzhou7/Awesome-Digital-Twin-AI** (Companion curated repository whose README restates the four-stage framework and per-stage methodology labels, used as a cross-check for the extracted mapping) — https://github.com/rongzhou7/Awesome-Digital-Twin-AI

## Evaluation signals

- CSV exists and contains exactly four stage rows (Modeling, Mirroring, Intervening, Autonomous Management)
- Each row's methodology field matches the paper's stated assignment (e.g. 'Autonomous Management' maps to 'LLMs, foundation models, and agentic AI')
- Stage names are verbatim from the paper's own definitions, not paraphrases
- Assignments agree between the article's methods section and the repository README's framework overview

## Limitations

- The repository README's per-stage resource listings (physics-based methods, physics-informed AI models, simulators, LLM resources, etc.) are placeholder 'Coming soon' sections, so the mapping is validated only at the stage/methodology-label level, not against populated resource lists
- The framework is the paper's own distillation; stage boundaries (e.g. Mirroring vs. Intervening) follow the authors' conceptualization and may not transfer to other digital-twin taxonomies

## Evidence
