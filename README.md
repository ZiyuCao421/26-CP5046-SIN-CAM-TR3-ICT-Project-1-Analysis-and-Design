# Mouse Dynamics: Lightweight Three-Feature User Identification

> **Status:** implementation guide. This README is a project reference, not evidence of executed results. The final, run notebook is the source of truth for the dataset version, preprocessing, figures, metrics, and conclusions.

## Project overview

This repository contains a deliberately lightweight mouse-dynamics experiment for **multi-class user identification**. Each observation is a short three-point mouse-movement segment. The task is to predict one of the available user labels from just three interpretable, movement-derived features.

The project is designed as a compact alternative to a larger feature-selection and deep-learning workflow. It prioritises a transparent baseline that can be inspected, reproduced, and honestly evaluated before more complex methods are considered.

## Research question

To what extent can a small, interpretable feature set derived from mouse movement distinguish between users in the supplied three-point-segment dataset?

This is an exploratory classification exercise. It does **not** by itself establish that the method is suitable for real-world authentication or fraud detection.

## Selected features

| Column | Meaning in this project | Why retain it |
| --- | --- | --- |
| `avg_d` | Average movement-distance feature for a segment | Represents movement magnitude. |
| `avg_tv` | Dataset's average velocity-related (`TV`) feature | Represents movement speed. |
| `avg_a` | Average acceleration feature for a segment | Represents how movement changes over time. |

Together, these features represent a compact **distance–speed–acceleration** description of a segment. The exact feature definitions remain those supplied by the dataset/notebook.

### Why not use raw timestamps or coordinates?

The reference notebook contains raw point values such as `x0`, `y0`, and `t0`. This lightweight version intentionally avoids them: absolute coordinates may reflect screen/layout conditions, and absolute timestamps or collection order can introduce leakage that inflates a random-split result. The chosen three features are derived behavioural summaries and are easier to justify.

## Data

Use the course-provided mouse-dynamics CSV files, available from the [shared data folder](https://drive.google.com/drive/folders/1hAYr22yo8wWvUvJF88JZfNQvgHlL7hxW). The reference data layout uses files similar to:

```text
User1_Calib_3MouseMovement_MCQ_Segment.csv
...
User10_Calib_3MouseMovement_MCQ_Segment.csv
```

The final notebook should validate that these columns exist before modelling:

```python
REQUIRED_COLUMNS = ["avg_d", "avg_tv", "avg_a", "label"]
```

Do not commit course data, participant data, or identifying information to a public repository.

## Reproducible workflow

1. Set one editable `DATA_DIR` path in the notebook.
2. Load and concatenate the available user CSV files; remove export-only columns such as `Unnamed: 0` when present.
3. Validate the label and all three selected features; coerce only the selected features to numeric and report rows removed because of missing or infinite values.
4. Explore class counts and the distributions/correlation of the three features.
5. Make a stratified 80/20 train–test split with `random_state=42`.
6. Fit a `StandardScaler` on training data only, then evaluate an RBF Support Vector Classifier (SVC).
7. Evaluate a Random Forest classifier as a nonlinear tree-based comparison.
8. Report accuracy, macro precision, macro recall, macro F1, a classification report, and confusion matrices.
9. Record the best model **only after execution** and interpret errors without overstating the result.

## Models

| Model | Role | Notes |
| --- | --- | --- |
| RBF SVC | scaled margin-based baseline | Uses `StandardScaler` fitted on training data only. |
| Random Forest | nonlinear, tree-based comparison | Provides a robust small-data comparison without scaling. |

The notebook should keep hyperparameters modest and explicit. Hyperparameter tuning is out of scope unless it is added as a documented, separately evaluated extension.

## Evaluation

This is a multi-class identification task, so macro-averaged metrics matter: they give each user equal weight even if class counts differ. A confusion matrix should be used to identify which user labels are most often confused.

The project should **not** report a single false-positive rate as though this were a binary authentication task. If the problem is later redesigned as claimed-user verification, define the positive class and use appropriate verification measures (for example, FAR/FRR) with a separate protocol.

### Result table

Populate this table only from a completed notebook run.

| Model | Accuracy | Macro precision | Macro recall | Macro F1 |
| --- | ---: | ---: | ---: | ---: |
| RBF SVC | To be generated | To be generated | To be generated | To be generated |
| Random Forest | To be generated | To be generated | To be generated | To be generated |

## Repository layout

```text
.
├── Mouse_Dynamics_Lightweight_3_Features.ipynb
├── README.md
└── requirements.txt                 # optional, if the project is exported from Colab
```

Typical dependencies are `numpy`, `pandas`, `matplotlib`, `seaborn`, and `scikit-learn`.

## Limitations and responsible use

- A random segment-level split may place related movements from one recording session in both train and test data. If session/recording identifiers become available, compare against a group- or session-based split.
- Performance can be influenced by device, screen geometry, task context, sampling rate, and data imbalance.
- Mouse dynamics are behavioural biometric data. Store and share the data responsibly and follow course and institutional requirements.
- The notebook is written independently for this project. The supplied reference notebook informs dataset understanding only; it is not copied as the implementation.

## Reference material

- Course sample/documentation example: [Google Colab example](https://colab.research.google.com/drive/1J36r0VPZt6JI9g8DUiH0zeeAU82tzpAF)
- Feature-selection background: [scikit-learn feature selection documentation](https://scikit-learn.org/stable/modules/feature_selection.html)

