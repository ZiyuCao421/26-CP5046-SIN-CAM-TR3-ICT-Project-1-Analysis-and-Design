# V3 Stage 1 — Single-Feature Screening Protocol

**Notebook:** `V3/notebooks/01_all_feature_screening.ipynb`
**Configuration mirror:** `V3/configs/screening_config.json`
**Scope:** Stage 1 of the V3 workflow only — the all-candidate single-feature screening pass.

> **Results validity.** Every number produced by this stage is **only valid after a real Colab run
> on the real course data**. The notebook is committed without stored outputs and contains no result
> values. Nothing in this document reports a metric: the document describes the protocol, not its
> outcome.

---

## 1. Purpose

Stage 1 answers one question:

> **Which columns of the delivered mouse-dynamics data carry enough information to be worth putting
> into a feature set, and under which model?**

The answer is produced by evaluating **every eligible column on its own**, under one fixed
protocol, with the project's three models. This gives the later stages (feature-set evaluation and
reporting) an evidence base instead of a preference, and it makes the candidate pool itself
auditable: every column that was removed, and the rule that removed it, is written to a table.

**What this stage is not.** It is not feature selection. A single-feature score measures the
information a column carries *on its own*; it says nothing about what a combination of columns can
achieve, and the best single column is not automatically part of the best set. Stage 2 — the
evaluation of candidate feature sets — is a separate notebook and must not be started until this
stage has been run, reviewed and approved.

---

## 2. Why one common cohort and one split

A ranking is only meaningful if every item on it was measured the same way. Screening each
feature on its own rows cannot produce that ranking, for three reasons:

1. **Different rows.** Each column has its own missing and infinite values, so a per-feature screen
   would build a different sample per column.
2. **Different classes.** A per-feature screen would also drop rare labels per feature, so the
   class space would differ from one feature to the next.
3. **A shared seed is not a shared split.** `train_test_split` with the same `random_state` on
   *different* row sets produces *different* splits. An identical seed guarantees reproducibility,
   **not** comparability. Two features scored on different rows cannot be ordered, whatever the
   seed.

The protocol therefore fixes a single evaluation basis:

| Step | Definition |
| --- | --- |
| Merge first | All matched participant files are concatenated **before** the class space is inspected, because a single file usually carries a single label (a 286-row, 69-column file with `label = 2` is one participant, not the class space). |
| Cohort (complete case) | A row enters the cohort when its label is present, its label class has at least `MIN_CLASS_COUNT` rows in the merged data, and **every** candidate column is finite after numeric coercion (missing and ±inf both count as missing). |
| Row accounting | Rows removed for a missing label, for a class below `MIN_CLASS_COUNT`, and for an incomplete candidate column are counted separately and written to `run_metadata.json`. |
| One split | A **single** stratified `train_test_split` (`stratify = y`, `random_state = 42`, `test_size = 0.20`) is drawn on the cohort. The row positions and a SHA-256 **split fingerprint** are recorded. |
| Same evaluation | Every candidate feature is evaluated on exactly the same training rows, the same test rows and the same class space. `n_train`, `n_test` and `n_classes` are therefore identical on every row of `single_feature_metrics.csv`, and the fingerprint repeats on every row so that a reader can verify it at a glance. |
| Failure policy | If the cohort cannot be built, cannot be split in a stratified way, or does not preserve the class space across the two portions, the run **fails explicitly and produces no primary ranking**. |

**Cohort mode.** `SCREENING_COHORT_MODE` selects the behaviour:

| Value | Status | Meaning |
| --- | --- | --- |
| `complete_case_all_candidates` | **Primary, the default** | The cohort and split above. Rankable. |
| `per_feature` | Exploratory only | Rebuilds a sample and a split per feature. The results are stamped `exploratory / not rankable`, the common-cohort fields are left empty, and the run must not be used for stage-2 selection. |

**What is *not* a per-feature operation.** Per-feature data-quality diagnostics (non-null rate,
numeric-coercion rate, infinite-value count, constant columns) are still reported. They are
diagnostics: they never rebuild a training or test sample and never change the cohort.

---

## 3. Data handling and privacy

The course data is behavioural biometric data (mouse dynamics) with per-participant labels. The
handling rules of this protocol are therefore constraints on the code, not reminders.

| Rule | How it is enforced |
| --- | --- |
| The raw data is read only from the directory the user configures | `DATA_DIR` + `FILE_PATTERN`; nothing is bundled, downloaded or cached |
| The raw data is never written, uploaded, copied or committed | The notebook writes only aggregated tables; there is no upload or export call anywhere in it |
| Results are never written into a Git checkout | `ensure_output_dir()` walks up the path and raises if it finds a `.git` entry, unless `ALLOW_OUTPUT_IN_GIT_REPO` is set to `True` (default `False`). Aggregated artefacts that are to be kept are copied in **by hand** after inspection. |
| Participant-linked values stay out of the outputs | `INCLUDE_SOURCE_FILE_NAMES = False` and `INCLUDE_LABEL_VALUES = False`; the manifest carries counts and a digest instead |
| Local absolute paths stay out of the manifest | `DATA_DIR` and `OUTPUT_DIR` are recorded in `run_metadata.json` as a placeholder, not as a path |
| The label is the data's own, never an artefact of the file name | A file without the label column is **skipped and reported**; no label is ever derived, renamed or overwritten |
| Every output states its own status | Each table carries a `results_validity` column and the manifest carries a `results_validity` field, both set to `only valid after a real Colab run` (or the exploratory stamp) |

**Repository hygiene.** The five artefacts are written to a directory outside the repository. If a
run's aggregated tables are later copied into `V3/results/single_feature/`, the repository should
ignore them (or they should be committed deliberately and documented as the output of a specific
run). Raw data must never enter the repository, in any form.

---

## 4. Protocol

### 4.1 Discovery

`DATA_DIR` is scanned with `FILE_PATTERN`. Only the aggregate number of matching files is printed to the console before
anything is read. An empty match stops the run with an actionable message: the notebook has no
bundled sample data and no synthetic fallback, so it cannot produce a number without the real
files.

### 4.2 Loading and merging

* Each matched file is read with `pd.read_csv`. A file whose columns cannot be typed automatically
  (a stray string inside a numeric column, for example) is re-read as text and coerced later,
  per feature, instead of crashing the run.
* A file **without** the label column is skipped and recorded with its reason. The label is never
  inferred from the file name — that would create a label that the data does not contain.
* The surviving frames are concatenated **before** any class inspection. A single participant file
  usually carries a single label, so the class space is a property of the merged data, not of a
  file.
* The originating file name is kept on each row in a reserved bookkeeping column, which is
  excluded from the candidate pool by rule.
* The label column is used exactly as it appears in the CSV files. Its value space is never
  recoded.

### 4.3 Candidate pool

A column becomes a candidate only if it survives every rule below. The rules are applied in this
fixed order, and the first matching rule wins; the matched pattern is recorded.

| # | Rule | Exclusion reason recorded |
| --- | --- | --- |
| 1 | The label column | `target_label_column` |
| 2 | The source-file bookkeeping column | `source_file_bookkeeping_column` |
| 3 | Listed in `EXCLUDED_COLUMNS` (case-insensitive): the blank-header index column `Unnamed: 0` and the segment identifier `Seg_id` | `config_excluded_column` |
| 4 | Matches `EXCLUDED_COLUMN_PATTERNS` | `config_excluded_pattern` |
| 5 | **Protected family** (`PROTECTED_COLUMN_PATTERNS`) — checked here, **before** the name rules below | not excluded; `protected_by_name_rules = True` |
| 6 | Matches `IDENTIFIER_COLUMN_PATTERNS` (user / file / source / session identifiers) | `identifier_like_column` |
| 7 | Matches `RAW_COORDINATE_PATTERNS` — the raw coordinates `x0`–`x2`, `y0`–`y2` | `raw_coordinate_column` |
| 8 | Matches `TIMESTAMP_PATTERNS` — the raw times `t0`–`t2`, `t` | `timestamp_or_time_column` |
| 9 | Not numeric and not safely coercible (`coercion_rate < MIN_COERCION_RATE`) | `not_numeric_or_safely_coercible` |
| 10 | A single distinct usable value | `constant_column` |
| 11 | Usable rate below `MIN_NON_MISSING_RATE` | `insufficient_non_missing_rate` |

**Protected families.** The behavioural and statistical measures of the dataset are protected from
the *name* rules (6–8):

* `avg_*`, `min_*`, `max_*` — the summary measures;
* `d0`–`d2`, `TV0`–`TV2`, `HV0`–`HV2`, `VV0`–`VV2`, `A0`–`A2`, `AM0`–`AM2`, `AV0`–`AV2`, `J0`–`J2` —
  the per-point families.

They are matched case-insensitively and may carry an optional numeric suffix, so an uppercase name
or a numeric suffix can never remove them by accident. Protection overrides the name rules only:
the type, constant and completeness tests (9–11) still apply to a protected column.

*Usable* means: the value is present, convertible to float and finite. A text column is accepted
as a candidate when at least `MIN_COERCION_RATE` of its non-null values convert to float; the
values that do not convert are dropped at cohort construction and counted, never silently ignored.

The exclusion of raw coordinates, raw times and identifiers is a **principle**, not a measurement:
those columns plausibly encode the recording session rather than the participant, so a model that
used them would partly answer "which recording is this?" rather than "which person is this?". The
cost of the restriction is real and is stated in the limitations of the notebook.

### 4.4 Evaluation on the shared split

For each candidate feature, on the shared cohort and the shared split:

1. Coerce to numeric; replace ±inf with missing.
2. Take the cohort rows — by construction every candidate value is finite there, so no feature
   loses a row at this point.
3. Fit and evaluate the three models on the shared training rows, and score them on the shared
   test rows: accuracy, macro precision, macro recall and macro F1, with the shared sample sizes
   and class counts.

The three models are fixed by the protocol:

| Model | Configuration |
| --- | --- |
| `DummyClassifier_most_frequent` | `strategy="most_frequent"` — the majority-class floor |
| `RBF_SVC_scaled` | `Pipeline([StandardScaler(), SVC(C=1.0, gamma="scale", class_weight="balanced")])` — the scaler is inside the pipeline and therefore fitted on **training rows only** |
| `RandomForest_balanced` | `n_estimators=200`, `random_state=42`, `class_weight="balanced"` |

**Constraints on the protocol.** No hyper-parameter search is performed. The held-out portion is
never used to select a feature, a model or a threshold. The test portion is read once per feature
and only to compute the reported metrics.

### 4.5 Failure handling

Every stage is defensive. A problem is recorded and the run continues with the next column or
model; it never aborts the screening of the remaining candidates.

| Failure | Recorded as | Effect |
| --- | --- | --- |
| The common cohort cannot be built or split | `common_cohort_failed` | **No primary ranking is produced**; the manifest records `run_status = failed - no primary ranking produced` |
| Column cannot be converted | `coercion_failed` | Feature skipped |
| Too few usable rows | `insufficient_usable_rows` | Feature skipped |
| Fewer than two classes survive `MIN_CLASS_COUNT` | `insufficient_classes_after_min_class_count` | Feature skipped |
| The stratified split raises | `split_failed` | Feature skipped |
| The model raises on fit or predict | `model_fit_or_predict_failed` | That model only is skipped for that feature |
| Missing or infinite values | counted per feature in the metrics table | Rows are dropped at cohort construction, not fatal |
| Constant column | `constant_column` in the exclusion table | Column excluded before screening |
| A file without the label column | `missing_label_column` in the run manifest | File skipped, no label inferred |

---

## 5. Outputs

All five files are written to `OUTPUT_DIR` and are aggregated: no raw row, no participant value,
no participant-linked file name and no local absolute path is written.

| File | Content |
| --- | --- |
| `feature_inventory.csv` | Every column: type, counts, usable rate, screening status, protection flag, exclusion reason, matched rule |
| `excluded_features.csv` | The rejected columns only, with the exact rule that rejected them — the audit trail |
| `single_feature_metrics.csv` | One row per (feature × model): accuracy, macro precision, macro recall, macro F1, the shared split, and the per-feature quality diagnostics |
| `single_feature_failures.csv` | One row per feature or feature × model that could not be evaluated, with stage, reason code, message and timestamp |
| `run_metadata.json` | The manifest: the required cohort fields, the cohort definition and accounting, the split fingerprint, the configuration snapshot, the environment, the data footprint, the model roster, the failure summary and a digest of each written table |

**Required fields.** `single_feature_metrics.csv` and `run_metadata.json` both carry
`cohort_mode`, `n_rows_before_cohort`, `n_rows_common_cohort`, `n_train_common`, `n_test_common`,
`n_classes_common`, `split_fingerprint` and `results_validity`. In the primary mode every row of
the metrics table shows the same `n_train`, `n_test` and `n_classes_common`, and the same
`split_fingerprint` — that is the evidence that the comparison is like for like.

The column-by-column schema of each table is documented in the notebook itself (the *Output
reference* section), so the artefact can be read without reading the code.

---

## 6. Reproducibility

* Every seed is fixed: `RANDOM_STATE = 42` for the split and the forest.
* The configuration used in a run is written into `run_metadata.json`, together with the library
  versions and the SHA-256 digest of the input file set (a digest, not a file list, so the
  participant-linked file names stay out of the record).
* The split fingerprint identifies the shared split without publishing it.
* `V3/configs/screening_config.json` mirrors the notebook configuration with a one-line reason for
  each threshold. The notebook cell is the authority at run time; the two must be updated in the
  same commit if they ever disagree.

---

## 7. Limitations carried into stage 2

1. **The complete-case cohort is smaller than the merged data**, and the rows it loses are not
   missing at random — they tend to be the harder or shorter movements. The ranking therefore
   describes the complete-case rows. The cohort accounting states how many rows were lost and
   why.
2. **Segment-level random split.** Rows are three-point segments of one continuous recording, so
   neighbouring segments are not independent and a random split is optimistic as a generalisation
   estimate. A session-level split would be stricter; no session identifier exists in the delivered
   data and none is invented.
3. **Single-feature scores understate combinations.** Several weak columns can form a strong
   model, and one strong column can be an artefact of the split. Stage 2 must evaluate sets.
4. **One split, one seed.** The stage-1 numbers do not quantify split-to-split variation.
5. **The exclusion of raw coordinates and raw times is a deliberate restriction**, and the
   resulting scores are therefore a statement about behavioural information, not the maximum
   achievable accuracy on the delivered data.
6. **Small samples and few labels.** Macro metrics move when one class is small; the class counts
   in the manifest are the context in which any number must be read.

---

## 8. Stage gate

Stage 2 may begin only when all of the following hold.

- [ ] The notebook has been executed in Colab against the real course data.
- [ ] `run_metadata.json` exists and records the configuration, environment, cohort accounting and
      split fingerprint of that run.
- [ ] `excluded_features.csv` has been reviewed — no useful column was removed by accident, the
      protected families are present as candidates, and `Unnamed: 0`, `x0`–`x2`, `y0`–`y2`,
      `t0`–`t2` and `Seg_id` are excluded.
- [ ] `single_feature_metrics.csv` shows a single `split_fingerprint` and identical `n_train` /
      `n_test` / `n_classes_common` on every row.
- [ ] `single_feature_failures.csv` has been reviewed — no systematic failure (a whole column
      family failing, a model failing everywhere) is hiding behind a short table.
- [ ] The majority-class floor from `DummyClassifier_most_frequent` is known and will be used as the
      reference for every later claim.
- [ ] The supervisor has reviewed this stage's output.

**Stage 1 is the only deliverable of this phase.** The feature-set notebook is deliberately not
created here.
