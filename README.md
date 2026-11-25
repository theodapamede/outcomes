# Outcomes

Extracts outcome cohorts from ICD code definitions. See `00_extract_outcomes.ipynb`.

## Data privacy

This is a public repository. **No patient data is committed, including anonymized data.** Only code and code
definitions (`definitions/**/*.txt`) belong here.

- `.gitignore` excludes `outputs/` and all data formats (`*.parquet`, `*.csv`, ...).
- Notebook outputs are stripped on commit by [nbstripout](https://github.com/kynan/nbstripout). The filter is
  registered in `.git/config`, which is not versioned, so run this once after cloning:

  ```bash
  uv sync
  uv run nbstripout --install --attributes .gitattributes
  ```

## Input tables

The pipeline needs a cohort table (one row per patient and index exam) and an ICD table (one row per code
occurrence, with days relative to the index exam). See [`examples/`](examples/) for synthetic sample tables, the
column descriptions, and the expected results. Set `USE_EXAMPLE_DATA = True` in the notebook to try it out.

## Adding outcome definitions

Add `.txt` files to `definitions/<set_name>/`, named `<outcome>_<suffix>.txt` (e.g. `afib_icd9.txt`,
`afib_icd10.txt`). One code per line: `I21.4` matches exactly, `I63.` or `I63*` matches the prefix.
