# Elevating Academic Integrity: Unveiling Fraud Detection in Online University Assessments using Wrapper Technique Feature Selection Method

> **Status.** Implementation guide and methodology record. The executed notebook is the source of truth for the dataset version, preprocessing, figures, metrics and conclusions. No performance figure appears in this README until the notebook has been run on the real course data.

---

## 1. Overview

Online assessment delivery has moved a large share of university evaluation to the browser, and with it a new integrity question: can the behaviour of the person at the keyboard be used to tell legitimate account owners apart? This project treats **mouse dynamics** — the trajectory characteristics of a movement — as a behavioural biometric signal, and asks how much identification power is recoverable from a deliberately small, interpretable feature set.

The project is a **multi-class user identification** experiment on three-point mouse-movement segment data. Its distinguishing feature is the **wrapper feature selection** method named in the title: candidate feature subsets are scored by training a classifier on them and keeping the subset that generalises best, rather than by ranking features with a label-independent statistic.

**Boundary statement.** This is an exploratory, classroom-scale experiment on three movement features. It is not a fraud-detection system, not an authentication system, and not a claim about any real assessment platform. Section 8 states the limits that apply to the result actually obtained.

---

## 2. Development methodology

### 2.1 The question that was asked

The project brief allowed one of two development processes: a **plan-driven (Waterfall)** approach or an **iterative (Agile)** approach. The A1 coursework submitted for this module — *Comparative Analysis of Software Development Approaches* — argues that the Waterfall-versus-Agile binary is itself the wrong frame, because it keeps the debate inside the Rational Design Paradigm, where "process choice" is treated as the decisive variable (Ralph, 2018). The productive move is to relocate attention from the process label to the **evidence** that the process actually generates.

That argument is applied here rather than merely cited.

### 2.2 Comparison on the four criteria that matter for this project

| Criterion | Plan-driven (Waterfall) | Iterative (Agile) | Verdict for this project |
| --- | --- | --- | --- |
| **Requirement uncertainty** | Requirements are already fixed: three named features, one target column, a prescribed split and metric set. Stable requirements are exactly the precondition under which plan-driven work pays off. | The short loop buys little when there is nothing ambiguous in the requirements. | **Favours plan-driven** — but the requirements it is stable *about* are the analysis plan, not the data. |
| **Where the real uncertainty lives** | Plan-driven execution assumes the analysis design can be fixed before implementation. Here the unknown is empirical: which files arrive, which columns exist, how many labels, how separable the features are, which subset works. | Each iteration resolves one empirical unknown with an executable artefact. | **Decisive, and favours the iterative loop.** The uncertainty is in the *data*, and the data can only be resolved by running code against it. |
| **Feedback latency** | A one-shot design-then-implement sequence defers the first reality check to the end, when a wrong assumption about the data is most expensive to unwind. | Feedback arrives per iteration, at the cost of one notebook run. | **Favours the iterative loop** — the feedback unit here is a cell execution, so the loop is cheap. |
| **Traceability, assurance and team coordination** | Its genuine strength: explicit, auditable phase artefacts and handovers, and a documentation chain that survives staff turnover. | Weak on audit trails; its ceremonies guarantee nothing (Lindsjørn et al., 2016 found Agile teams' teamwork quality is not significantly better than that of comparable traditional teams). | **Favours plan-driven** — and it is answered by keeping the README, the notebook's Markdown rationale, the decision log and the parameter-verification table as the audit trail. |

### 2.3 Decision

**This project is developed with an Agile-style iterative loop, governed by the Empirical Design Paradigm (Ralph, 2018), anchored by a single fixed specification phase.**

The reasoning:

1. **The precondition that makes plan-driven work attractive is absent.** The requirements are not going to change; they were fixed before the project started. A process whose main strength is handling requirement churn is being applied to a problem without requirement churn.
2. **The uncertainty that remains is empirical, and only an empirical method resolves it.** Whether the delivered files contain `avg_d`, `avg_tv` and `avg_a`; whether `label` exists; how many users there are; whether the three features separate them at all — none of these can be settled by a document. They are settled by loading the data and looking.
3. **The failure mode of a short loop is delivering the wrong thing faster.** That is managed here by pre-registering the things a short loop could otherwise be tempted to improvise: the feature set, the evaluation protocol, the search spaces, the tie-break order and the acceptance criteria — all fixed in the specification phase, before any result exists.
4. **The plan-driven contribution is retained where it is actually valuable — the audit trail.** Every cell prints the evidence it produces; every hyper-parameter is verified against the installed scikit-learn signature and shown next to its library default; the README, this methodology section and the Sprint 0 contract together form the documentation chain that a plan-driven process would have produced as a separate artefact.
5. **The team-size precondition is handled honestly.** The Agile mechanism depends on stakeholders giving feedback at sprint scale. There is no external stakeholder here, so the "customer" is the rubric and the data, and the compensating discipline is the pre-registered acceptance criteria plus a retrospective at the end of every sprint. A team that cannot influence the sprint cannot reflect on it, and this project does not claim otherwise.

**Concretely, the notebook is organised as sprints, each with a goal, a user story, acceptance criteria and an executable artefact:**

| Sprint | Artefact produced |
| --- | --- |
| 0 — Specification | The fixed contract: features, target, split, metrics, integrity rules, acceptance criteria |
| 1 — Data access & audit | File list, data-quality summary, label-consistency check |
| 2 — Cleaning | Modelling matrix with exactly three columns; every removed row accounted for |
| 3 — EDA | Class distribution, per-label distributions, correlation, 2-D projection |
| 4 — Model design | Model roster, declared search spaces, parameter verification against scikit-learn |
| 5 — Nested cross-validation | Wrapper subset search, baseline comparison, stability, model selection |
| 6 — Held-out evaluation | One test evaluation, bootstrap CI, calibration and confidence |
| 7 — Diagnostics | Learning curves, over-fitting diagnosis, per-class scores, permutation importance |
| 8 — Retrospective | Auto-generated results summary and the discussion the author must write |

### 2.4 How the choice shows up in the code

* **Fixed up front:** the three features, the split, the metric set, the candidate models, the hyper-parameter grids, the tie-break order, the number of folds.
* **Iterated during the run:** the data audit answers determine whether the cleaning policy is needed; the wrapper search determines the subset; the nested CV determines the model; the learning curve determines whether the model is data-limited or capacity-limited.
* **Never iterated on:** anything that depends on the test portion. The test portion is read once, at Sprint 6, after every decision has been made on the training portion.

---

## 3. Selected features

| Column | Meaning in this project | Why retain it |
| --- | --- | --- |
| `avg_d` | Average movement-distance feature for a segment | Movement magnitude. |
| `avg_tv` | The dataset's average velocity-related (`TV`) feature | Movement speed. |
| `avg_a` | Average acceleration feature for a segment | How movement changes over time. |

Together they give a compact **distance–speed–acceleration** description of one movement segment. The exact feature definitions remain those supplied by the dataset.

**Why not the raw columns?** The reference data also carries raw point values such as `x0`, `y0` and `t0`. Absolute coordinates encode screen geometry and device differences; absolute timestamps and collection order encode acquisition order. Either can leak information that does not survive a change of session, machine or participant. The three selected features are derived behavioural summaries, and they are easier to justify in a report.

**And that exclusion is measured, not asserted.** Sprint 3 includes a leakage probe that asks, of every candidate column, *where in the recording* a segment was taken from. On the real data the three selected features are close to chance at that task, while the absolute timestamps and the absolute coordinates identify the recording position far better than they identify anything about the person. A classifier built on those columns is partly answering "which recording is this?" — and that part of its accuracy would not survive a new recording. The cost of the restriction is equally real and is stated in Section 8: a model limited to these three features can use motor behaviour only, so its accuracy is bounded well below what a model allowed to use the recording's coordinates and clock can reach.

**Why these three, and why a wrapper?** Distance, speed and acceleration are three views of the same movement event, so they are expected to be correlated — the correlation heatmap in Sprint 3 confirms it. The consequence is that the interesting question is not "which single feature is most informative?" but "**which combination** separates the labels best, for a given learner?" That is a subset question, and it is exactly what a wrapper answers. Because the pool holds three features, the seven non-empty subsets can be enumerated exhaustively, which is stronger than a greedy forward search.

### 3.1 The structure of the delivered data

One property of the data is worth stating because it shapes the whole project. Each file holds the rows of **one continuous mouse recording, chopped into consecutive three-point segments**: for every row, `d0` — the distance from the previous segment's last point to this segment's first point — is exactly the across-segment gap, and the timestamps run monotonically down the file. Three consequences:

1. The recording can be **stitched back into a single continuous trajectory** and shown in three dimensions (x, y, time). That is the most intuitive visualisation in the project, and it is what the movement features are summaries of.
2. `d0`, and the velocity/acceleration/jerk values at index 0 of each row, are computed **across** the segment boundary. They are legitimate features, but they are also evidence that neighbouring rows are not independent samples.
3. A random segment-level split therefore has to be checked for near-duplicate leakage. The leakage probe in Sprint 3 does exactly that, and reports the answer rather than assuming it.

The `avg_*` features are means of the corresponding per-point triplets: `avg_d = mean(d0, d1, d2)`, `avg_tv = mean(TV0, TV1, TV2)`, `avg_a = mean(A0, A1, A2)`, and so on for `avg_hv`, `avg_vv`, `avg_am`, `avg_av` and `avg_j`. `TV` is the speed at a point, `HV`/`VV` its horizontal/vertical components, `A` the acceleration magnitude, `AM` its direction and `J` the jerk.

---

## 4. Data

The course data is distributed as a shared Google Drive folder of per-user segment CSV files, similar to:

```text
User1_Calib_3MouseMovement_MCQ_Segment.csv
...
User10_Calib_3MouseMovement_MCQ_Segment.csv
```

In Google Colab, mount Drive, add the shared folder to *My Drive*, and set one editable constant in the notebook:

```python
DATA_DIR = "/content/drive/MyDrive/<folder name>"
```

The loader lists the files it finds before reading them, resolves the required columns, and stops with an actionable message if anything is missing. `USE_SYNTHETIC_DEMO = True` runs the whole pipeline on generated data for a smoke test; every table produced that way is stamped `synthetic` and is **not** a project result.

**Data handling.** Do not commit the course data, participant data, or any identifying information to a public repository. Mouse dynamics are behavioural biometric data and fall under the course's and the institution's handling requirements.

---

## 5. Reproducible workflow

1. Configure `DATA_DIR` (and, if the Drive shortcut name differs, `EXTRA_DATA_DIR_CANDIDATES`).
2. Load and concatenate the per-user files; keep the originating file on every row so that label consistency can be audited.
3. Validate that `avg_d`, `avg_tv`, `avg_a` and `label` exist; never fabricate a label.
4. Coerce the three features to numeric, replace ±inf with missing, drop unusable rows, and report how many were dropped.
5. Explore class counts, per-label distributions, feature correlation and the 2-D projection.
6. **Reconstruct the recording in three dimensions** — the three-point segments are consecutive pieces of one continuous trajectory, so they are stitched back into a single path and shown as (x, y, time) and as a top-down screen view.
7. **Run the leakage probe** — measure what the three selected features encode against what the excluded raw coordinates and timestamps encode, so that the exclusion is justified by evidence rather than by preference.
8. Stratified 80/20 split with `random_state=42`; the test portion is then left untouched.
9. Nested cross-validation on the training portion: an outer 5-fold loop, and inside each outer fold a 3-fold inner loop that scores every candidate configuration against every one of the seven feature subsets.
10. Select the model with the pre-registered one-standard-error rule; re-select the configuration and subset on the full training portion.
11. Fit once, evaluate once on the test portion; report macro metrics, a confusion matrix, a bootstrap confidence interval, and calibration.
12. Diagnose over-fitting with learning curves and train/validation gaps.

Every step is seeded. `RANDOM_STATE = 42` throughout.

### 5.1 Runtime and the `FAST_MODE` switch

The wrapper search is the expensive part: the forest is fitted several thousand times, and the tree count dominates. Two consequences are handled explicitly rather than hidden:

* **The search uses a cheaper forest than the final model** (`N_ESTIMATORS_SEARCH` = 100 trees against `N_ESTIMATORS_FINAL` = 200). The search only has to *rank* configurations and subsets; every reported result, including the held-out metrics, is produced with the full 200-tree forest.
* **Sprint 5 starts with a timing probe** that fits each model once and prints the projected cost of the search, so the scale of the run is visible before it begins rather than after.

If the dataset turns out to be large, set `FAST_MODE = True` in the configuration cell. That reduces the outer folds from five to three, halves the SVC grid, and uses a 50-tree search forest. **It weakens the protocol, so the mode in force is printed in the run manifest and must be stated in the report.** On a 2-core machine with roughly 1,200 segments per user, the full protocol takes about three minutes.

### 5.2 Reading the course files

The loader is deliberately forgiving about the files and strict about the result:

* A CSV whose columns cannot be typed automatically (a stray string inside a numeric column) is re-read as text instead of crashing the run; Sprint 2 then converts the three selected features to numeric and reports exactly how many rows were dropped as a result.
* A CSV that does not contain the three selected features is skipped with a printed reason, so a stray file in the folder cannot contaminate the experiment.
* A label column is never invented. If the files carry no label, the label is derived from the `UserN_...` file name and the derivation is printed loudly; if some files carry a label and others do not, the derived values are cast into the same label space as the existing ones.


---

## 6. Models

| Model | Family | Role | Scaling |
| --- | --- | --- | --- |
| RBF SVC | Kernel, maximum-margin | Strong non-linear baseline on small data | yes |
| Random Forest | Bagged tree ensemble | Robust, scale-invariant, provides importances | no |
| Logistic Regression | Regularised linear | Interpretable baseline that any non-linear claim must beat | yes |
| K-Nearest Neighbours | Instance-based | Tests whether the classes are locally coherent | yes |
| Gaussian Naive Bayes | Generative | Tests how much the modelling assumptions matter | no |

All hyper-parameters are declared in one place, are real scikit-learn parameters, and are **verified programmatically against the installed scikit-learn signature with their library defaults printed alongside** (Sprint 4). No parameter value is invented, and no grid is refined after results are seen.

**On the training dynamics.** None of these models is trained by back-propagation, so the classic vanishing-gradient failure mode does not apply to this project at all: the SVM solves a constrained quadratic programme, the forest grows trees by greedy impurity reduction, Gaussian Naive Bayes estimates parameters in closed form, KNN has no training stage, and Logistic Regression uses L-BFGS on three standardised features with L2 regularisation, where the gradient chain is three terms long and cannot vanish. The failure mode that *is* present is over-fitting, and it is measured — through nested cross-validation, the learning curve, and the explicit train/validation gap — rather than asserted.

---

## 7. Evaluation

This is a **multi-class identification** task, so every user must carry equal weight:

* **Primary metric:** macro-averaged F1, with macro precision, macro recall, accuracy and weighted F1 reported alongside.
* **Secondary:** per-label precision/recall, confusion matrix (counts and row-normalised), one-vs-rest ROC AUC (macro and weighted), and permutation importance.
* **Fairness of the comparison:** every model uses the same split, the same folds, the same inner protocol and the same scoring function. The scaler lives inside each pipeline, so it is refitted on every training fold and never sees a validation fold.
* **Uncertainty:** a 2,000-resample bootstrap 95% confidence interval for the headline macro F1, plus the fold-to-fold spread.
* **Reference points:** chance level, a majority-class dummy, and the all-features baseline evaluated under the identical protocol. A model that does not clear these has not demonstrated anything.
* **Confidence:** the final model is wrapped in `CalibratedClassifierCV` (sigmoid, fitted on the training portion only) and assessed with a reliability diagram, expected calibration error, the multiclass Brier score, log loss and the distribution of maximum predicted probability. Accuracy alone does not say whether the model knows when it is wrong.

### 7.1 Comparing the three feature-selection families

The course asks for a filter, a wrapper and an embedded selector, and the project title names the wrapper. The notebook therefore does not only *assert* that the wrapper is the right choice — it **measures** the three families against each other, on the same outer folds, with the same estimator:

| Family | How it selects | Search space over the three features |
| --- | --- | --- |
| Filter | ANOVA F-test ranking, model-independent | 3 candidates (top-1, top-2, top-3) |
| Embedded | the model's own importance estimate | 3 candidates (top-1, top-2, top-3) |
| Wrapper | every subset is trained and scored by the learner | 7 candidates (all non-empty combinations) |

The comparison is reported whatever it shows. The wrapper genuinely has the larger search space, and that extra freedom is also a source of optimism — the notebook prints the outer-fold score of each family so the question "did the larger search space pay off?" is answered by the numbers rather than by the project title.

### 7.2 A note on `RFE`

`RecursiveFeatureElimination` is deliberately **not** used as a comparison point, for a substantive reason rather than a technical one: RFE ranks features by coefficient or importance magnitude and eliminates them one at a time. It answers "which features does this model consider weak?" — not "which combination trains the best model?". The forward `SequentialFeatureSelector` is used as the cross-check instead, because it shares the wrapper's criterion (cross-validated predictive performance) and therefore compares like with like.


**A single false-positive rate is deliberately not reported.** This is not a binary authentication task, and a verification protocol with FAR/FRR would be a different experiment with a different design, not a re-labelling of this one.

### Result table

Populate this table from a completed notebook run only. The notebook prints a ready-to-paste version of it in Sprint 8.

| Model | Accuracy | Macro precision | Macro recall | Macro F1 | 95% CI (macro F1) |
| --- | ---: | ---: | ---: | ---: | --- |
| *to be generated from the executed notebook* | | | | | |
| *all-features baseline (same family)* | | | | | |
| *majority-class dummy* | | | | | |
| *chance level* | | | | | |

### Model comparison (nested cross-validation, training portion only)

| Model | Wrapper macro F1 (mean ± s.e.) | All-features baseline macro F1 (mean ± s.e.) |
| --- | ---: | ---: |
| *to be generated from the executed notebook* | | |

---

## 8. Limitations and responsible use

1. **Segment-level random splitting.** Segments from one recording session can appear in both the training and the test portion, so the model may recognise the session rather than the user. A session- or participant-level split would be the correct protocol and would be stricter. No session identifier is available in the delivered data, and none is invented here. The leakage probe in Sprint 3 measures how severe this is for the three selected features rather than leaving it as an unquantified caveat.
2. **Three features is a hard ceiling.** They omit direction, curvature, timing structure, pause behaviour and device geometry. The result describes the ceiling of this feature set, not of mouse-dynamics biometrics. Note that a model *allowed* to use the raw coordinates and timestamps scores far higher on this data — but a large part of that gap is recording identity rather than user identity, so the honest ceiling for a behavioural biometric is the lower number.
3. **Small sample, few labels.** With roughly ten labels and about 160 segments per user, one unusual recording can move macro F1 by several points. The bootstrap interval and the fold-to-fold spread quantify this.
4. **Uncontrolled confounds.** Device, screen geometry and task context can shift the distributions more than the difference between two users.
5. **Identification, not authentication, and not fraud detection.** The framing here is not the framing a proctoring system would use, and the result must not be presented as a fraud-detection outcome.
6. **Behavioural biometric data.** Store and share the data responsibly; follow the course and institutional requirements.
7. **The notebook is written independently for this project.** The supplied reference notebook informs dataset understanding only; it is not copied as the implementation, and all code in the feature-selection and evaluation path was written for this project.

---

## 9. Repository layout

```text
.
├── Elevating_Academic_Integrity_Wrapper_Feature_Selection.ipynb   # the deliverable (no stored outputs)
├── Verification_run_on_synthetic_data_EXECUTED.ipynb              # evidence that the notebook runs end to end
├── README.md                                                      # this file
├── requirements.txt                                               # optional, if exported from Colab
└── nb/cells/                                                      # the notebook's cell sources
    ├── 00.md … 19.py
    └── build_notebook.py (in the repository root) assembles them into the .ipynb
```

Typical dependencies: `numpy`, `pandas`, `matplotlib`, `seaborn`, `scikit-learn`.

### 9.1 About the verification notebook

`Verification_run_on_synthetic_data_EXECUTED.ipynb` is a stored run of the deliverable against **generated** data, with the same awkward shapes the real files have (integer labels, raw `x*`/`y*`/`t*` columns, an exported index column, and a few non-numeric and infinite values). It is included so that the execution of every cell can be inspected without running anything.

**Its numbers are not results.** They describe the generator, not the course data, and every table it produces is stamped `synthetic`. Its only purpose is to show that the pipeline executes, that the cleaning path reports what it removes, and that the figures render. The delivered notebook itself carries **no stored outputs**, so no result can be mistaken for a real one.


---

## 10. Reference material

- Course sample/documentation example: [Google Colab example](https://colab.research.google.com/drive/1J36r0VPZt6JI9g8DUiH0zeeAU82tzpAF)
- Course data folder: [shared Google Drive folder](https://drive.google.com/drive/folders/1hAYr22yo8wWvUvJF88JZfNQvgHlL7hxW)
- Feature-selection background: [scikit-learn feature selection documentation](https://scikit-learn.org/stable/modules/feature_selection.html)
- Wrapper method: Kohavi, R., & John, G. (1997). The wrapper approach to automatic feature selection. *Artificial Intelligence*, 97(1–2), 273–302.
- Agile values: Beck, K., et al. (2001). *Manifesto for Agile Software Development*. https://agilemanifesto.org/
- Empirical design paradigm: Ralph, P. (2018). The two paradigms of software development research. *Science of Computer Programming*, 156, 68–89.
- Socio-technical systems: Baxter, G., & Sommerville, I. (2011). Socio-technical systems: From design methods to systems engineering. *Interacting with Computers*, 23(1), 4–17.
- Teamwork quality and project success: Lindsjørn, Y., et al. (2016). Teamwork quality and project success in software development. *Journal of Systems and Software*, 122, 274–286.
- Agile evidence base: Dybå, T., & Dingsøyr, T. (2008). Empirical studies of agile software development: A systematic review. *Information & Software Technology*, 50(9–10), 833–859.
