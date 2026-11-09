---
name: mimic-cxr-report-evaluation
description: Use when reproducing or verifying human-expert evaluation scores of agent-generated
  structured radiology reports, specifically the Clinician (4.64 Accuracy / 4.59 Structure)
  and Radiologist (4.52 / 4.48) averages from MedPAO's Table 4.
when_to_use_negative:
- Evaluating the concept-categorization sub-task, which is measured by F1-score (0.96),
  not by the human expert rubric
- Reports not drawn from the 50 MIMIC-CXR samples evaluated in the paper's human study,
  since the published scores cannot be verified against them
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3063
tools:
- name: MedPAO
  role: Agent framework whose final structured MIMIC-CXR report outputs are the object
    of the expert evaluation being reproduced
  repo: https://github.com/MiRL-IITM/medpao-agent
- name: Python
  role: Runtime for the tabulation script (Python 3.8 or higher per README)
- name: pandas
  role: Used to build and store the score table as a CSV with evaluator-method, accuracy,
    and structure columns
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/978-3-032-06004-4_4
    title: 'MedPAO: A Protocol-Driven Agent for Structuring Medical Reports'
  source_content_hash: sha256:4fc7ba8b5288752175ad2b88586e3c5c51f05716606355f2a4c52424e3e36fe6
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3063
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mimic-cxr-report-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the MedPAO expert-evaluation score table (Table 4), which reports average Accuracy and Structure ratings on a 5-point rubric given by a clinician and a panel of two radiologists to 50 agent-generated structured MIMIC-CXR reports.

## When to use

Use when reproducing or verifying human-expert evaluation scores of agent-generated structured radiology reports, specifically the Clinician (4.64 Accuracy / 4.59 Structure) and Radiologist (4.52 / 4.48) averages from MedPAO's Table 4.

## When NOT to use

- Evaluating the concept-categorization sub-task, which is measured by F1-score (0.96), not by the human expert rubric
- Reports not drawn from the 50 MIMIC-CXR samples evaluated in the paper's human study, since the published scores cannot be verified against them

## Inputs

- Published expert-evaluation values from MedPAO Section 4.4, Table 4 (Clinician 4.64/4.59; Radiologists 4.52/4.48)
- 5-point rating rubric definition from MedPAO Table 3
- 50 agent-generated structured MIMIC-CXR report samples as the evaluated population

## Outputs

- CSV table with columns: evaluator method, average accuracy score, average structure score (four score values: 4.64, 4.59, 4.52, 4.48)
- Validation check confirming all scores lie within the 1-5 rubric range and match the published values exactly

## How to apply

Locate Section 4.4 of the MedPAO paper and its Table 4 ('Radiologist and Clinician evaluation of agent generated reports'), which scores 50 agent-generated MIMIC-CXR report samples on the 5-point rubric defined in Table 3. Record the mean Accuracy and Structure scores per evaluator group: Clinician 4.64 Accuracy and 4.59 Structure; Panel of two Radiologists 4.52 Accuracy and 4.48 Structure. Tabulate the four values in Python with pandas as a CSV with columns for evaluator method, average accuracy score, and average structure score. Validate by checking that the stored values exactly equal the published Table 4 numbers and that every score lies within the 1-5 rubric range.

## Related tools

- **MedPAO** (Agent framework whose final structured MIMIC-CXR report outputs are the object of the expert evaluation being reproduced) — https://github.com/MiRL-IITM/medpao-agent
- **Python** (Runtime for the tabulation script (Python 3.8 or higher per README))
- **pandas** (Used to build and store the score table as a CSV with evaluator-method, accuracy, and structure columns)

## Examples

```
import pandas as pd; df = pd.DataFrame([{'method':'Clinician','avg_accuracy':4.64,'avg_structure':4.59},{'method':'Radiologists','avg_accuracy':4.52,'avg_structure':4.48}]); df.to_csv('table4_scores.csv', index=False)
```

## Evaluation signals

- Stored Clinician average Accuracy equals 4.64 and average Structure equals 4.59 exactly
- Panel of two Radiologists average Accuracy equals 4.52 and average Structure equals 4.48 exactly
- All four stored scores lie within the 1-5 range defined by the Table 3 rubric
- CSV schema contains the expected columns: evaluator method, average accuracy score, average structure score
- Overall mean across evaluator groups is consistent with the abstract's reported average score of 4.52 out of 5

## Limitations

- Scores come from a small human panel (one clinician and a panel of two radiologists) on only 50 MIMIC-CXR report samples, limiting generalizability
- The 4.52/5 figure is an average over evaluator groups; reproducing per-group values requires the full Table 4 breakdown, not just the abstract-level summary

## Evidence

- [results] Table 4 workflow definition: [evidence span withheld — anchor status: not_found]
- [results] Evaluation sample size: [evidence span withheld — anchor status: not_found]
