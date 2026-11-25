# Example input tables

**These tables are synthetic. They contain no real patients.** They show the input format that `00_extract_outcomes.ipynb`
expects. To run the pipeline on them, set `USE_EXAMPLE_DATA = True` in the notebook. Outputs go to
`outputs/examples/`.

Both CSV and parquet are accepted. If your columns have other names, map them in `COLUMN_MAP`
(e.g. `{"empi": "patient_id"}`).

## `cohort.csv`: one row per patient and index exam

This table is the denominator. Patients in it with no matching codes are counted as outcome-negative.

| Column | Type | Description |
|---|---|---|
| `patient_id` | integer or string | Patient identifier |
| `accession_id` | integer or string | Index exam identifier. All `days_to_*` columns are measured from this exam. |
| `days_to_follup` | integer | Days from the index exam to the end of follow-up (last contact). Carried through to the output for time-to-event analysis. |

## `icd.csv`: one row per diagnosis code occurrence

| Column | Type | Description |
|---|---|---|
| `patient_id` | integer or string | Must match `cohort.patient_id` |
| `accession_id` | integer or string | Must match `cohort.accession_id` |
| `icd_code` | string | ICD-9 or ICD-10 code **with the dot**, e.g. `I21.4`, `410.71` |
| `days_to_icd` | number | Days from the index exam to the code. Negative means before the index exam. May be empty. |

Use day offsets, not dates. If your source has dates, compute `code_date - index_date` in days before running the
notebook, and leave the dates out of the tables. Exact dates are identifying; offsets from an index exam are not.

## What each example patient demonstrates

Results use the `dapamede2026_ehj_bac` definitions.

| patient_id | Codes (day) | Result |
|---|---|---|
| 1001 | `I21.4` (120, 125), `I10` | AMI incident, first on day 120 |
| 1002 | `410.71` (-800), `I21.9` (900) | AMI prevalent (ICD-9) and incident (ICD-10) |
| 1003 | `I50.23` (0) | HF incident: day 0 counts as incident |
| 1004 | `428.0` (200), `I50.9` (400) | No HF: these codes are not in the definition (shown in the QA near-miss table) |
| 1005 | `434.91` (-2000), `I63.512` (1500) | Stroke prevalent, and incident through the `I63.` prefix |
| 1006 | `I21.A1` (300) | No AMI: Type 2 MI is not in the definition (near miss) |
| 1007 | `I21.01` (30), `I63.9` (31), `I50.21` (45) | Incident AMI, stroke and HF |
| 1008 | `I50.43` (empty day) | HF `any` is true, but neither prevalent nor incident because the timing is unknown |
| 1009 | `I10`, `Z23`, `E11.9`, `Unspecified Diagnosis Code` | No outcomes: unrelated codes only |
| 1010 | *(no ICD rows)* | Kept in the output, all outcomes negative |
| 1011 | `I21.4` (10) | Dropped: not in the cohort table |
| 1012 | `I63.9` (-100), `433.11` (700) | Stroke prevalent and incident |

Expected outcome counts (11 cohort patients):

| outcome | any | prevalent | incident |
|---|---|---|---|
| ami | 3 | 1 | 3 |
| hf | 3 | 0 | 2 |
| stroke | 3 | 2 | 3 |
