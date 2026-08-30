# P&S Benford Project

Mini-research project for a Probability Theory and Statistics course, testing whether
**Benford's Law** can distinguish fair elections from fraudulent ones, and whether it
holds equally well in urban vs. rural precincts.

**Team:** Andrii Kulbaba · Nestor Leyko · Bohdan Pavlunyshyn

## What's in here

| File | Description |
|---|---|
| `Benford.Rmd` | The full analysis: data loading, Benford's Law tests, chi-squared tests, bootstrap, and plots. |
| `voting_data_geo_eng.csv` | Russia, 2018 presidential election, per-polling-station results. |
| `first_round_results_from_district_electoral_comissions.csv` | Poland, 2025 presidential election (first round), per-polling-station results. |
| `Protocol_data.csv` | Belarus, 2020 presidential election, per-polling-station results. |

## Background

Benford's Law states that in many naturally occurring datasets, the leading digit `d`
of a number follows

$$P(d) = \log_{10}\left(1+\frac{1}{d}\right), \quad d \in \{1, \dots, 9\}$$

so `1` appears as a leading digit ~30% of the time and `9` less than 5% of the time —
far from the ~11.1% a uniform distribution would give. Because the law holds
independently of the unit system, deviations from it are sometimes used as a
red flag for manipulated data, including election results. We treat the random
variable $X$ as *the vote count for a specific candidate in a specific precinct*,
and check how closely the leading (and second) digits of $X$ follow Benford's Law.

## Tests

**Test 1 — Were the elections fair?** Compares the leading/second-digit distributions
of vote counts from Poland 2025 (commonly regarded as a fair election) against Russia
2018 and Belarus 2020 (both widely reported as fraudulent), using a chi-squared
goodness-of-fit test against Benford's Law. Because the three countries have very
different numbers of precincts, we also resample 5,000 precincts per country for a
size-normalized comparison.

**Test 2 — Does precinct size matter (urban vs. rural)?** Uses only the Polish data,
split into urban (`miasto`) and rural (`wieś`) precincts, and tests whether urban
precincts (which span a wider range of vote counts) conform to Benford's Law more
closely than rural ones — via both a chi-squared test and a 1,000-iteration bootstrap
on the difference in chi-squared statistics.

## Results

Chi-squared goodness-of-fit against Benford's Law, first digit (df = 8) and second
digit (df = 9), on the full per-precinct data:

| Country | First digit χ² | Second digit χ² | p-value |
|---|---|---|---|
| Poland (2025) | 8628.0 | 518.7 | < 2.2e-16 |
| Russia (2018) | 7204.0 | 766.9 | < 2.2e-16 |
| Belarus (2020) | 152.1 | 16.9 | < 2.2e-16 (first) / 0.050 (second) |

Resampled to 5,000 precincts per country (`set.seed(91)`), first digit only:

| Country | χ² | p-value |
|---|---|---|
| Poland | 195.6 | < 2.2e-16 |
| Russia | 362.0 | < 2.2e-16 |
| Belarus | 126.7 | < 2.2e-16 |

All three countries — including Poland, the "fair" baseline — reject the null
hypothesis of conforming to Benford's Law at any reasonable significance level. This
is a known limitation of naive Benford chi-squared tests on large `n`: with thousands
of precincts, even small, practically irrelevant deviations from the theoretical
distribution become statistically significant. The first-digit χ² test alone is not
enough here to separate "fair" from "fraudulent" — the second-digit gap between
Belarus (p = 0.050, essentially borderline-consistent with Benford) and the other two
countries is the more informative signal in this dataset.

**Test 2 (urban vs. rural, Poland only):** Both groups reject the Benford null
(miejski χ² = 4772.8, wiejski χ² = 494.7, both p ≈ 0). The bootstrap comparison gives
an observed difference of −0.344 and a bootstrap p-value of 0.492 — **not**
significant at α = 0.05, so we do not reject $H_0$: the data does not support the
hypothesis that urban precincts conform to Benford's Law more closely than rural ones.

## Known data issue

The Poland CSV (`first_round_results_from_district_electoral_comissions.csv`) has a
mislabeled header: the column named `Voivodeship` actually contains each polling
station's address/name, while the real voivodeship names (`dolnośląskie`, etc.) sit
under the column labeled `County ID`. This is a data quality issue in the source file
itself (verified — every row has a consistent, well-formed 45-field structure, so it
is not a CSV-parsing artifact). It doesn't affect the analysis in this notebook (which
works per-precinct and never groups by `Voivodeship`), but keep it in mind if you
extend this project to do actual region-level breakdowns for Poland.

## Running it

Requires **R** (≥ 4.x) with the `reticulate`, `dplyr`, `rmarkdown`, and `ggplot2`
packages, and a working Python 3 available to `reticulate` (it will provision an
isolated virtual environment and install `pandas` automatically on first run).

```r
install.packages(c("reticulate", "dplyr", "rmarkdown", "ggplot2"))
rmarkdown::render("Benford.Rmd")
```

or open `Benford.Rmd` in RStudio and click **Knit**. All three CSV files must stay in
the project root, as the notebook loads them by relative path.
