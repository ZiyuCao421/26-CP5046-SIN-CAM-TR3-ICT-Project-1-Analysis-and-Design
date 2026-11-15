# Final Combined Feature-Set Champion Report

## Decision in one sentence

**`ZIYU_FS2_svc_diverse` is the provisional champion feature set because it achieved the highest mean Macro F1 across the two real models (0.6376); the recommended model is the 200-tree balanced Random Forest, which reached Macro F1 = 0.7740 with this set.**

---

## 1. Purpose

This final comparison joined Shah Htet Naing's three feature-selection candidates with Ziyu Cao's three correlation-controlled V3 sets. The aim was to select the best-supported set under one shared and reproducible evaluation contract, rather than compare results produced on different cohorts, splits, or models.

## 2. Common evaluation contract

| Item | Applied setting |
| --- | --- |
| Source data | 10 course CSV files; 2,002 valid rows after the shared complete-case check |
| Classification task | 9-class user identification |
| Data split | Fixed stratified 80/20 split: 1,601 training rows and 401 held-out rows; `random_state = 42` |
| Feature sets | 6 total: Shah ANOVA Top-10, Shah Random Forest Top-10 proxy, Shah Lasso Top-10, Ziyu FS1, FS2 and FS3 |
| Models | Majority-class DummyClassifier, scaled RBF SVC, and 200-tree class-balanced Random Forest |
| Primary measure | Held-out **Macro F1** |
| Secondary measures | Accuracy, macro precision and macro recall |

Macro F1 was the primary measure because every user class receives equal weight. This matters because the classes were not equally sized.

## 3. Champion feature set

### `ZIYU_FS2_svc_diverse`

`avg_t`, `max_y`, `min_x`, `VV2`, `min_hv`, `VV0`

FS2 contains six complementary position, time and vertical-velocity summaries. It was selected using the pre-declared rule: the highest **mean Macro F1 across RBF SVC and Random Forest**.

| Model | Held-out Macro F1 |
| --- | ---: |
| DummyClassifier | 0.0342 |
| RBF SVC (scaled) | 0.5013 |
| Random Forest (balanced, 200 trees) | **0.7740** |
| Mean of the two real models | **0.6376** |

The champion mean is 0.0020 above FS3 (0.6376 versus 0.6356). The margin is small, so FS2 should be described as the **provisional held-out feature-set champion**, not a final deployment-ready answer.

## 4. Champion model

### 200-tree balanced Random Forest

The Random Forest was the strongest model family for every Ziyu V3 set. With the selected FS2 it achieved:

- Accuracy: **0.7606**
- Macro precision: **0.7948**
- Macro recall: **0.7643**
- Macro F1: **0.7740**

This is substantially above the majority-class DummyClassifier Macro F1 of 0.0342 and above the FS2 scaled RBF SVC Macro F1 of 0.5013. The practical recommended pipeline from the declared feature-set rule is therefore:

> **`ZIYU_FS2_svc_diverse` → RandomForestClassifier (200 trees, `class_weight='balanced'`, `random_state=42`)**

## 5. Full comparison summary

| Rank | Feature set | Features | RBF SVC Macro F1 | Random Forest Macro F1 | Mean Macro F1 |
| ---: | --- | ---: | ---: | ---: | ---: |
| 1 | `ZIYU_FS2_svc_diverse` | 6 | 0.5013 | 0.7740 | **0.6376** |
| 2 | `ZIYU_FS3_random_forest_diverse` | 6 | 0.4963 | **0.7749** | 0.6356 |
| 3 | `ZIYU_FS1_consensus_compact` | 6 | **0.5015** | 0.7514 | 0.6265 |
| 4 | `SHAH_anova_top10` | 10 | 0.1957 | 0.2430 | 0.2193 |
| 5 | `SHAH_lasso_top10` | 10 | 0.1654 | 0.2526 | 0.2090 |
| 6 | `SHAH_random_forest_top10_proxy` | 10 | 0.1398 | 0.2497 | 0.1948 |

### Important distinction

The **highest single model–set score** was `ZIYU_FS3_random_forest_diverse` with Random Forest (Macro F1 = **0.7749**). However, FS2 is the official feature-set champion because the stated decision rule ranks sets by their performance across **both** real models. Choosing FS3 only because it is 0.0009 higher under Random Forest would change the declared rule after inspecting the results.

## 6. Contribution of the two development streams

Shah's ANOVA, Random Forest importance and Lasso work provided three different candidate-selection perspectives and produced the cross-team benchmark sets. Ziyu's V3 workflow supplied the complete-case contract, fixed split, correlation-controlled FS1–FS3 definitions, unchanged models, stored metrics and visual outputs.

The controlled comparison shows that Ziyu's six-feature, low-redundancy sets outperformed Shah's current Top-10 candidates under this shared protocol. This does not mean Shah's exploratory work failed: it supplied useful feature-engineering evidence. It means that individual importance rankings need a later controlled feature-set evaluation before they can be treated as an optimal combined set.

## 7. Evidence files to retain

Keep these outputs together in the final results folder:

- `combined_feature_set_metrics.csv` — all 18 set/model evaluations.
- `combined_feature_set_ranking.csv` — feature-set decision table.
- `combined_feature_set_manifest.json` — data, split, model and provenance record.
- `provisional_champion_feature_set.json` — chosen FS2 membership.
- `figures/combined_macro_f1_by_feature_set_and_model.png` — overall visual comparison.
- `figures/combined_feature_set_champion_ranking.png` — feature-set ranking.
- `figures/combined_real_model_macro_f1_heatmap.png` — SVC/Random Forest matrix.

## 8. Limitation and next action

This result is based on one fixed held-out split from the same course dataset. It does not establish performance for new users, new sessions, new devices, or real online examinations. Shah's Random Forest entry is also a documented displayed Top-10 proxy, not the complete 19-feature `SelectFromModel` set.

The next validation step is repeated and group/session-aware evaluation. Until then, report FS2 plus the balanced Random Forest as the **current provisional recommendation**.
