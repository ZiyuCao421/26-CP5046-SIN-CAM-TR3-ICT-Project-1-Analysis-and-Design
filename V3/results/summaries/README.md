# V3 results summary and reproducibility record

## Workflow status

| Stage | Status | Main output |
|---|---|---|
| 1 — all-feature screening | Completed | 57 comparable single-feature results |
| 2 — visualisation and shortlist | Completed | 12 candidate features |
| 3 — correlation and set definition | Completed | 3 provisional low-redundancy sets |
| 4 — feature-set evaluation | Completed | provisional champion: `FS3_random_forest_diverse` |

All four stages used the same 2,002-row, nine-class complete-case cohort and the same fingerprinted 80/20 stratified split: `sha256:297284c375de6cb3535fbd2268907915`.

## What belongs in this repository

Only aggregate metrics, figures, manifests and reports should be committed. Raw CSV files, per-row values, labels, predictions, participant filenames and local absolute paths must remain outside GitHub.

For the complete interpretation, methodology, findings and limitations, see [`../../docs/04_v3_complete_report.md`](../../docs/04_v3_complete_report.md).
