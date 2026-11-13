# V3 Phase Report — Stage 1 (Single-Feature Screening) and Stage 2 (Results, Visualisation and Candidate Shortlist)

> **Status: completed, and reproducible from the artefacts in this folder.**
> Every number below was produced by a real Google Colab run on the real course data. The run is
> identified by the split fingerprint and the input digests in
> [`stage2_candidate_manifest.json`](../results/summaries/stage2_candidate_manifest.json). No
> figure or table in this report was hand-edited, and no value was estimated.

**Project:** multi-class mouse-dynamics user identification (CP5046 Information Security, V3
workflow).
**Report generated:** 2026-11-13.
**Chinese version:** [`02_phase1_stage2_report_zh.md`](02_phase1_stage2_report_zh.md).

---

## 1. What this phase delivers

| Stage | Notebook | Purpose | Artefacts |
| --- | --- | --- | --- |
| 1 | [`01_all_feature_screening.ipynb`](../notebooks/01_all_feature_screening.ipynb) | Build an auditable candidate pool from the course CSVs and screen **every** candidate feature on its own under one fixed protocol | `feature_inventory.csv`, `excluded_features.csv`, `single_feature_metrics.csv`, `single_feature_failures.csv`, `run_metadata.json` |
| 2 | [`02_single_feature_visualisation.ipynb`](../notebooks/02_single_feature_visualisation.ipynb) | Validate the stage-1 outputs, visualise them, and produce a deterministic short list of candidate features | `selected_features.csv`, `selection_summary.csv`, `stage2_candidate_manifest.json`, `correlation_next_step.md`, `figures/` |

Stage 2 **does not train a model and does not read the raw course CSV**. It consumes the aggregate
outputs of stage 1 only. Feature-set construction (stage 3) has not been started.

**Configuration of record:** [`../configs/screening_config.json`](../configs/screening_config.json).
**Protocol of record:** [`01_single_feature_screening_protocol.md`](01_single_feature_screening_protocol.md).

---

## 2. Data handling

The course data is behavioural biometric data (mouse dynamics) carrying per-participant labels.
The handling rules were constraints on the code, not reminders:

* the raw CSV files are read from a configured Drive folder at run time and are **never written,
  uploaded, copied or committed**;
* the label is read **from inside the CSV files only** — never derived from a file name, never
  renamed, never overwritten; a file without the label column is skipped and reported;
* participant file names, label values and local absolute paths are **withheld from every
  artefact**; the manifest carries counts and SHA-256 digests instead;
* stage 1 refuses to write its outputs into a Git checkout; stage 2 writes only aggregate tables
  and figures;
* every artefact carries the stamp `only valid after a real Colab run`.

What is committed here is aggregate: column names, exclusion reasons, per-feature metrics,
counts and figures. **No raw row, no raw value, no participant value.**

---

## 3. The protocol that produced these numbers

### 3.1 One cohort, one split

A feature can only be ranked against another feature if both were measured on the same data. The
stage-1 protocol therefore builds **one global cohort** and draws **one single stratified split**:

1. all participant files are merged first, because a single file usually carries a single label;
2. the cohort keeps the rows whose label is present, whose label class has at least
   `MIN_CLASS_COUNT = 5` rows, and for which **every** candidate column is finite after numeric
   coercion (missing and ±inf both count as missing);
3. **one** stratified `train_test_split` (`stratify = y`, `random_state = 42`, `test_size = 0.20`)
   is drawn on that cohort, and its fingerprint is recorded;
4. every candidate feature is evaluated on exactly the same training rows, the same test rows and
   the same class space.

An identical seed on *different* rows does **not** produce the same split, so a per-feature
screen could not have been ranked. That is why the earlier per-feature design was replaced.

### 3.2 Models

| Model | Configuration | Role |
| --- | --- | --- |
| `DummyClassifier_most_frequent` | `strategy="most_frequent"` | Majority-class **reference floor**; never ranked |
| `RBF_SVC_scaled` | `Pipeline([StandardScaler(), SVC(C=1.0, gamma="scale", class_weight="balanced")])` | Margin-based baseline; the scaler is fitted on training rows only |
| `RandomForest_balanced` | `n_estimators=200`, `random_state=42`, `class_weight="balanced"` | Tree-based comparison |

No hyper-parameter search was performed, and the held-out portion was never used to fit or tune a
model. Stage 2 reuses its already-recorded metrics only to create a descriptive, provisional short
list; it is not a new estimate of generalisation.

### 3.3 Column rules

Removed: the blank-header index column (`Unnamed: 0`), `Seg_id`, the raw per-point coordinates
(`x0`–`x2`, `y0`–`y2`) and the raw per-point times (`t0`–`t2`) — 13 columns in total, every one
of them with a recorded reason. Kept as candidates: the behavioural and statistical families
`avg_*`, `min_*`, `max_*` and the per-point families `d`, `TV`, `HV`, `VV`, `A`, `AM`, `AV`, `J`.
The protected families are checked *before* the name rules, so an uppercase name or a numeric
suffix can never remove them by accident.

---

## 4. Results of this run

### 4.1 Cohort and candidate pool

| Quantity | Value |
| --- | ---: |
| Rows before cohort filtering | 2 002 |
| Rows removed by cohort filtering in this run | 0 |
| Rows in the common cohort | 2 002 |
| Training rows / test rows | 1 601 / 401 |
| Classes in the cohort | 9 |
| Columns in the merged data | 70 |
| Candidate features screened | 57 |
| Columns excluded | 13 |
| Failed screenings | 0 |
| Split fingerprint | `sha256:297284c375de6cb3535fbd2268907915` |

Exclusion reasons, all 13 accounted for:

| Reason | Columns |
| --- | ---: |
| raw coordinate column | 6 |
| timestamp or time column | 3 |
| config excluded column | 2 |
| source file bookkeeping column | 1 |
| target label column | 1 |

![Audit of the candidate pool](../results/summaries/figures/candidate_pool_audit.png)

### 4.2 Reference points

The measured majority-class baseline is the only baseline reported as a Macro-F1 reference here:

| Reference | Macro F1 | What it means |
| --- | ---: | --- |
| Majority-class floor (dummy) | 0.0342 | The measured score of always predicting the largest class. It is **low because macro averaging punishes it**: it scores 1 on one class and 0 on the other eight. |

**49 of 57 features beat the dummy floor** under the RBF SVC; all 57 do so under the Random
Forest. Because the dummy's Macro-F1 is a weak floor, clearing it alone is not evidence that a
feature will generalise.

| Model | Median macro F1 | Best single feature |
| --- | ---: | --- |
| RBF SVC | 0.0794 | 0.2629 (`avg_t`) |
| Random Forest | 0.1746 | 0.3048 (`min_y`) |

`1 / 9` is a class-count heuristic, not a valid expected Macro-F1 baseline for this imbalanced
multi-class sample. It is therefore not used to label features as “above chance”. A calibrated
chance reference would require an explicitly defined random-prediction procedure and the class
prevalences.

![Distribution of macro F1 per model](../results/summaries/figures/distribution_macro_f1_by_model.png)

### 4.3 The candidate shortlist

The shortlist is the first 12 features in the combined order: mean of the two models' ranks →
the larger of the two macro F1 values (descending) → feature name (ascending). All twelve beat the
majority-class floor under **both** models.

| # | Feature | Rank (SVC) | Rank (RF) | Mean rank | Macro F1 (SVC) | Macro F1 (RF) |
| ---: | --- | ---: | ---: | ---: | ---: | ---: |
| 1 | `max_y` | 4 | 2 | 3.0 | 0.2216 | 0.2981 |
| 2 | `min_y` | 6 | 1 | 3.5 | 0.2174 | 0.3048 |
| 3 | `avg_y` | 5 | 6 | 5.5 | 0.2184 | 0.2511 |
| 4 | `min_t` | 2 | 10 | 6.0 | 0.2591 | 0.2347 |
| 5 | `max_t` | 3 | 12 | 7.5 | 0.2579 | 0.2307 |
| 6 | `avg_t` | 1 | 16 | 8.5 | 0.2629 | 0.2194 |
| 7 | `min_x` | 7 | 13 | 10.0 | 0.2004 | 0.2294 |
| 8 | `VV2` | 17 | 5 | 11.0 | 0.1050 | 0.2585 |
| 9 | `max_x` | 9 | 19 | 14.0 | 0.1797 | 0.2129 |
| 10 | `avg_x` | 8 | 20 | 14.0 | 0.1999 | 0.1969 |
| 11 | `VV0` | 25 | 4 | 14.5 | 0.0819 | 0.2708 |
| 12 | `min_hv` | 21 | 9 | 15.0 | 0.0906 | 0.2375 |

Full table with every candidate, the ranks, the dummy comparison and the explicit selection
reason for each row: [`selected_features.csv`](../results/summaries/selected_features.csv).
Per-model Top-K lists: [`selection_summary.csv`](../results/summaries/selection_summary.csv).

![Top-12 by RBF SVC](../results/summaries/figures/topk_macro_f1_RBF_SVC_scaled.png)

![Top-12 by Random Forest](../results/summaries/figures/topk_macro_f1_RandomForest_balanced.png)

![Top-12 by DummyClassifier (the reference floor)](../results/summaries/figures/topk_macro_f1_DummyClassifier_most_frequent.png)

---

## 5. What the results actually say

Three findings are worth stating plainly, because they change what the next stage should do.

### 5.1 The two models do not agree about what matters

The Spearman correlation between the two models' **performance rankings** is **ρ = 0.322**. Only
**five** features appear in both models' Top-12 (`min_y`, `max_y`, `avg_y`, `min_t`, `max_t`);
inside the shortlist the median gap between the two ranks is 9.5 positions and the largest is 21
(`VV0`: rank 25 under the SVC, rank 4 under the forest).

The two models are not disagreeing about noise; they are picking out **different families**:

* the **RBF SVC** prefers the timing aggregates — `avg_t` (rank 1), `min_t` (2), `max_t` (3);
* the **Random Forest** prefers the velocity and position summaries — `min_y` (1), `max_y` (2),
  `max_vv` (3), `VV0` (4), `VV2` (5).

The mean-rank consensus therefore mixes two different notions of information, and a shortlist
built from it is not obviously the shortlist either model would choose. This is a property of the
data and the learner, not a defect in the ranking, but stage 3 must decide which view it trusts.

![Scatter of the two models](../results/summaries/figures/scatter_macro_f1_svc_vs_random_forest.png)

![Rank agreement between the two models](../results/summaries/figures/rank_agreement_svc_vs_random_forest.png)

> **This is rank agreement, not feature correlation.** It measures how similarly the two models
> *order* the features. The stage-1 outputs contain no row-level values, so a feature-to-feature
> Pearson or Spearman correlation cannot be computed from them, and none was fabricated. See
> `correlation_next_step.md` for the step that must come first.

### 5.2 Nine of the twelve shortlisted features are absolute position or time aggregates

This is the most important caveat in this report.

The protocol excluded the raw per-point coordinates (`x0`–`x2`, `y0`–`y2`) and the raw per-point
times (`t0`–`t2`) because they plausibly encode **the recording** rather than **the person**. But
the shortlist is led by `max_y`, `min_y`, `avg_y`, `min_x`, `max_x`, `avg_x`, `min_t`, `max_t`,
`avg_t` — the **aggregate counterparts of exactly those quantities**. `avg_t` is the mean of the
timestamps that were excluded; `min_y` and `max_y` are the extremes of the excluded coordinates.

The `avg_*` / `min_*` / `max_*` families were protected because they are *usually* behavioural
summaries. In this dataset, several high-scoring members are summaries of **absolute screen
position and absolute acquisition time**. This creates a recording or session-leakage *risk*: where
on the screen and when in a recording a segment occurred may be easier to recognise than a person's
movement behaviour. The present random split cannot establish that this risk is the cause of the
scores.

The highest-scoring single features may therefore be answering “which recording is this?” rather
than “which person is this?”. If neighbouring segments from the same recording occur on both sides
of the random split, this could inflate the score. Whether it does so, and whether performance
would persist in a new session or for a new participant, require a grouped or session-aware
evaluation; they are not established by the current run.

**Recommendation for the supervisor.** Treat the current shortlist as *provisional*. Before stage 3,
re-run the screening with a restricted candidate pool that keeps only position-invariant and
time-invariant measures (the `d`, `TV`, `HV`, `VV`, `A`, `AM`, `AV`, `J` families and any summary
that is not an absolute coordinate or timestamp aggregate), and compare the two rankings. If the
restricted pool produces a much weaker screen, that is the honest result for this dataset, and it
is the one to report.

### 5.3 The majority-class floor is a weak reference

The dummy classifier's Macro-F1 of 0.0342 is a deliberately weak reference, so “beats the dummy”
is satisfied by 49 of 57 features under the SVC and all 57 under the Random Forest. The
`beats_dummy_rbf` / `beats_dummy_rf` columns remain useful as a sanity check — all twelve
shortlisted features clear the floor in both models — but they are not evidence of generalisation.

---

## 6. Limitations

1. **One split, one seed.** Everything rests on a single stratified 80/20 split. The order of
   two close features should be treated as arbitrary; the score of a feature near the floor should
   be treated as uncertain.
2. **Segment-level random split.** Rows are three-point segments and may not be independent if
   nearby segments from a recording occur on both sides of the split. A random split can then be
   optimistic as a generalisation estimate. A session/group-aware evaluation requires a reliable
   grouping variable and has not been run here.
3. **Complete-case rule.** The rule retained all 2 002 rows in this run: zero rows were removed
   for missing labels, low-support classes or incomplete candidates. Complete-case bias is thus
   not observed in this run, but remains a consideration for later runs with missing values.
4. **Absolute performance is modest.** The best single feature reaches 0.3048 macro F1 under the
   forest and 0.2629 under the SVC. A single feature is weak evidence for a feature *set*; several
   weak features can combine into a strong model, and one strong feature can be an artefact of the
   split.
5. **The shortlist is descriptive.** It reuses stage-1 held-out metrics. It is not a re-estimate
   of generalisation and not a final selection.
6. **No feature-to-feature correlation exists yet** — it needs the next notebook and the raw data
   in a private environment.
7. **The strongest features may measure the recording, not the person** (§5.2). This limitation
   currently dominates all the others.

---

## 7. Next steps

1. **Supervisor review of this report**, in particular of §5.2.
2. **Restricted-pool re-run** (stage 1 with position- and time-invariant candidates only) to
   establish how much of the current signal survives when the absolute position and time
   aggregates are removed.
3. **Correlation notebook** (its scope is already specified in `correlation_next_step.md`): load
   the raw CSV in Colab only, rebuild the same complete-case cohort, and compute Pearson and
   Spearman correlations over the shortlisted columns as a **redundancy diagnostic**. Never
   commit raw data.
4. **Stage 3 — feature-set evaluation**: candidate sets under one protocol, with the same cohort
   and split, and the chance reference reported alongside the dummy floor.
5. **Consider a session-aware evaluation** if any recording identifier becomes available.

---

## 8. Reproducibility and provenance

| Item | Value |
| --- | --- |
| Run timestamp (UTC) | 2026-11-13T11:25:18+00:00 |
| Stage-2 notebook | `V3/notebooks/02_single_feature_visualisation.ipynb` |
| Seed | `RANDOM_STATE = 42` |
| Split fingerprint | `sha256:297284c375de6cb3535fbd2268907915` |
| Cohort mode | `complete_case_all_candidates` |
| Stage-1 run status | `completed` |
| Input-contract checks | 19 / 19 passed |
| Environment | Python 3.13.16, numpy 2.4.6, pandas 2.2.3, matplotlib 3.10.0 |

SHA-256 of the stage-1 inputs consumed by this run (also in
[`stage2_candidate_manifest.json`](../results/summaries/stage2_candidate_manifest.json)):

| File | SHA-256 |
| --- | --- |
| `single_feature_metrics.csv` | `5285c6dca812e64b427eab64a8cf3c5909f6e4321088887120b5c5b750709abf` |
| `feature_inventory.csv` | `7f41e11665c95fffe61514fbb8cc8826dee7c18c63e07056a084da5ba31bb362` |
| `excluded_features.csv` | `67c0261810043620d030ac8aa6dcf99c1c5f03c3cd65ee976abc7345ca9c1ddf` |
| `single_feature_failures.csv` | `8537f46d6275be72cd3a5a5c5f5240a5e1d31154eb7c030246081d25518f432f` |
| `run_metadata.json` | `c8d1d04699810d85843db8c97533fc821e170b48db8e36158feb87d0557cbf5a` |

Anyone re-running stage 1 on the same data version and comparing the digests can confirm that
this report describes that run.

### How to reproduce

1. Run `01_all_feature_screening.ipynb` in Colab against the course data folder.
2. Run `02_single_feature_visualisation.ipynb` in Colab with `INPUT_DIR` pointing at the stage-1
   output folder.
3. Compare the `split_fingerprint` and the five input digests with the tables above.

---

## 9. Repository layout of this phase

```text
V3/
├── notebooks/
│   ├── 01_all_feature_screening.ipynb        # stage 1 — screening
│   └── 02_single_feature_visualisation.ipynb # stage 2 — results and shortlist
├── configs/
│   └── screening_config.json                  # configuration of record
├── docs/
│   ├── 01_single_feature_screening_protocol.md
│   ├── 02_phase1_stage2_report.md            # this report
│   └── 02_phase1_stage2_report_zh.md         # 中文翻译版
└── results/
    └── summaries/
        ├── selected_features.csv
        ├── selection_summary.csv
        ├── stage2_candidate_manifest.json
        └── figures/*.png
```

---

## 10. Statement of validity

Every figure and table in this report comes from the run identified in §8 and is **only valid
after a real Colab run on the real course data**. The notebooks are committed without stored
outputs, so nothing in the repository can be mistaken for evidence of an experiment that has not
been run. No raw data, no label values, no participant-linked file names and no local absolute
paths appear in any committed artefact.

*Mouse dynamics are behavioural biometric data. Store and share them responsibly, and follow the
course and institutional handling requirements.*
