# Reproducibility package: Worst-Case Contagion in Financial Networks

This repository contains the code, fixed input instances, reference outputs, and
paper source needed to reproduce the numerical examples and figures in
**Worst-Case Contagion in Financial Networks: MILP Clearing and Robust Loss Analysis**.

The numerical section of the manuscript uses synthetic financial-network instances
with long-only common-asset exposures.  The experiments validate fixed-shock
clearing, pre-insolvency LP-dual loss curves, post-insolvency scalar geometry,
certified lower/upper bounds, localized fixed-pattern certificates, and a 353-bank
scenario lower-bound experiment.

## Repository layout

```text
.
├── data/                      # fixed .npz input instances used by the paper
├── results/
│   ├── data/                  # reference curve/trace data used by the figures
│   ├── figures/               # reference figures in PNG and PDF
│   └── tables/                # reference tables in CSV and LaTeX
├── src/milp_kink/             # reusable clearing, LP/MILP and plotting code
├── scripts/                   # command-line reproduction scripts
├── experiments/               # reserved for user-added experiment notebooks/scripts
├── paper/                     # LaTeX source used for the revised manuscript
└── docs/MANIFEST.md           # figure/table-to-script mapping
```

The `data/*.npz` files contain the five fixed instances from Table 1:
`D-val`, `D-pre`, `M-bound`, `D-post`, and `L-353`.

## Requirements

The reference environment used in the manuscript was Python 3.13 with SciPy/HiGHS.
The code has also been tested with Python 3.11+.

Core Python dependencies:

```text
numpy
scipy
pandas
matplotlib
```

A LaTeX installation is needed only if you want to compile the manuscript in
`paper/`.

## Installation

From a fresh clone:

```bash
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
python -m pip install --upgrade pip
python -m pip install -e .
```

Alternatively, without editable installation, every script prepends `src/` to
`PYTHONPATH`, so the scripts can be run directly after installing the dependencies:

```bash
python -m pip install -r requirements.txt
```

## Quick reproduction of all paper figures

This command regenerates all figures from the distributed reference CSV files:

```bash
python scripts/reproduce_all.py
```

It writes PNG and PDF files to `results/figures/`.  To also regenerate generic
LaTeX versions of the CSV tables, run

```bash
python scripts/reproduce_all.py --with-auto-tex
```

The manuscript-formatted `.tex` tables are already included in `results/tables/`.
The `_auto.tex` files are meant as convenience exports and may not match the exact
manual formatting of the paper tables.

## Sanity checks

Run the lightweight installation check:

```bash
python scripts/check_installation.py
```

This checks the margins in Table 1, one 353-bank post-insolvency clearing
calculation, and one pre-insolvency robust LP-dual solve.

## Recomputing selected experiments

The repository provides transparent recomputation scripts.  Some randomized or
localized-certificate timings may differ across machines, but all scripts use
deterministic seeds by default.

### Table 2: fixed-shock clearing validation on `D-val`

```bash
python scripts/recompute_validation.py
```

Output: `results/tables/validation_table_recomputed.csv`.

### Figure 1/Table 3: pre-insolvency robust LP-dual curve on `D-pre`

```bash
python scripts/recompute_pre_curve.py --points 241
```

Output: `results/data/pre_curve_recomputed.csv`.

### Figure 3/Table 5: exact scalar post-insolvency curve on `D-post`

```bash
python scripts/regenerate_dpost_scalar_exact.py
```

Outputs:

```text
results/data/post_scalar_curve.csv
results/tables/post_slope_decreases.csv
results/tables/post_slope_decreases.tex
results/figures/post_insolvency_scalar_curve.{png,pdf}
```

### Figure 4/Table 6: scenario lower bounds and simple global fixed-pattern UBs on `M-bound`

```bash
python scripts/recompute_bounds_mbound.py
```

Output: `results/tables/bounds_table_recomputed_basic.csv`.

The exact manuscript tables also use greedy regime generation and localized
refinement.  The reference outputs for those steps are distributed in
`results/tables/` and `results/data/`.

### Figures 5-7/Tables 8-9: localized fixed-pattern certificate

```bash
python scripts/recompute_localized_certificate.py --instance M-bound --epsilon-over-ub 2 --splits 60
```

For a quick smoke test use fewer splits:

```bash
python scripts/recompute_localized_certificate.py --instance M-bound --epsilon-over-ub 2 --splits 5
```

### Figure 8/Table 10: 353-bank scenario lower bounds

```bash
python scripts/recompute_large_scenario.py
```

Output: `results/tables/large_scenario_table_recomputed.csv`.

### Figure 9/Table 11: localized certificates on `L-353`

A compact implementation of the localized certificate is available:

```bash
python scripts/recompute_localized_certificate.py --instance L-353 --epsilon-over-ub 1.5 --splits 5
```

The full reference counters and outputs used in the manuscript are included in
`results/tables/large_localized_bounds_table.csv`.

## Main algorithms implemented

The package implements:

- maximal clearing by monotone fixed-point iteration initialized at `pbar`;
- fixed-shock clearing LP;
- exact MILP clearing oracle using `scipy.optimize.milp`;
- default-resilience and insolvency-resilience margin computation;
- pre-insolvency robust LP-dual computation for long-only `l1` shocks;
- scenario lower bounds;
- fixed-pattern upper bounds over the adverse long-only `l1` simplex;
- a transparent localized fixed-pattern certificate via longest-edge simplex bisection;
- figure and table generation from reference outputs.

## Rebuilding the manuscript PDF

The paper source is included in `paper/`.  The LaTeX source refers to figures and
tables under `results/`, so build from the repository root using the helper script:

```bash
bash paper/build_paper.sh
```

The helper sets `TEXINPUTS=paper//:` so that the Springer class and section files
are found while the `results/` paths resolve from the repository root.  Keep the
repository layout unchanged, or adjust the figure/table paths in the LaTeX source.

## Notes on reproducibility

- Losses in the tables are normalized by `sum(pbar)` unless explicitly stated.
- Timing columns are hardware- and solver-version-dependent.
- The reference CSV files are the source of truth for reproducing the exact figures
  as shown in the manuscript.
- Recomputed randomized scenario experiments use deterministic seeds, but may not
  be bit-for-bit identical to the original timing runs on a different system.
- The localized-certificate implementation in `src/milp_kink/certificates.py` is
  deliberately compact and transparent; the distributed reference CSV files retain
  the exact counters used for the manuscript figures and tables.

## Before public GitHub release

Please choose and add the intended open-source license.  The placeholder
`LICENSE.md` currently marks the package as provided for review reproducibility.
