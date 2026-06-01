# Kết quả thực nghiệm v2 — TCR Filter Metrics

**Dataset:** 4,414 mẫu, 53 Java project

**Feature sets:**
| Tên | Size | Mô tả |
|---|---:|---|
| `X_old` | 38 | Original Martins features |
| `X_only_ck` | 40 | CK features only |
| `X_full` | 78 | X_old + all CK |
| `X_selected` | 58 | X_old + top-20 CK (additive ablation, Protocol B RF) |
| `X_sel_global` | 75 | Top-75 từ X_full (global ranking, step=5 ablation) |

**Classifiers:** RandomForest (RF), XGBoost (XGB), LogisticRegression (LR), SVM-RBF (SVM), DecisionTree (DT), Dummy

**Protocols:**
- **A** — random 80/20 × 5 seeds (stratified)
- **B** — StratifiedKFold k=5 (random)
- **C** — StratifiedGroupKFold k=5, grouped by project

---

## F1 mean (± std → xem CSV)

### Protocol A — random split

| Classifier | X_old | X_only_ck | X_full | X_selected | X_sel_global |
|---|---:|---:|---:|---:|---:|
| RF | 0.706 | 0.681 | 0.732 | 0.729 | **0.732** |
| XGB | 0.704 | 0.669 | 0.716 | **0.718** | 0.717 |
| SVM | **0.701** | 0.689 | 0.695 | 0.700 | 0.695 |
| LR | 0.684 | **0.696** | 0.672 | 0.679 | 0.672 |
| DT | 0.646 | 0.621 | 0.670 | 0.663 | **0.671** |
| Dummy | 0.562 | 0.562 | 0.562 | 0.562 | 0.562 |

### Protocol B — KFold random

| Classifier | X_old | X_only_ck | X_full | X_selected | X_sel_global |
|---|---:|---:|---:|---:|---:|
| RF | 0.703 | 0.684 | 0.730 | 0.728 | **0.737** |
| SVM | **0.706** | 0.693 | 0.689 | 0.696 | 0.694 |
| XGB | 0.688 | 0.672 | **0.715** | 0.706 | 0.711 |
| LR | 0.681 | **0.688** | 0.672 | 0.681 | 0.673 |
| DT | 0.635 | 0.635 | **0.657** | 0.653 | 0.654 |
| Dummy | 0.554 | 0.554 | 0.554 | 0.554 | 0.554 |

### Protocol C — GroupKFold by project *(cross-project)*

| Classifier | X_old | X_only_ck | X_full | X_selected | X_sel_global |
|---|---:|---:|---:|---:|---:|
| SVM | **0.668** | 0.665 | 0.657 | 0.664 | 0.661 |
| LR | **0.651** | 0.649 | 0.619 | 0.623 | 0.623 |
| RF | 0.609 | 0.597 | 0.618 | **0.625** | 0.621 |
| XGB | 0.570 | 0.571 | 0.591 | **0.593** | 0.590 |
| DT | 0.551 | 0.529 | 0.565 | **0.571** | 0.564 |
| Dummy | 0.558 | 0.558 | 0.558 | 0.558 | 0.558 |

---

## Quan sát chính

**1. Protocol A ≈ Protocol B** — cả hai đều random nên kết quả gần như đồng nhất. Best: RF + X_sel_global → F1 = **0.732 / 0.737**.

**2. Protocol C thấp hơn rõ rệt** — gap ~0.06–0.10 so với A/B. Best: SVM + X_old → F1 = **0.668**. Thêm CK không nhất quán: RF/XGB tăng nhẹ, LR/SVM giảm.

**3. X_only_ck yếu hơn X_old** ở hầu hết classifier (RF −0.025, XGB −0.035 trên Protocol A). Ngoại lệ: **LR đạt F1 cao nhất với X_only_ck** (0.696/0.688 trên A/B) — CK mang tín hiệu tuyến tính không có trong original features, nhưng bị pha loãng khi kết hợp (X_full LR = 0.672).

**4. X_selected ≈ X_full trên A/B** — 58 features (X_old + top-20 CK) cho kết quả gần bằng 78 features. Trên Protocol C, X_selected nhỉnh hơn tất cả với RF/XGB/DT.

**5. X_sel_global tốt nhất trên A/B** — 75 features (top-75 từ X_full, loại 3 features yếu nhất) cho RF F1 = **0.737** (Protocol B), cao hơn X_full (0.730). Trên Protocol C không cải thiện so với X_old hay X_selected.

**6. Wilcoxon test:** 0/72 cặp có ý nghĩa thống kê (p < 0.05). Power thấp do n=5 pairs.

---

## Ablation (Protocol B, RandomForest)

### Additive: X_old + top-K CK features

| K | Total | F1 | AUC |
|---:|---:|---|---|
| 0 | 38 | 0.703 ± 0.014 | 0.718 ± 0.012 |
| 5 | 43 | 0.713 ± 0.011 | 0.737 ± 0.013 |
| 10 | 48 | 0.716 ± 0.010 | 0.742 ± 0.014 |
| 15 | 53 | 0.727 ± 0.016 | 0.748 ± 0.014 |
| **20** | **58** | 0.728 ± 0.015 | 0.749 ± 0.013 |
| 25 | 63 | 0.722 ± 0.012 | 0.747 ± 0.012 |
| 30 | 68 | 0.729 ± 0.013 | 0.751 ± 0.015 |
| 35 | 73 | **0.733 ± 0.008** | **0.753 ± 0.012** |
| 40 | 78 | 0.723 ± 0.010 | 0.749 ± 0.010 |

K_optimal = **20** — K nhỏ nhất trong vùng plateau (F1 ≥ max − 0.005, max tại K=35). K=35 có F1 cao nhất tuyệt đối nhưng std thấp hơn cũng không đơn điệu.

### Global: top-K từ X_full (mixed CK + original)

| K | CK trong top-K | F1 | AUC |
|---:|---:|---|---|
| 5 | 2 | 0.674 ± 0.007 | 0.678 ± 0.015 |
| 10 | 4 | 0.695 ± 0.008 | 0.715 ± 0.013 |
| 15 | 5 | 0.709 ± 0.010 | 0.727 ± 0.011 |
| 20 | 7 | 0.715 ± 0.009 | 0.734 ± 0.014 |
| 25 | 11 | 0.717 ± 0.008 | 0.740 ± 0.015 |
| 30 | 13 | 0.717 ± 0.014 | 0.746 ± 0.017 |
| 35 | 14 | 0.717 ± 0.016 | 0.743 ± 0.016 |
| 40 | 18 | 0.723 ± 0.014 | 0.748 ± 0.014 |
| 45 | 20 | 0.723 ± 0.012 | 0.751 ± 0.013 |
| 50 | 21 | 0.720 ± 0.012 | 0.747 ± 0.013 |
| 55 | 25 | 0.724 ± 0.014 | 0.749 ± 0.014 |
| 60 | 27 | 0.722 ± 0.007 | 0.750 ± 0.012 |
| 65 | 31 | 0.728 ± 0.017 | 0.748 ± 0.015 |
| 70 | 33 | 0.721 ± 0.010 | 0.750 ± 0.014 |
| **75** | **38** | **0.737 ± 0.010** | **0.755 ± 0.012** |
| 78 | 40 | 0.729 ± 0.010 | 0.751 ± 0.014 |

K_optimal = **75** — F1 cao nhất tại K=75; K=78 (X_full) thấp hơn, nghĩa là 3 features xếp cuối ranking thực sự gây noise.

---

## Feature lists theo ablation strategy

**Bold** = feature thuộc X_old (Martins original). Thường = CK feature.
Score = combined rank tổng hợp từ RF importance, XGB gain, permutation importance, MI, SHAP (normalize về [0,1], lấy mean).

### Additive — X_selected (58 features = tất cả X_old + top-20 CK)

| # | Feature | Nhóm | Score |
|---:|---|---|---:|
| 1 | **codeChurnMax** | churn_history | 0.851 |
| 2 | **codeChurnTotal** | churn_history | 0.822 |
| 3 | **removedLinesMax** | churn_history | 0.819 |
| 4 | assignmentsQty | ck_expressions | 0.817 |
| 5 | variablesQty | ck_expressions | 0.773 |
| 6 | **codeChurnAvg** | churn_history | 0.734 |
| 7 | **NAs** | size_complexity | 0.718 |
| 8 | stringLiteralsQty | ck_expressions | 0.708 |
| 9 | publicMethodsQty | ck_methods | 0.700 |
| 10 | **removedLinesAvg** | churn_history | 0.692 |
| 11 | **removedLinesCount** | churn_history | 0.690 |
| 12 | **AsD** | size_complexity | 0.683 |
| 13 | lcom* | ck_cohesion | 0.677 |
| 14 | **Magic Number Test** | test_smell | 0.669 |
| 15 | **RFC** | size_complexity | 0.656 |
| 16 | cbo | ck_coupling | 0.630 |
| 17 | **Lazy Test** | test_smell | 0.629 |
| 18 | **addLinesCount** | churn_history | 0.627 |
| 19 | **LOC** | size_complexity | 0.625 |
| 20 | totalFieldsQty | ck_fields | 0.601 |
| 21 | uniqueWordsQty | ck_misc | 0.599 |
| 22 | visibleMethodsQty | ck_methods | 0.591 |
| 23 | **Unknown Test** | test_smell | 0.581 |
| 24 | numbersQty | ck_expressions | 0.581 |
| 25 | nosi | ck_misc | 0.577 |
| 26 | cboModified | ck_coupling | 0.574 |
| 27 | loopQty | ck_expressions | 0.573 |
| 28 | **Exception Catching Throwing** | test_smell | 0.547 |
| 29 | **NOM** | size_complexity | 0.538 |
| 30 | **Assertion Roulette** | test_smell | 0.536 |
| 31 | **WMC** | size_complexity | 0.534 |
| 32 | lcom | ck_cohesion | 0.534 |
| 33 | **commits** | churn_history | 0.526 |
| 34 | **Sleepy Test** | test_smell | 0.521 |
| 35 | **General Fixture** | test_smell | 0.516 |
| 36 | tryCatchQty | ck_expressions | 0.514 |
| 37 | defaultFieldsQty | ck_fields | 0.510 |
| 38 | staticFieldsQty | ck_fields | 0.508 |
| 39 | mathOperationsQty | ck_expressions | 0.503 |
| 40 | **NumberOfMethods** | test_smell | 0.503 |
| 41 | totalMethodsQty | ck_methods | 0.500 |
| 42 | comparisonsQty | ck_expressions | 0.497 |
| 43 | **Sensitive Equality** | test_smell | 0.495 |
| 44 | **Duplicate Assert** | test_smell | 0.492 |
| 45 | **Eager Test** | test_smell | 0.491 |
| 46 | **addLinesAvg** | churn_history | 0.468 |
| 47 | **Conditional Test Logic** | test_smell | 0.456 |
| 48 | **addLinesMax** | churn_history | 0.442 |
| 49 | **IgnoredTest** | test_smell | 0.442 |
| 50 | **Resource Optimism** | test_smell | 0.409 |
| 51 | **contributorsExperience** | churn_history | 0.408 |
| 52 | **Mystery Guest** | test_smell | 0.370 |
| 53 | **EmptyTest** | test_smell | 0.352 |
| 54 | **minorContributors** | churn_history | 0.339 |
| 55 | **Contributors** | churn_history | 0.314 |
| 56 | **Print Statement** | test_smell | 0.310 |
| 57 | **Redundant Assertion** | test_smell | 0.308 |
| 58 | **Constructor Initialization** | test_smell | 0.184 |

### Global — X_sel_global (75 features = top-75 từ X_full)

Top-75 theo global ranking — khác Additive ở chỗ không buộc giữ toàn bộ X_old: loại 1 X_old feature (`Constructor Initialization`, score thấp nhất) và 2 CK features (`noc`, `defaultMethodsQty`).
Rank 1–47 giống hệt bảng Additive. Từ rank 48 trở đi:

| # | Feature | Nhóm | Score |
|---:|---|---|---:|
| 48 | modifiers | ck_misc | 0.443 |
| 49 | **addLinesMax** | churn_history | 0.442 |
| 50 | **IgnoredTest** | test_smell | 0.442 |
| 51 | staticMethodsQty | ck_methods | 0.425 |
| 52 | finalFieldsQty | ck_fields | 0.423 |
| 53 | protectedFieldsQty | ck_fields | 0.419 |
| 54 | dit | ck_inheritance | 0.412 |
| 55 | **Resource Optimism** | test_smell | 0.409 |
| 56 | **contributorsExperience** | churn_history | 0.408 |
| 57 | maxNestedBlocksQty | ck_expressions | 0.406 |
| 58 | protectedMethodsQty | ck_methods | 0.391 |
| 59 | **Mystery Guest** | test_smell | 0.370 |
| 60 | **EmptyTest** | test_smell | 0.352 |
| 61 | logStatementsQty | ck_misc | 0.352 |
| 62 | privateMethodsQty | ck_methods | 0.351 |
| 63 | privateFieldsQty | ck_fields | 0.340 |
| 64 | **minorContributors** | churn_history | 0.339 |
| 65 | lambdasQty | ck_misc | 0.319 |
| 66 | **Contributors** | churn_history | 0.314 |
| 67 | fanin | ck_coupling | 0.314 |
| 68 | returnQty | ck_expressions | 0.312 |
| 69 | **Print Statement** | test_smell | 0.310 |
| 70 | **Redundant Assertion** | test_smell | 0.308 |
| 71 | publicFieldsQty | ck_fields | 0.296 |
| 72 | innerClassesQty | ck_misc | 0.283 |
| 73 | parenthesizedExpsQty | ck_expressions | 0.281 |
| 74 | anonymousClassesQty | ck_misc | 0.275 |
| 75 | finalMethodsQty | ck_methods | 0.243 |

**3 features bị loại** (rank 76–78): `Constructor Initialization` (X_old, 0.184), `noc` (CK), `defaultMethodsQty` (CK).

**Điểm khác biệt chính giữa hai chiến lược:** Ở rank 48, Global xen `modifiers` (CK, 0.443) trước `addLinesMax` và `IgnoredTest` (X_old, 0.442) — Additive không thể làm vậy vì buộc phải giữ toàn bộ X_old. Ngoài ra Global đưa thêm 18 CK features tầm trung (rank 51–75) mà Additive bỏ qua, đổi lại loại `Constructor Initialization` (X_old yếu nhất).

---

## Top-10 CK features (combined ranking)

| # | Feature | Nhóm |
|---:|---|---|
| 1 | assignmentsQty | expressions |
| 2 | variablesQty | expressions |
| 3 | stringLiteralsQty | expressions |
| 4 | publicMethodsQty | methods |
| 5 | lcom* | cohesion |
| 6 | cbo | coupling |
| 7 | totalFieldsQty | fields |
| 8 | uniqueWordsQty | misc |
| 9 | visibleMethodsQty | methods |
| 10 | numbersQty | expressions |

Nhóm **expressions** chiếm 4/10 top features. Nhóm **inheritance** (dit, noc) đóng góp thấp nhất.

---
