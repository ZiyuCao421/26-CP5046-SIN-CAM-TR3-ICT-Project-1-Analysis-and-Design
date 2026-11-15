# V3.5 Combined Champion Results

These are aggregate, non-row-level outputs from the combined comparison of Shah's three candidate feature sets and Ziyu's three V3 feature sets.

The result is a **provisional held-out comparison** on one fixed split. Raw CSV data, individual rows, predictions and absolute paths are intentionally excluded.

## Outcome

- **Champion feature set:** `ZIYU_FS2_svc_diverse`
- **Recommended model:** 200-tree class-balanced Random Forest
- **FS2 + Random Forest held-out Macro F1:** `0.7740`
- **Feature-set ranking score:** mean Macro F1 across RBF SVC and Random Forest; FS2 = `0.6376`

`ZIYU_FS3_random_forest_diverse` has the highest single model–set score (Random Forest Macro F1 = `0.7749`), but FS2 is the champion under the pre-declared two-model mean rule.

See [the final combined report](../../docs/06_final_combined_champion_report.md) for interpretation and limitations.
