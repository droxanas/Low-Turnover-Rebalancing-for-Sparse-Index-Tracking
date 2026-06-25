# Low-Turnover Rebalancing for Sparse Index Tracking

SPDX-License-Identifier: GPL-3.0-or-later

This repository contains the code, processed data, and archived CSV outputs used to reproduce the empirical results for the paper **"Low-Turnover Rebalancing for Sparse Index Tracking"**.

The paper studies a generalised-Bayes workflow for sparse index tracking in which tracker construction and tracker maintenance are treated as separate decisions. The initial tracker is built once using a calibrated shrinkage model and posterior support screening. Subsequent rebalance dates are handled in the self-financing change variable `Delta w`; the default action is no trade, and bounded local repairs are implemented only when realised tracking deterioration and posterior directional evidence jointly support intervention.

This repository is intended for reproducibility of the reported case study. It is not investment advice, and it is not a production trading system.

## Repository structure

The repository is organised as follows.

```text
.
├── 01_tracker_construction.ipynb
├── 02_tracker_maintenance.ipynb
├── requirements.txt
├── LICENSE
├── README.md
├── data/
│   ├── merged_sp500_returns_2020_2025.csv
│   ├── asset_universe.csv
│   └── rolling_windows.csv
├── construction/
│   ├── construction_fit01_cgrid_summary.csv
│   ├── construction_selected_fit01_row.csv
│   ├── construction_refit_floor_audit.csv
│   ├── practical_positive_support_sensitivity.csv
│   ├── construction_mala_diagnostics.csv
│   ├── construction_mala_coordinate_ess.csv
│   ├── sapg_trace_selected_fit01.csv
│   ├── sapg_trace_summary_selected_fit01.csv
│   └── fista_trace_selected_fit01.csv
├── maintenance/
│   ├── hold_summary.csv
│   ├── decision_summary.csv
│   ├── daily_paths.csv
│   ├── full_compositions.csv
│   ├── portfolio_composition_summary.csv
│   ├── top50_compositions.csv
│   ├── recovery_candidates.csv
│   ├── recovery_state_log.csv
│   ├── recovery_funding_actions.csv
│   ├── small_weight_summary.csv
│   ├── sequential_rebalance_audit.csv
│   ├── delta_w_selected_rows.csv
│   ├── delta_w_cgrid_all_holds.csv
│   └── delta_w_mala_diagnostics.csv
├── comparisons/
│   └── te_turnover_frontier_full_horizon_data.csv
├── csv_manifest.csv
└── checksums.csv
```

The CSV files are archived at the repository root level under `construction/`, `maintenance/`, `comparisons/`, and `data/`. They are not stored under a separate `outputs/` directory.

## Data

The main input file is:

```text
data/merged_sp500_returns_2020_2025.csv
```

It contains daily returns for the fixed S&P 500-style universe used in the paper, with asset-return columns first and the benchmark column named `SP500` last. The empirical horizon is 2020-01-03 to 2025-12-31.

The supplementary files:

```text
data/asset_universe.csv
data/rolling_windows.csv
```

record the ticker universe and the locked rolling-window protocol used in the paper.

The returns were prepared from publicly available Yahoo Finance data. Users are responsible for respecting the terms of any original data providers.

## Python environment

The notebooks use standard scientific Python packages. A minimal environment can be installed with:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

The notebooks can also be run in Google Colab. Production runs involve long MALA chains and may take several hours, depending on hardware.

## Reproducing the paper results

### 1. Construct the initial tracker

Open and run:

```text
01_tracker_construction.ipynb
```

For a quick syntax and pipeline check, leave:

```python
RUN_PRODUCTION = False
```

For the paper run, set:

```python
RUN_PRODUCTION = True
```

The production construction run should select the `c = 25` construction row, produce an initial 59-name support before the one-off implementability floor, and produce a 53-name tracker after the 0.5% floor. The archived construction outputs in `construction/` reproduce the construction grid, selected row, SAPG/FISTA traces, posterior diagnostics, support sensitivity, and floor audit reported in the paper and supplement.

The construction notebook exports a tracker cache used by the maintenance notebook. The expected production cache is named along the lines of:

```text
initial_tracker_fit01_nnz150_c25_floor005_53names.npz
```

### 2. Run the sequential maintenance experiment

Open and run:

```text
02_tracker_maintenance.ipynb
```

For the paper run, set:

```python
RUN_PRODUCTION = True
```

The maintenance notebook loads the floor-applied tracker cache and then runs the H1--H16 sequential holding and recovery experiment. The archived maintenance outputs in `maintenance/` reproduce the hold-level TE path, implemented decisions, recovery diagnostics, posterior candidate logs, complete `Delta w` grids, selected grid rows, MALA diagnostics, daily realised paths, and full portfolio-composition path.

The production maintenance run should reproduce the following headline values, up to small numerical tolerance:

| Quantity | Paper value |
|---|---:|
| Mean holding-window TE | 20.556 bp |
| Median holding-window TE | 19.409 bp |
| Maximum holding-window TE | 36.074 bp |
| Total sequential one-way turnover | 5.0% |
| Final active names | 55 |

Small numerical differences can occur across machines because the production workflow contains stochastic MCMC steps, although the notebooks set random seeds and use fixed algorithmic settings for reproducibility.

## Archived CSV outputs

The archived CSV files are included so that the tables and figures can be audited without rerunning the full production chains.

- `construction/` contains construction-stage loss-scale calibration, selected-row, SAPG, FISTA, posterior-MALA, support-sensitivity, and floor-audit outputs.
- `maintenance/` contains full-horizon hold summaries, decisions, recovery states, candidate-level posterior diagnostics, complete and selected `Delta w` grid rows, MALA diagnostics, daily paths, and portfolio compositions.
- `comparisons/` contains the data used for the full-horizon tracking-error--turnover frontier figure.
- `data/` contains the processed return matrix, asset universe, and locked rolling-window definitions.
- `csv_manifest.csv` describes the archived CSV files.
- `checksums.csv` records SHA256 checksums for the archived CSV files.

## Notes on interpretation

The posterior distributions in the notebooks are generalised-Bayes decision distributions built from calibrated tracking losses. The variance-like scale is an effective loss scale or temperature, not a physical market-noise variance. Posterior probabilities are therefore used operationally as uncertainty summaries for support and rebalancing decisions.

The proposed method is designed as a low-turnover maintenance layer for an existing sparse tracker. It is not intended to dominate full sparse reconstruction baselines in tracking error. The empirical claim is a tracking-error--turnover--support-persistence trade-off: the proposed tracker gives up some tracking precision relative to the strongest full reconstruction baselines while using much lower sequential turnover and preserving a more stable sparse composition.

## Licence

Unless otherwise stated, the source code and notebooks in this repository are released under the GNU General Public License version 3, or any later version published by the Free Software Foundation (`GPL-3.0-or-later`). See `LICENSE` for the full GPLv3 licence text.

The archived CSV outputs are provided to support reproducibility of the reported case study. The input market data may be subject to the terms of the original data provider.

## Citation

If you use this repository, please cite the associated paper:

```text
Dimitrios Roxanas. Low-Turnover Rebalancing for Sparse Index Tracking.
```
