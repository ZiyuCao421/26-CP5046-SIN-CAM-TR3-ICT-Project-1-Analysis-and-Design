# Stages 1–2 — Single-feature results

## Scope

This directory records the aggregate outcome of the single-feature workflow. Raw course CSV files, participant identifiers, labels and row-level values are intentionally excluded from Git.

## Run summary

| Item | Result |
|---|---:|
| Merged rows / common cohort rows | 2,002 / 2,002 |
| Classes | 9 |
| Shared train / test split | 1,601 / 401 |
| Candidate features screened | 57 |
| Excluded columns | 13 |
| Model-feature evaluations | 171 |
| Failed evaluations | 0 |
| Split fingerprint | `sha256:297284c375de6cb3535fbd2268907915` |

The shared cohort was built once with `complete_case_all_candidates`; every feature reused the same stratified 80/20 split. The models were DummyClassifier (majority-class reference), scaled RBF SVC, and balanced Random Forest.

## Stage-2 shortlist

The descriptive, deterministic Stage-2 shortlist contains 12 candidates:

`max_y`, `min_y`, `avg_y`, `min_t`, `max_t`, `avg_t`, `min_x`, `VV2`, `max_x`, `avg_x`, `VV0`, `min_hv`.

The shortlist was ranked from existing held-out metrics; it is a candidate pool, not a final generalisation claim. In particular, absolute position/time summaries require caution because they may encode recording conditions.

See the full phase report in [`../../docs/02_phase1_stage2_report_reviewed.md`](../../docs/02_phase1_stage2_report_reviewed.md).
