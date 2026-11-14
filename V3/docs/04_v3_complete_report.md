# V3 Complete Report — Mouse-dynamics Feature Selection and Feature-set Evaluation

## Executive summary

V3 implemented an auditable four-stage workflow for multi-class mouse-dynamics identification. From 57 eligible features, a deterministic Stage-2 shortlist of 12 features was produced; Stage 3 removed highly redundant combinations; and Stage 4 compared three six-feature sets with the same Dummy, RBF SVC and Random Forest models.

The provisional winner is **`FS3_random_forest_diverse`**: `min_y`, `VV0`, `VV2`, `min_hv`, `min_t`, `min_x`. Its Macro F1 was 0.487 for RBF SVC and 0.807 for Random Forest, with a two-model mean of 0.647.

## Reproducible protocol

All stages used 2,002 rows, nine classes and one `complete_case_all_candidates` cohort. One stratified 80/20 split produced 1,601 training and 401 held-out rows. Its irreversible identifier was `sha256:297284c375de6cb3535fbd2268907915`.

The three unchanged models were a majority-class DummyClassifier reference, a training-only scaled RBF SVC (`C=1.0`, `gamma='scale'`, balanced classes), and a 200-tree balanced Random Forest. Macro F1 was primary because it weights every class equally; accuracy, macro precision and macro recall were retained as secondary diagnostics.

## Results pathway

1. **Screening:** 57 candidate features were each evaluated on the shared cohort; 13 non-feature, ID, raw-coordinate or raw-time columns were excluded and no screening failed.
2. **Shortlisting:** 12 candidates were selected transparently from existing held-out metrics.
3. **Correlation:** 66 feature pairs were checked; nine reached the 0.85 redundancy threshold. This prevented closely related `avg`, `min` and `max` summaries from being silently combined.
4. **Set evaluation:** the three candidate sets were tested under the same split and model parameters.

| Set | RBF SVC Macro F1 | Random Forest Macro F1 | Mean |
|---|---:|---:|---:|
| `FS3_random_forest_diverse` | 0.487 | 0.807 | 0.647 |
| `FS2_svc_diverse` | 0.488 | 0.795 | 0.642 |
| `FS1_consensus_compact` | 0.484 | 0.790 | 0.637 |

## Interpretation and limitations

Random Forest consistently outperformed the RBF SVC for these sets, and FS3 had the highest average real-model Macro F1. However, FS3 and FS2 are close; the result should be called a **provisional held-out winner**, not a final deployed model.

Some selected features are absolute position/time summaries. They were retained only with an explicit recording/session-risk flag: they may describe the setup or session rather than stable person-specific behaviour. Furthermore, all selection and comparison decisions reuse one held-out split. Repeated splits, group-aware validation and independent data are required before claiming generalisation.

## Repository guide

- [`../results/single_feature/`](../results/single_feature/): stages 1–2 result report.
- [`../results/feature_sets/`](../results/feature_sets/): stages 3–4 result report.
- [`../results/summaries/`](../results/summaries/): cross-stage reproducibility summary.
- [`../notebooks/`](../notebooks/): the four Colab notebooks that produce the aggregate artefacts.

Raw course data is intentionally not included in this repository.
