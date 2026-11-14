# Stages 3–4 — Feature-set results

## Correlation screen

Stage 3 rebuilt the Stage-1 cohort from the raw CSVs inside Colab and passed 28 input-contract checks. It calculated Pearson and Spearman correlations for the 12 shortlisted features only.

| Item | Result |
|---|---:|
| Features correlated | 12 |
| Upper-triangle pairs | 66 |
| Redundant pairs at \|r\| or \|rho\| ≥ 0.85 | 9 |
| Correlation threshold | 0.85 |

The strongest redundancy was among related `avg`, `min` and `max` summaries of the same coordinate/time family. Three low-redundancy, six-feature candidate sets were therefore created:

| Set | Members |
|---|---|
| `FS1_consensus_compact` | `max_y`, `min_t`, `min_x`, `VV2`, `VV0`, `min_hv` |
| `FS2_svc_diverse` | `avg_t`, `max_y`, `min_x`, `VV2`, `min_hv`, `VV0` |
| `FS3_random_forest_diverse` | `min_y`, `VV0`, `VV2`, `min_hv`, `min_t`, `min_x` |

## Feature-set evaluation

Stage 4 recreated the same 2,002-row cohort and fingerprint, then evaluated every set with the unchanged models. Macro F1 is the primary metric.

| Rank | Set | RBF SVC | Random Forest | Mean of real models |
|---:|---|---:|---:|---:|
| 1 | `FS3_random_forest_diverse` | 0.487 | 0.807 | 0.647 |
| 2 | `FS2_svc_diverse` | 0.488 | 0.795 | 0.642 |
| 3 | `FS1_consensus_compact` | 0.484 | 0.790 | 0.637 |

`FS3_random_forest_diverse` is the **provisional champion feature set** for this fixed held-out split. The advantage over FS2 is small (about 0.005 mean Macro F1), so no robust generalisation claim is warranted without repeated, group-aware or external validation.
