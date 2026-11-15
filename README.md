# Mouse Dynamics Behavioural Biometrics — Analysis and Design

> **CP5046 ICT Project 1 · Group 2**
> **Current outcome:** `ZIYU_FS2_svc_diverse` with a 200-tree class-balanced Random Forest is the **provisional held-out recommendation** (Macro F1 = **0.7740**).

## Project purpose

This repository documents a staged, reproducible investigation of whether engineered mouse-movement features can distinguish user labels in a course mouse-dynamics dataset. The work is framed as a classroom-scale **multi-class user-identification experiment**. It is not a deployed authentication system, a cheating detector, or evidence of performance in live examinations.

The project deliberately progresses from a small, explainable prototype to an auditable feature-selection workflow:

```text
V2: three-feature baseline
        ↓
V3: screen all eligible features → shortlist → correlation control → evaluate sets
        ↓
V3.5: compare Shah and Ziyu feature sets under one shared evaluation contract
        ↓
Provisional held-out recommendation
```

## Final combined result

Six declared feature sets were tested on the same 2,002-row, 9-class complete-case cohort with the same stratified 80/20 split (1,601 training rows / 401 held-out rows), unchanged models and Macro F1 as the primary measure.

### Champion feature set

**`ZIYU_FS2_svc_diverse`**

`avg_t`, `max_y`, `min_x`, `VV2`, `min_hv`, `VV0`

FS2 was selected using the pre-declared rule: highest **mean Macro F1 across the two real models**.

| Model | Held-out Macro F1 |
| --- | ---: |
| DummyClassifier (most frequent) | 0.0342 |
| Scaled RBF SVC | 0.5013 |
| 200-tree balanced Random Forest | **0.7740** |
| **Mean of the two real models** | **0.6376** |

### Recommended model

The recommended model is:

```python
RandomForestClassifier(
    n_estimators=200,
    class_weight="balanced",
    random_state=42,
)
```

With FS2, it achieved accuracy = **0.7606**, macro precision = **0.7948**, macro recall = **0.7643**, and macro F1 = **0.7740** on the fixed held-out portion.

### Important interpretation boundary

`ZIYU_FS3_random_forest_diverse` paired with Random Forest produced the highest **single** model–set Macro F1 (0.7749). FS2 remains the feature-set champion because the declared ranking rule uses performance across **both** RBF SVC and Random Forest: FS2 = 0.6376 versus FS3 = 0.6356. The margin is small, so this is a **provisional held-out decision**, not a generalisation or deployment claim.

## Workflow

| Version / stage | Purpose | Main evidence |
| --- | --- | --- |
| **V2** | Lightweight, explainable baseline using `avg_d`, `avg_tv` and `avg_a`. | [V2 guide](V2/README%20(2).md) and executed verification notebook. |
| **V3 Stage 1** | Screen every eligible numeric feature under a repeatable contract. | 57 eligible features; 13 non-feature/leakage/bookkeeping columns excluded. |
| **V3 Stage 2** | Visualise single-feature outputs and create a transparent shortlist. | 12 candidates retained. |
| **V3 Stage 3** | Use Pearson and Spearman correlation to document redundancy and define sets. | 66 pairs checked; 9 redundant pairs at the stated threshold. |
| **V3 Stage 4** | Compare FS1, FS2 and FS3 with unchanged models. | [V3 complete report](V3/docs/04_v3_complete_report.md). |
| **V3.5** | Compare three Shah candidate sets and three Ziyu V3 sets under one shared contract. | Current combined recommendation reported above. |

## Feature-set sources

### Shah Htet Naing — exploratory candidate sets

- **ANOVA / SelectKBest Top-10:** univariate separation ranking.
- **Random Forest Top-10 proxy:** displayed feature-importance ranking; the full original SelectFromModel output contained 19 columns and is not represented as a complete set here.
- **Lasso Top-10:** L1 Logistic Regression ranking.

These methods provide useful candidate evidence, but individual importance ranks are not treated as final winners until evaluated under the shared V3.5 cohort, split, models and metric.

### Ziyu Cao — correlation-controlled sets

- `FS1_consensus_compact`
- `FS2_svc_diverse` **← current feature-set champion**
- `FS3_random_forest_diverse`

The V3 workflow and its audit trail are described in the [team design portfolio report](V3/docs/05_group2_team_design_portfolio_report.md).

## Data and evaluation contract

| Item | Applied setting |
| --- | --- |
| Input files | Course-provided `UserN_Calib_3MouseMovement_MCQ_Segment.csv` files |
| Observed data used in combined run | 10 files, 2,002 rows, 70 source columns, 9 observed user labels |
| Raw data location | Course Google Drive only — not committed to this repository |
| Split | Stratified 80/20; `random_state=42` |
| Baseline | `DummyClassifier(strategy="most_frequent")` |
| Real models | Training-only scaled RBF SVC; 200-tree class-balanced Random Forest |
| Primary metric | Macro F1 |
| Secondary diagnostics | Accuracy, macro precision and macro recall |

Raw `x0–x2`, `y0–y2` and `t0–t2` coordinate/time columns are not used as combined feature-set inputs. They can encode layout or acquisition conditions rather than transferable behavioural identity signals.

## Repository structure

```text
.
├── V2/                         # Lightweight three-feature prototype
├── V3/
│   ├── notebooks/               # Reusable four-stage Colab workflow
│   ├── configs/                 # Screening configuration
│   ├── docs/                    # Methods, reports, portfolio evidence
│   └── results/                 # Aggregate outputs and result guidance
├── Mouse_Dynamics_Lightweight_3_Features.ipynb
└── README.md
```

## Reproducibility and responsible use

- Run notebooks in Google Colab after mounting the authorised course-data folder.
- Keep data paths configurable; do not commit raw CSV files, participant data or identifiers.
- Preserve configurations, manifests, aggregate metrics, figures and limitation statements alongside each run.
- Do not present a fixed-split classroom result as a production fraud-detection claim.
- A stronger future validation would use repeated splits plus session-, participant- or device-aware grouping where those identifiers are available.

## Related project evidence

- [V3 complete technical report](V3/docs/04_v3_complete_report.md)
- [Group 2 team design portfolio report](V3/docs/05_group2_team_design_portfolio_report.md)
- [Feature-set evaluation plan](V3/docs/03_feature_set_evaluation_plan.md)

---

**Summary:** The project evolved from a transparent three-feature baseline into an auditable screening and feature-set evaluation pipeline. Under the current shared held-out comparison, **FS2 + a balanced Random Forest** is the most defensible provisional recommendation.
