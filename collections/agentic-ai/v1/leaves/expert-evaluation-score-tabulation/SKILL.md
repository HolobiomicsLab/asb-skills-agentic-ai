---
name: expert-evaluation-score-tabulation
description: Use when a paper reports expert (radiologist/clinician) Likert-style
  ratings of generated outputs in a results table that must be verified, extracted,
  or re-tabulated programmatically (e.g., mean accuracy/structure scores on a 1-5
  scale).
when_to_use_negative:
- The evaluation you need to reproduce is automated (e.g., the F1-score of 0.96 on
  concept categorization) rather than human expert ratings
- The target table reports protocol-categorized concepts or ontology mappings rather
  than scalar expert scores
tools:
- name: MedPAO
  role: Agent whose final structured outputs were the subject of the expert radiologist/clinician
    evaluation being tabulated
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: pandas
  role: Used in Python to tabulate the four published mean scores into a CSV with
    evaluator, accuracy, and structure columns
- name: Python
  role: Runtime environment for the tabulation and validation script
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:af4fc1e3db92d0766f5d72daa204e0800d2617a6fdd8a73684236a2dae71ebc9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/MiRL-IITM/medpao-agent
  repo_url: https://github.com/MiRL-IITM/medpao-agent
  tools:
  - MedPAO
  - Python
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# expert-evaluation-score-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces and tabulates the human expert-evaluation score table from the MedPAO paper (Table 4), which reports how expert radiologists and clinicians rated agent-generated structured MIMIC-CXR reports on a 5-point rubric.

## When to use

Use when a paper reports expert (radiologist/clinician) Likert-style ratings of generated outputs in a results table that must be verified, extracted, or re-tabulated programmatically (e.g., mean accuracy/structure scores on a 1-5 scale).

## When NOT to use

- The evaluation you need to reproduce is automated (e.g., the F1-score of 0.96 on concept categorization) rather than human expert ratings
- The target table reports protocol-categorized concepts or ontology mappings rather than scalar expert scores

## Inputs

- Published expert-evaluation values from Table 4 of the MedPAO paper (Section 4.4)
- Table 3 evaluation rubric definition (1-5 point scale)
- 50 agent-generated MIMIC-CXR report samples evaluated by experts

## Outputs

- CSV table with columns: evaluator method, average accuracy score, average structure score
- Validated pandas DataFrame of the four mean scores (4.64, 4.59, 4.52, 4.48)

## How to apply

Locate the expert-evaluation values in Section 4.4's Table 4 (Radiologist and Clinician evaluation of agent-generated reports) and note that scores follow the 1-5 rubric defined in Table 3. Record the mean scores per evaluator group obtained on 50 agent-generated MIMIC-CXR report samples: Clinician average Accuracy 4.64 and average Structure 4.59; Panel of two Radiologists average Accuracy 4.52 and average Structure 4.48. Use Python with pandas to build a CSV with columns for the evaluator method, average accuracy score, and average structure score. Validate by checking that the four stored values exactly equal the published Table 4 values and that all scores lie within the 1-5 rubric range.

## Related tools

- **MedPAO** (Agent whose final structured outputs were the subject of the expert radiologist/clinician evaluation being tabulated) — https://github.com/MiRL-IITM/medpao-agent
- **pandas** (Used in Python to tabulate the four published mean scores into a CSV with evaluator, accuracy, and structure columns)
- **Python** (Runtime environment for the tabulation and validation script)

## Examples

```
import pandas as pd; df = pd.DataFrame({'evaluator':['Clinician','Panel of two Radiologists'],'avg_accuracy':[4.64,4.52],'avg_structure':[4.59,4.48]}); df.to_csv('expert_eval.csv', index=False)
```

## Evaluation signals

- The four stored score values exactly equal the published Table 4 values: Clinician 4.64/4.59 and Radiologists 4.52/4.48
- All stored scores lie within the 1-5 rubric range defined by Table 3
- The CSV schema contains exactly three columns (evaluator method, average accuracy score, average structure score) with one row per evaluator group
- The overall average reported in the paper (4.52 out of 5) is consistent with the extracted per-group values
- The evaluation context matches: scores were obtained on 50 agent-generated MIMIC-CXR report samples

## Limitations

- The expert scores are subjective human ratings on a 5-point rubric (Table 3) and can only be reproduced as transcribed published values, not re-measured without the original expert panel
- The evaluation covered a limited sample (50 agent-generated MIMIC-CXR report samples), so the means carry sampling uncertainty
- The paper's abstract reports a single aggregate average of 4.52/5, which must not be conflated with the per-group means in Table 4

## Evidence
