# Appendix B. Supplementary Statistical Details

**Documentation version:** 2026-10-07  
**Status:** PROVISIONAL; see [Appendix A](Appendix_A_Sources_and_Coding.md) for source and coding limitations.

中文说明：本文件提供匹配规则、bootstrap、删除率与新增率置换检验、两种相似度零模型的完整设置与输出，并附可独立运行的Python代码。代码直接读取附录A的CSV，不需要私有数据库。此文件不报告已从论文正文删除的逐项Fisher检验或条件模拟拟合。

## 1. Analytical scope

Thirty recipes with 21 binary features produce 29 earlier-comparator/later-record pairs. The first recipe contributes to descriptive summaries and can be a comparator, but is not a child in rate estimation. Historical, early online, and late online groups contain 5, 9, and 15 child pairs, respectively. Period comparisons concern only the 24 online pairs; descriptive differences across all three groups are not a formal test of a continuous historical trend.

Only the retained analyses are documented here: pooled rates and uncertainty, early–late deletion/addition contrasts, two similarity null models, and source-related sensitivity checks. There is no independent-feature conditional simulation or feature-by-feature Fisher testing in the current paper analysis.

## 2. Comparator rule and estimands

Rows follow the archived analytical date order. Jaccard similarity counts shared presences relative to the union and does not reward shared zeros. Each child selects its most similar earlier observation. Ties favor the smaller year gap and then the later `sort_date`; if still tied, the lexicographically largest record identifier is used. Within this chronological corpus, this favors the most recent earlier record. A computed comparator is not an observed source of borrowing.

An empty/empty vector pair is assigned Jaccard 1 by the implementation; there are no empty vectors in the observed revised corpus. Deletion opportunities are features present in the comparator; addition opportunities are features absent from it. All features receive equal weight.

$$J(i,j)=\frac{\sum_{f=1}^{21}x_{if}x_{jf}}{\sum_{f=1}^{21}\max(x_{if},x_{jf})}$$

$$\widehat\mu_G=\frac{\sum_{i\in G}D_i}{\sum_{i\in G}S_i},\qquad \widehat\nu_G=\frac{\sum_{i\in G}A_i}{\sum_{i\in G}(21-S_i)},\qquad \widehat\rho_G=1-\widehat\mu_G.$$

Rates are opportunity-weighted proportions, not equally weighted means of pair-level rates, and are not annualized. Addition denotes a comparator-relative difference within the fixed codebook, not invention. Equal/common transition probabilities are a descriptive model simplification; pair dependence is not eliminated.

## 3. Bootstrap intervals

- Unit: complete child/comparator pairs, sampled with replacement, preserving counts and opportunity denominators together. Comparators are not reselected.
- Pooled rates: 5,000 repetitions per group with `random.Random(20260818)`, processed in order: all, historical, early online, late online using the same continuing generator.
- Early–late contrasts: 10,000 stratified repetitions with a fresh `random.Random(20260819)`; each repetition draws 9 early and 15 late pairs. The same draws supply both deletion and addition contrasts.
- Percentile limits: 2.5th and 97.5th percentiles, using linear interpolation at index `(N-1)*probability` in sorted values.
- Retention interval: one minus the deletion upper limit to one minus the deletion lower limit.
- Difference intervals are pointwise, not simultaneous Holm-adjusted intervals.

These intervals are conditional on the recorded data and fixed pairs. They do not include coding uncertainty, alternative matching, or all dependence due to shared comparators. Repetition does not create additional independent historical observations.

## 4. Exact period permutations and Holm adjustment

Each assignment selects 9 of the 24 online pairs as early; the remaining 15 are late. Both deletion and addition quantities and their respective denominators remain attached to each pair. Pooled rates are recalculated for all 1,307,504 assignments. The test statistic for each rate is late minus early. Two-sided p-values count assignments whose absolute difference is at least as large as observed, divided by the total number of assignments.

The supplemental implementation uses integer cross-products to compare absolute differences exactly. It reproduces the earlier addition test's result, whose floating-point implementation used tolerance 1e-15. No Monte Carlo plus-one correction is used for exhaustive enumeration.

For Holm adjustment, sort the two raw p-values. The smaller is multiplied by two and capped at one; the larger adjusted value is the maximum of the smaller adjusted value and the larger raw value. This family contains only the two period-rate tests. Retention is not tested separately because its contrast is the negative of the deletion contrast.

Deletion testing was added on 2026-10-06 after descriptive results were inspected. The comparisons are exploratory and were not preregistered confirmatory tests. Exact enumeration removes Monte Carlo error, not the exchangeability requirement. Shared parents and expanding earlier-candidate pools may compromise exchangeability; no causal period or internet effect is estimated.

## 5. Similarity null models

Both models recalculate the mean best-prior Jaccard statistic and reselect every comparator after randomization. This applies the same best-match rule to observed and randomized data.

$$T=\frac{1}{n-1}\sum_{i=2}^{n}\max_{j<i}J(i,j).$$

### 5.1 Margin-preserving swaps

Start with the observed 30×21 matrix at fixed row/date positions. Each attempt chooses two distinct rows and two distinct columns uniformly via `random.sample`. Swap only when the selected 2×2 block is checkerboard-shaped (diagonal ones and off-diagonal zeros, or the reverse). Each row count and column count therefore remains unchanged. Unsuccessful attempts count toward the schedule.

Use a fresh generator seeded 20260820, 100,000 initial attempts, then 1,000 attempts before each of 5,000 retained matrices. Retained matrices belong to one continuing chain; they are not independently restarted simulations. Row and column margins are asserted after every retained sample. No independent mixing diagnostic was performed; 100,000 attempts must not be described as proof of convergence.

The null does not enforce mutual exclusions, ingredient/process dependencies, or all plausible culinary constraints. Rejection can reflect those constraints as well as transmission-related structure. It does not control text formatting or word count: 'recipe size' here means the number of coded presences.

### 5.2 Randomized chronology

Keep each complete vector intact and shuffle the row order 10,000 times. Each shuffle starts from a copy of the observed vector list. The random generator continues from the margin-preserving run instead of being reseeded. Thus exact chronological-null draws depend on running the preceding swap stage with the documented settings.

### 5.3 Monte Carlo significance

Upper-tail p-values use the following correction, where b counts randomized statistics at least as large as observed and B is the number of randomizations. The code allows 1e-15 floating-point tolerance in the comparison.

$$p_{\mathrm{MC}}=\frac{b+1}{B+1}.$$

The value about 0.0002 is the minimum resolution with 5,000 randomizations and zero exceedances, not an exactly known tail probability. Central 95% null intervals describe randomized statistics, not confidence intervals for the observed mean. The two null tests are reported separately without a joint multiple-testing adjustment. Neither test identifies a documented transmission genealogy.

## 6. Source-related sensitivity scenarios

1. **Exclude image-dependent record:** remove XCF_000094, giving 29 recipes and 28 pairs; early/late online pairs become 9/14. Reselect all comparators. The exact period test enumerates 817,190 assignments.
2. **Restore archived ambiguous shrimp categories:** for XCF_000086 and XCF_000304, set `fermented_shrimp=0` and `dried_shrimp=1`, retaining every other correction. Recompute feature counts and reselect all comparators.

The random seeds are reset to their documented starting values for each scenario. The same 5,000/10,000 repetition settings apply. These scenarios address specific documented issues, not all coding or source uncertainty.

## 7. Numerical results

### 7.1 Pooled rate estimates

| Group | Pairs | Deletion | Retention | Addition | Deletion 95% CI | Addition 95% CI |
| --- | --- | --- | --- | --- | --- | --- |
| all | 29 | 27.778% | 72.222% | 14.642% | [22.939%, 32.543%] | [9.657%, 20.497%] |
| historical_1984_2010 | 5 | 32.558% | 67.442% | 25.806% | [21.818%, 41.860%] | [3.571%, 50.000%] |
| early_web_2011_2017 | 9 | 27.835% | 72.165% | 15.217% | [18.918%, 35.849%] | [8.888%, 21.212%] |
| late_web_2018_2020 | 15 | 26.351% | 73.649% | 10.180% | [19.580%, 33.113%] | [6.666%, 13.939%] |

Overall mean coded feature count is 8.666667; historical, early online, and late online means are 8.333333, 9.333333, and 8.400000. Overall retention's derived 95% interval is 67.457042%–77.061160%.

### 7.2 Deletion and addition contrasts

All differences and interval endpoints below are percentage points (pp). Raw p-values are two-sided exact tests; Holm adjustment is within each scenario's two-rate family.

| Scenario | Rate | Early count/opportunities | Late count/opportunities | Difference (pp) | 95% CI (pp) | Raw p | Holm p |
| --- | --- | --- | --- | --- | --- | --- | --- |
| main_provisional_30 | deletion | 27/97 | 39/148 | -1.484 | [-11.878, 9.738] | 0.802216284 | 0.802216284 |
| main_provisional_30 | addition | 14/92 | 17/167 | -5.038 | [-11.905, 2.124] | 0.166905799 | 0.333811598 |
| exclude_image_dependent_record | deletion | 27/97 | 37/140 | -1.406 | [-12.352, 10.150] | 0.817403541 | 0.817403541 |
| exclude_image_dependent_record | addition | 14/92 | 16/154 | -4.828 | [-11.925, 2.418] | 0.202671349 | 0.405342699 |
| retain_legacy_ambiguous_shrimp | deletion | 27/97 | 39/145 | -0.938 | [-11.385, 10.262] | 0.879172071 | 0.879172071 |
| retain_legacy_ambiguous_shrimp | addition | 14/92 | 20/170 | -3.453 | [-10.095, 3.463] | 0.294061815 | 0.588123631 |

### 7.3 Similarity null results

| Scenario | Null | Observed | Null mean | Central 95% null interval | Upper-tail p |
| --- | --- | --- | --- | --- | --- |
| main_provisional_30 | margin_preserving_feature_swap | 0.624124928 | 0.571719206 | [0.547032587, 0.597362906] | 0.000199960 |
| main_provisional_30 | chronology_permutation | 0.624124928 | 0.609900867 | [0.589769816, 0.629014206] | 0.077792221 |
| exclude_image_dependent_record | margin_preserving_feature_swap | 0.622605581 | 0.567271481 | [0.542480321, 0.593071686] | 0.000199960 |
| exclude_image_dependent_record | chronology_permutation | 0.622605581 | 0.609194725 | [0.589086437, 0.629137097] | 0.092090791 |
| retain_legacy_ambiguous_shrimp | margin_preserving_feature_swap | 0.612462740 | 0.568757851 | [0.544407339, 0.594903126] | 0.000999800 |
| retain_legacy_ambiguous_shrimp | chronology_permutation | 0.612462740 | 0.602583930 | [0.582561611, 0.621309468] | 0.165483452 |

### 7.4 Selected main-analysis pairs

| Child | Earlier comparator | Present opportunities | Deletion count | Absent opportunities | Addition count | Jaccard |
| --- | --- | --- | --- | --- | --- | --- |
| `R1997_JINYUECHI` | `R1984_PU_COMPLETE` | 5 | 2 | 16 | 2 | 0.428571 |
| `R2001_PATENT_INDUSTRIAL` | `R1997_JINYUECHI` | 5 | 1 | 16 | 7 | 0.333333 |
| `R2009_PATENT_HOME` | `R2001_PATENT_INDUSTRIAL` | 11 | 5 | 10 | 0 | 0.545455 |
| `R2010_BLOG_MAY` | `R2001_PATENT_INDUSTRIAL` | 11 | 4 | 10 | 0 | 0.636364 |
| `R2010_BLOG_DEC` | `R2001_PATENT_INDUSTRIAL` | 11 | 2 | 10 | 7 | 0.500000 |
| `XCF_000153` | `R2009_PATENT_HOME` | 6 | 0 | 15 | 3 | 0.666667 |
| `XCF_000126` | `R2010_BLOG_DEC` | 16 | 5 | 5 | 1 | 0.647059 |
| `XCF_000018` | `XCF_000126` | 12 | 3 | 9 | 3 | 0.600000 |
| `XCF_000041` | `R2010_BLOG_MAY` | 7 | 2 | 14 | 3 | 0.500000 |
| `XCF_000184` | `XCF_000126` | 12 | 6 | 9 | 1 | 0.461538 |
| `XCF_000411` | `XCF_000126` | 12 | 2 | 9 | 1 | 0.769231 |
| `XCF_000374` | `XCF_000018` | 12 | 5 | 9 | 1 | 0.538462 |
| `XCF_000206` | `XCF_000411` | 11 | 3 | 10 | 1 | 0.666667 |
| `XCF_000425` | `XCF_000153` | 9 | 1 | 12 | 0 | 0.888889 |
| `XCF_000025` | `XCF_000018` | 12 | 6 | 9 | 1 | 0.461538 |
| `XCF_000036` | `XCF_000126` | 12 | 2 | 9 | 2 | 0.714286 |
| `XCF_000010` | `XCF_000411` | 11 | 3 | 10 | 1 | 0.666667 |
| `XCF_000175` | `XCF_000126` | 12 | 3 | 9 | 1 | 0.692308 |
| `XCF_000391` | `XCF_000206` | 9 | 4 | 12 | 0 | 0.555556 |
| `XCF_000021` | `XCF_000010` | 9 | 4 | 12 | 2 | 0.454545 |
| `XCF_000086` | `XCF_000036` | 12 | 2 | 9 | 0 | 0.833333 |
| `XCF_000305` | `XCF_000374` | 8 | 1 | 13 | 3 | 0.636364 |
| `XCF_000091` | `R2001_PATENT_INDUSTRIAL` | 11 | 4 | 10 | 1 | 0.583333 |
| `XCF_000094` | `XCF_000425` | 8 | 2 | 13 | 1 | 0.666667 |
| `XCF_000040` | `XCF_000036` | 12 | 1 | 9 | 1 | 0.846154 |
| `XCF_000227` | `XCF_000175` | 10 | 2 | 11 | 1 | 0.727273 |
| `XCF_000083` | `XCF_000374` | 8 | 3 | 13 | 1 | 0.555556 |
| `XCF_000407` | `XCF_000184` | 7 | 1 | 14 | 2 | 0.666667 |
| `XCF_000304` | `R2010_BLOG_MAY` | 7 | 1 | 14 | 0 | 0.857143 |

## 8. Reproduction instructions

The following Python block is a self-contained, standard-library implementation adapted from the existing analysis functions. It does not use the original database or rerun any source scraping. It reconstructs the selected pairs from Appendix A and calculates the retained analyses. Source coding must still be judged from the evidence; a successful rerun does not independently validate source interpretation.

1. Keep both Markdown documents in the same directory.
2. Copy the Python block below into `reproduce_kimchi.py`.
3. Use Python 3.10 or later and run the full command below. It creates a new JSON output and refuses to overwrite an existing file.
4. A complete run includes both sensitivity scenarios. The `--main-only` option runs only the 30-record analysis. `--skip-nulls` is a faster rate-only check and does **not** reproduce similarity tests.
5. Compare generated output with section 7. Randomized outputs can depend on runtime implementation/version; archive the Python version with a published release.

```bash
python3 reproduce_kimchi.py Appendix_A_Sources_and_Coding.md reproduced_results.json
```

```python
"""Reproduce the retained kimchi analyses from the CSV in Appendix A.
Python 3.10+; standard library only. No database, network, or source downloads.
Results remain provisional because XCF_000094 has unverified process zeros.
"""
from __future__ import annotations
import argparse, copy, csv, io, itertools, json, math, random, statistics
from pathlib import Path

MODEL_FEATURES = ["fish_sauce","fermented_shrimp","dried_shrimp","korean_chili_powder","generic_chili_powder_only","korean_chili_paste","soy_sauce","msg_or_stock_powder","glutinous_rice_paste","wheat_flour_paste","sugar","apple","pear","alcohol","tomato_product","room_temperature_stage","cool_or_refrigerated_stage","quick_or_immediate","long_fermentation","cut_into_pieces","whole_or_quartered"]
FEATURE_LABELS = {"fish_sauce":"鱼露/鱼酱","fermented_shrimp":"盐渍/发酵虾调味料","dried_shrimp":"虾皮/海米","korean_chili_powder":"明确韩式辣椒粉","generic_chili_powder_only":"普通辣椒粉（未标韩式）","korean_chili_paste":"韩式辣椒酱","soy_sauce":"酱油/生抽","msg_or_stock_powder":"味精/鸡精/高汤粉","glutinous_rice_paste":"糯米糊","wheat_flour_paste":"面粉/非糯米淀粉糊","sugar":"添加甜味料","apple":"苹果","pear":"梨","alcohol":"酒类","tomato_product":"番茄制品","room_temperature_stage":"室温发酵阶段","cool_or_refrigerated_stage":"低温/冷藏阶段","quick_or_immediate":"即食/隔夜/24小时","long_fermentation":"长期发酵（至少一周）","cut_into_pieces":"切块制作","whole_or_quartered":"完整叶片/整棵/对半/四开制作"}
BOOTSTRAP_REPLICATES = 5000
COPY_NULL_REPLICATES = 5000
COPY_NULL_BURN_IN_ATTEMPTS = 100000
COPY_NULL_ATTEMPTS_BETWEEN = 1000
CHRONOLOGY_NULL_REPLICATES = 10000
COPY_NULL_SEED = 20260820


def jaccard(a: dict[str, int], b: dict[str, int]) -> float:
    shared = sum(bool(a[f]) and bool(b[f]) for f in MODEL_FEATURES)
    union = sum(bool(a[f]) or bool(b[f]) for f in MODEL_FEATURES)
    return shared / union if union else 1.0


def infer_parent_edges(rows: list[dict]) -> list[dict]:
    edges: list[dict] = []
    for index, child in enumerate(rows):
        if index == 0:
            continue
        candidates = []
        for parent in rows[:index]:
            similarity = jaccard(child, parent)
            year_gap = child["year"] - parent["year"]
            candidates.append((similarity, -year_gap, parent["sort_date"], parent["record_id"], parent))
        _, _, _, _, parent = max(candidates, key=lambda item: item[:4])
        shared = sum(parent[f] and child[f] for f in MODEL_FEATURES)
        deleted = sum(parent[f] and not child[f] for f in MODEL_FEATURES)
        added = sum(not parent[f] and child[f] for f in MODEL_FEATURES)
        union = shared + deleted + added
        parent_size = sum(parent[f] for f in MODEL_FEATURES)
        child_size = sum(child[f] for f in MODEL_FEATURES)
        edges.append(
            {
                "child_id": child["record_id"],
                "child_title": child["title"],
                "child_year": child["year"],
                "child_period": child["period"],
                "parent_id": parent["record_id"],
                "parent_title": parent["title"],
                "parent_year": parent["year"],
                "year_gap": child["year"] - parent["year"],
                "parent_feature_count": parent_size,
                "child_feature_count": child_size,
                "shared_count": shared,
                "deleted_count": deleted,
                "added_count": added,
                "jaccard": shared / union if union else 1.0,
                "deletion_rate": deleted / parent_size if parent_size else 0.0,
                "addition_rate": added / (len(MODEL_FEATURES) - parent_size) if parent_size < len(MODEL_FEATURES) else 0.0,
                "deleted_features": "; ".join(FEATURE_LABELS[f] for f in MODEL_FEATURES if parent[f] and not child[f]),
                "added_features": "; ".join(FEATURE_LABELS[f] for f in MODEL_FEATURES if not parent[f] and child[f]),
            }
        )
    return edges


def estimate_rates(edges: list[dict]) -> tuple[float, float]:
    deletion_opportunities = sum(edge["parent_feature_count"] for edge in edges)
    addition_opportunities = sum(len(MODEL_FEATURES) - edge["parent_feature_count"] for edge in edges)
    mu = sum(edge["deleted_count"] for edge in edges) / deletion_opportunities
    nu = sum(edge["added_count"] for edge in edges) / addition_opportunities
    return mu, nu


def percentile(values: list[float], probability: float) -> float:
    ordered = sorted(values)
    index = (len(ordered) - 1) * probability
    low = math.floor(index)
    high = math.ceil(index)
    if low == high:
        return ordered[low]
    return ordered[low] * (high - index) + ordered[high] * (index - low)


def bootstrap_rates(edges: list[dict], rng: random.Random) -> dict[str, float]:
    mu_values: list[float] = []
    nu_values: list[float] = []
    for _ in range(BOOTSTRAP_REPLICATES):
        sample = [rng.choice(edges) for _ in edges]
        mu, nu = estimate_rates(sample)
        mu_values.append(mu)
        nu_values.append(nu)
    return {
        "mu_ci_low": percentile(mu_values, 0.025),
        "mu_ci_high": percentile(mu_values, 0.975),
        "nu_ci_low": percentile(nu_values, 0.025),
        "nu_ci_high": percentile(nu_values, 0.975),
    }


def model_parameters(edges: list[dict], rng: random.Random) -> list[dict]:
    groups = [("all", edges)]
    groups.extend((period, [edge for edge in edges if edge["child_period"] == period]) for period in (
        "historical_1984_2010", "early_web_2011_2017", "late_web_2018_2020"
    ))
    results = []
    for group, group_edges in groups:
        mu, nu = estimate_rates(group_edges)
        intervals = bootstrap_rates(group_edges, rng)
        results.append(
            {
                "parameter_group": group,
                "edge_count": len(group_edges),
                "mu_deletion": mu,
                "retention_one_minus_mu": 1 - mu,
                "nu_addition": nu,
                **intervals,
            }
        )
    return results


def matrix_masks(matrix: list[list[int]]) -> list[int]:
    return [sum(value << column for column, value in enumerate(row)) for row in matrix]


def mean_best_prior_jaccard(masks: list[int]) -> float:
    best_similarities = []
    for child_index in range(1, len(masks)):
        child = masks[child_index]
        candidates = []
        for parent in masks[:child_index]:
            union_size = (child | parent).bit_count()
            candidates.append((child & parent).bit_count() / union_size if union_size else 1.0)
        best_similarities.append(max(candidates))
    return statistics.mean(best_similarities)


def attempt_margin_preserving_swaps(matrix: list[list[int]], attempts: int, rng: random.Random) -> int:
    """Perform 2x2 swaps that preserve every row sum and column sum."""
    accepted = 0
    for _ in range(attempts):
        row_a, row_b = rng.sample(range(len(matrix)), 2)
        col_a, col_b = rng.sample(range(len(matrix[0])), 2)
        a = matrix[row_a][col_a]
        b = matrix[row_a][col_b]
        c = matrix[row_b][col_a]
        d = matrix[row_b][col_b]
        if a == d and b == c and a != b:
            matrix[row_a][col_a] = b
            matrix[row_a][col_b] = a
            matrix[row_b][col_a] = a
            matrix[row_b][col_b] = b
            accepted += 1
    return accepted


def copy_structure_null_tests(rows: list[dict]) -> list[dict]:
    """Test whether max-prior similarity exceeds two selection-aware nulls."""
    original_matrix = [[row[feature] for feature in MODEL_FEATURES] for row in rows]
    observed = mean_best_prior_jaccard(matrix_masks(original_matrix))
    original_row_sums = [sum(row) for row in original_matrix]
    original_column_sums = [sum(row[column] for row in original_matrix) for column in range(len(MODEL_FEATURES))]
    rng = random.Random(COPY_NULL_SEED)

    randomized = [row[:] for row in original_matrix]
    accepted = attempt_margin_preserving_swaps(randomized, COPY_NULL_BURN_IN_ATTEMPTS, rng)
    margin_null: list[float] = []
    for _ in range(COPY_NULL_REPLICATES):
        accepted += attempt_margin_preserving_swaps(randomized, COPY_NULL_ATTEMPTS_BETWEEN, rng)
        if [sum(row) for row in randomized] != original_row_sums:
            raise RuntimeError("Margin-preserving null changed recipe feature counts")
        if [sum(row[column] for row in randomized) for column in range(len(MODEL_FEATURES))] != original_column_sums:
            raise RuntimeError("Margin-preserving null changed feature prevalences")
        margin_null.append(mean_best_prior_jaccard(matrix_masks(randomized)))

    observed_masks = matrix_masks(original_matrix)
    chronology_null: list[float] = []
    for _ in range(CHRONOLOGY_NULL_REPLICATES):
        shuffled = observed_masks[:]
        rng.shuffle(shuffled)
        chronology_null.append(mean_best_prior_jaccard(shuffled))

    output = []
    definitions = [
        (
            "margin_preserving_feature_swap",
            margin_null,
            COPY_NULL_REPLICATES,
            "每条食谱的特征数、每种特征的总出现次数、真实年代顺序",
            f"2×2交换；预热{COPY_NULL_BURN_IN_ATTEMPTS}次尝试；样本间{COPY_NULL_ATTEMPTS_BETWEEN}次尝试；累计接受{accepted}次交换",
        ),
        (
            "chronology_permutation",
            chronology_null,
            CHRONOLOGY_NULL_REPLICATES,
            "每条食谱的完整特征向量与全部两两相似度",
            f"随机打乱{len(rows)}条食谱的年代顺序，再应用相同的‘最相似早期母本’选择规则",
        ),
    ]
    for test_id, null_values, replicates, constraints, method in definitions:
        null_mean = statistics.mean(null_values)
        p_value = (1 + sum(value >= observed - 1e-15 for value in null_values)) / (replicates + 1)
        interpretation = (
            "观察到的最相似早期母本Jaccard显著高于该零模型。"
            if p_value < 0.05
            else "观察值未显著高于该零模型；不能仅凭最高相似母本宣称存在复制谱系。"
        )
        output.append({
            "test_id": test_id,
            "statistic": "mean_max_prior_jaccard",
            "observed": observed,
            "null_mean": null_mean,
            "null_sd": statistics.stdev(null_values),
            "null_ci_low": percentile(null_values, 0.025),
            "null_ci_high": percentile(null_values, 0.975),
            "observed_minus_null_mean": observed - null_mean,
            "p_one_sided": p_value,
            "replicates": replicates,
            "random_seed": COPY_NULL_SEED,
            "constraints_preserved": constraints,
            "method": method,
            "interpretation": interpretation,
        })
    return output


def analyze(edges):
    early = [e for e in edges if e['child_period'] == 'early_web_2011_2017']
    late = [e for e in edges if e['child_period'] == 'late_web_2018_2020']
    observed = [b-a for a, b in zip(estimate_rates(early), estimate_rates(late))]
    rng = random.Random(20260819)
    samples = [[], []]
    for _ in range(10000):
        es = [rng.choice(early) for _ in early]
        ls = [rng.choice(late) for _ in late]
        for k, (a, b) in enumerate(zip(estimate_rates(es), estimate_rates(ls))):
            samples[k].append(b-a)
    web = early + late
    # Integer cross-products implement the absolute-difference comparison exactly.
    specs = []
    for name, count_key in [('deletion', 'deleted_count'), ('addition', 'added_count')]:
        counts = [e[count_key] for e in web]
        opp = [e['parent_feature_count'] if name == 'deletion' else 21-e['parent_feature_count'] for e in web]
        ec, eo = sum(counts[:len(early)]), sum(opp[:len(early)])
        tc, to = sum(counts), sum(opp)
        specs.append(dict(name=name, counts=counts, opp=opp, ec=ec, eo=eo, tc=tc, to=to,
                          obs_num=abs(tc*eo-ec*to), obs_den=eo*(to-eo), extreme=0))
    assignments = math.comb(len(web), len(early))
    for selected in itertools.combinations(range(len(web)), len(early)):
        for s in specs:
            ec = sum(s['counts'][j] for j in selected)
            eo = sum(s['opp'][j] for j in selected)
            assert 0 < eo < s['to']
            num = abs(s['tc']*eo-ec*s['to'])
            den = eo*(s['to']-eo)
            s['extreme'] += num*s['obs_den'] >= s['obs_num']*den
    results = []
    for k, s in enumerate(specs):
        results.append(dict(rate=s['name'], early_count=s['ec'], early_opportunities=s['eo'],
            late_count=s['tc']-s['ec'], late_opportunities=s['to']-s['eo'],
            early_rate=s['ec']/s['eo'], late_rate=(s['tc']-s['ec'])/(s['to']-s['eo']),
            delta=observed[k], bootstrap_95_ci=[percentile(samples[k], .025), percentile(samples[k], .975)],
            p_two_sided=s['extreme']/assignments, extreme_assignments=s['extreme']))
    running = 0
    for rank, k in enumerate(sorted(range(2), key=lambda k: results[k]['p_two_sided'])):
        running = max(running, min(1, (2-rank)*results[k]['p_two_sided']))
        results[k]['p_holm_two_rate_family'] = running
    return dict(early_pairs=len(early), late_pairs=len(late), assignments=assignments,
                bootstrap_repetitions=10000, bootstrap_seed=20260819, results=results)


def read_rows(path):
    text = path.read_text(encoding="utf-8")
    blocks = text.split("```csv\n")
    if len(blocks) != 2:
        raise ValueError("Appendix A must contain exactly one CSV block.")
    payload = blocks[1].split("\n```", 1)[0]
    rows = list(csv.DictReader(io.StringIO(payload)))
    numeric = ["chronological_order", "year", "feature_count"] + MODEL_FEATURES
    for row in rows:
        for key in numeric:
            row[key] = int(row[key])
        assert all(row[f] in (0, 1) for f in MODEL_FEATURES)
        assert row["feature_count"] == sum(row[f] for f in MODEL_FEATURES)
        assert row["korean_chili_powder"] + row["generic_chili_powder_only"] <= 1
    assert len(rows) == 30 and len({r["record_id"] for r in rows}) == 30
    assert [r["chronological_order"] for r in rows] == list(range(1, 31))
    assert [r["sort_date"] for r in rows] == sorted(r["sort_date"] for r in rows)
    return rows


def summaries(rows):
    groups = [("all", rows)]
    groups += [(period, [r for r in rows if r["period"] == period]) for period in
               ["historical_1984_2010", "early_web_2011_2017", "late_web_2018_2020"]]
    return [dict(group=name, n=len(group),
                 mean_feature_count=statistics.mean(r["feature_count"] for r in group),
                 counts={f: sum(r[f] for r in group) for f in MODEL_FEATURES})
            for name, group in groups]


def check_main(result, with_nulls):
    # Checks use unrounded archived values, not rounded manuscript percentages.
    overall = result["parameters"][0]
    assert math.isclose(overall["mu_deletion"], 0.2777777777777778, abs_tol=1e-12)
    assert math.isclose(overall["nu_addition"], 0.14641744548286603, abs_tol=1e-12)
    expected = {
        "deletion": (-0.01483700195040405, 0.8022162838507568,
                     [-0.11877606082904096, 0.09737828335195331]),
        "addition": (-0.05037750585784953, 0.16690579914095865,
                     [-0.11904849957734573, 0.021238228811861354])
    }
    for r in result["rate_contrasts"]["results"]:
        delta, p, ci = expected[r["rate"]]
        assert math.isclose(r["delta"], delta, abs_tol=1e-12)
        assert math.isclose(r["p_two_sided"], p, abs_tol=1e-12)
        assert all(math.isclose(x, y, abs_tol=1e-12)
                   for x, y in zip(r["bootstrap_95_ci"], ci))
    if with_nulls:
        assert math.isclose(result["null_tests"][0]["observed"],
                            0.6241249284859832, abs_tol=1e-12)


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("appendix_a", type=Path)
    parser.add_argument("output", type=Path)
    parser.add_argument("--main-only", action="store_true",
                        help="Run only the 30-record analysis.")
    parser.add_argument("--skip-nulls", action="store_true",
                        help="Check rates only; this does not reproduce similarity tests.")
    args = parser.parse_args()
    if args.output.exists():
        raise FileExistsError("Choose a new output filename; existing files are not overwritten.")
    original = read_rows(args.appendix_a)
    names = ["main_provisional_30"]
    if not args.main_only:
        names += ["exclude_image_dependent_record", "retain_legacy_ambiguous_shrimp"]
    outputs = {}
    for name in names:
        rows = copy.deepcopy(original)
        if name == "exclude_image_dependent_record":
            rows = [r for r in rows if r["record_id"] != "XCF_000094"]
        elif name == "retain_legacy_ambiguous_shrimp":
            for row in rows:
                if row["record_id"] in ["XCF_000086", "XCF_000304"]:
                    row["fermented_shrimp"] = 0
                    row["dried_shrimp"] = 1
                    row["feature_count"] = sum(row[f] for f in MODEL_FEATURES)
        print("Computing:", name, flush=True)
        edges = infer_parent_edges(rows)
        parameters = model_parameters(edges, random.Random(20260818))
        contrasts = analyze(edges)
        null_tests = [] if args.skip_nulls else copy_structure_null_tests(rows)
        outputs[name] = dict(n=len(rows), descriptive=summaries(rows),
                             edges=edges, parameters=parameters,
                             rate_contrasts=contrasts, null_tests=null_tests)
        if name == "main_provisional_30":
            check_main(outputs[name], not args.skip_nulls)
        print(json.dumps(dict(scenario=name, rates=contrasts["results"],
                              null_tests=null_tests), ensure_ascii=False), flush=True)
    result = dict(status="PROVISIONAL: XCF_000094 procedural images remain unverified",
                  pointwise_intervals=True,
                  holm_family="Two period-rate contrasts within each scenario",
                  nulls_skipped=args.skip_nulls, scenarios=outputs)
    with args.output.open("x", encoding="utf-8") as f:
        json.dump(result, f, ensure_ascii=False, indent=2)
    print("Saved:", args.output, flush=True)


if __name__ == "__main__":
    main()

```

## 9. Interpretation and reporting

**Reproduction check (2026-10-07):** The embedded code was run with Python 3.12.14 against the CSV in Appendix A. All 630 feature values matched the archived revised matrix. Pooled bootstrap outputs, deletion/addition contrasts and their Holm adjustments, and both similarity null outputs matched the archived results in all three scenarios. This is an implementation consistency check, not independent source verification or statistical validation of the assumptions.

Main findings: lower observed early–late deletion/addition contrasts are not statistically established; observed similarity exceeds the specified margin-preserving null but is not significant at .05 against randomized chronology. Avoid claims that formatting was controlled, that the internet caused convergence, or that actual copying was proven. The unfinished image-based verification keeps the 30-record analysis provisional.

These documents were prepared with AI assistance in source review, analysis coding, and drafting. This is not independent double coding or an independent researcher's replication. The release should identify the researcher's actual contributions and any subsequent human verification without claiming checks that did not occur.

## 10. Statistical references

- Efron, B. (1979). Bootstrap methods: Another look at the jackknife. *The Annals of Statistics, 7*(1), 1–26. https://doi.org/10.1214/aos/1176344552
- Holm, S. (1979). A simple sequentially rejective multiple test procedure. *Scandinavian Journal of Statistics, 6*(2), 65–70. https://www.jstor.org/stable/4615733
