# County-wise PCA variable screening (Python)

This independent workflow adapts the workbook sheet names, six weather variables,
and standardized county-wise PCA approach from `../PCA/code/pca_analysis.R`.
It starts with **all six variables**, without copying the R script's predetermined
variable removals. PM2.5 is not used to construct or rank the components.

## Run

From the repository root, with Python 3.10 or newer:

```sh
python3 -m venv PCA_SR/.venv
source PCA_SR/.venv/bin/activate
python -m pip install -r PCA_SR/requirements.txt
python PCA_SR/pca_variable_ranking.py '/path/to/workbook.xlsx'
# Optional subset (use original workbook sheet names):
python PCA_SR/pca_variable_ranking.py '/path/to/workbook.xlsx' --counties Baldwin Morgan
```

The input workbook is not included. Its sheets and column names must match the
original R workflow. Sheets: Baldwin, Clay, Dekalb, Etohaw, Jefferson_NB, Mobile,
Montgomery, Morgan, Sumter, Tuscaloosa. Columns: Temperature, Dew Point,
Relative Humidity, Wind Speed, Pressure, Precipitation. PM2.5 is optional and ignored.
No raw observations are exported or uploaded.

## What the ranking means

Each county is fitted independently. Variables are centered and scaled using
sample standard deviations, matching R's `prcomp(center=TRUE, scale.=TRUE)`.
SVD gives eigenvectors v and eigenvalues lambda. Keep the smallest number of PCs
explaining at least 80% of variance (change with `--variance-target 0.90`).

For variable j, retained communality is:

`h_j² = sum over retained PCs k of lambda_k * v_jk²`

The reported importance is `100 * h_j² / sum_j(h_j²)`, descending. It is the
variable's share of variance represented by the retained components. Correlation
loadings are `sqrt(lambda_k) * v_jk`, not just the eigenvectors reported as
loadings in the R script. Signs do not affect rankings.

**This is PCA contribution, not statistical significance, causal importance, or
predictive importance for PM2.5.** PCA alone cannot establish that a variable is
irrelevant to pollution. A low-ranked variable can still predict PM2.5 well.
A separate supervised analysis with time-aware validation would answer that question.
PCA maximizes variance; see the [PCA documentation](https://scikit-learn.org/1.7/modules/generated/sklearn.decomposition.PCA.html).

## Screening flags and limitations

- `low_representation`: retained communality < 0.50 (adjust `--low-threshold`).
  This means the retained PCs represent less than half of that variable's variance.
- `high_redundancy`: absolute correlation with another variable >= 0.90
  (adjust `--redundancy-threshold`). Both members are flagged; neither is deleted.
  Correlated duplicate variables can both rank highly.
- `constant`, `all_missing`, or `constant_on_complete_rows`: cannot enter scaled
  PCA; listed after ranked variables with no numerical rank or importance.

Thresholds are exploratory choices, not hypothesis tests. No variable is removed
based on rank. Compare rankings under multiple variance targets before making a
selection. When all PCs are retained, all standardized variables have communality
1 and equal importance; the script reports this limitation. Near ties (10 decimal
places in importance) share a rank. Rankings have no confidence intervals and may
be unstable when eigenvalues are close or sample sizes are small.

All-missing and constant columns are excluded first. Remaining missing weather
observations are handled by complete-case deletion, with counts in the output.
Constants arising on that complete-case subset are then excluded without recovering
previously dropped rows. Non-numeric and infinite values fail explicitly. At least
two varying variables and more complete observations than variables are required.
No imputation, KMO/Bartlett tests, or PM2.5 regression is performed here.

## Outputs

Default output: `PCA_SR/results/` (override with `--output PATH`).

Each county receives `variable_ranking.csv` (most to least contribution),
`variable_ranking.png`, `explained_variance.csv`, `correlation_loadings.csv`, and
`weather_correlations.csv`. The combined `all_county_rankings.csv` keeps each
county's ranking separate. `run_summary.json` records thresholds, row counts,
excluded variables, retained variance, and software versions.

Generated results and Excel files are ignored by Git. Running again overwrites
matching outputs; use a distinct output folder when comparing settings. Existing
outputs from counties not selected in a later run are left in place.

## Tests

```sh
python -m unittest discover -s PCA_SR -p 'test_*.py' -v
```

Tests use synthetic data; actual study rankings require the original workbook.
