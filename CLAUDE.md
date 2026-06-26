# Frank-Hertz Lab Analysis — Project Instructions

## Project Overview

This repository contains Jupyter notebooks analysing a Frank-Hertz experiment performed as part
of a physics lab course at Ben-Gurion University (Physics B). The experimental guide is
`Franck Hertz 2024B 31.1.2024.pdf` (Hebrew).

Four sub-experiments are analysed in separate notebooks:

| Notebook | Experiment | Gas | Key measurement |
|---|---|---|---|
| `Frank_Hertz_Exp1.ipynb` | Exp 1 | Hg | I-V at room temp (Child-Langmuir) |
| `Frank_Hertz_Exp2.ipynb` | Exp 2 | Hg | I-V at 130 °C — parabolic fits, excitation energy |
| `Frank_Hertz_Exp3.ipynb` | Exp 3 | Hg | Anode current vs. temperature / mean free path |
| `Frank_Hertz_Exp4.ipynb` | Exp 4 | Ne | I-V at 5 retarding voltages — best measurement selection, excitation energy |

## Data Files

Naming convention:

| Pattern | Meaning |
|---|---|
| `exp2m1.csv`, `exp2m2.csv` | Exp 2, measurement index |
| `exp3m1.csv` | Exp 3, measurement index |
| `exp4m1-10V.csv` … `exp4m5-8V.csv` | Exp 4 — suffix is the retarding voltage |
| `Exp 2 - Franck-Hertz Hg_Selection_*.csv` | Exp 2 amplifier-gain metadata |

All raw data files have two columns: `Voltage (V)` and `Current (a.u.)`.
Rename them to `voltage` and `current` at load time (see existing notebooks).

## Notebook Conventions

### Structure (follow Exp 2 as the canonical template)

1. Colab badge + title markdown cell
2. `## Required Installs and Imports` → `!pip install -q matplotlib iminuit scikit-learn`
3. Imports (numpy, matplotlib, iminuit, sklearn, scipy)
4. **Class definitions** — `Regressor` (ABC), `LeastSquaresReg`, `Chi2Reg` — copy verbatim
   from `Frank_Hertz_Exp2.ipynb`; do **not** import across notebooks (Colab independence)
5. **Utility functions** — `parab_min_representation`, `get_points_between_voltages`,
   `get_chi2_for_voltage_range`, `calculate_dI`, `get_best_fit` — also copied verbatim
6. Experiment-specific analysis cells

### Plots

- Every plot **must** have: `ax.set_title(...)`, `ax.set_xlabel(...)`, `ax.set_ylabel(...)`
- Use `fig.suptitle(...)` for multi-panel figures
- Axis labels must include physical units in parentheses, e.g. `"Accelerating voltage $V_{CG}$ (V)"`
- Follow the style used in Exp 2: `figsize=(14, 6)` for single-panel I-V plots,
  `figsize=(13, 18)` for 5-panel stacks

### Fitting

- Use `Chi2Reg` + `Minuit` for all parameter estimation
- Valley minima: fit `parab_min_representation = lambda v, v0, c: ((v-v0)**2) + c`
- Peak maxima: negate current, then apply the same `get_best_fit` routine
- Linear model for excitation energy: `lambda N, a, b: a * N + b`
- Always add `0.1 V` digitisation floor to `v0_err` before the linear fit:
  `dv0 = (minima_df['v0_err'] + 0.1).to_numpy()`

### Uncertainty

- `dv = 0.1` V (voltage digitisation, applied inside `get_chi2_for_voltage_range`)
- `dI` estimated from a flat "dead zone" below the first excitation threshold:
  - Exp 2 (Hg): `dead_zone_start=2, dead_zone_end=8`
  - Exp 4 (Ne): `dead_zone_start=3, dead_zone_end=12`

## Exp 4 – Specific Notes

- 5 measurements at retarding voltages 2, 4, 6, 8, 10 V (no 0 V measurement in the dataset)
- Valley search brackets (`valley_ranges_ne`) and peak brackets (`peak_ranges_ne`) are defined
  in the notebook — adjust if a specific measurement shifts peaks outside the brackets
- `MULTIPLIER = 1` (no external amplifier gain metadata for neon)
- Selection criterion: maximise valleys found; break ties by lowest mean `|χ²/ndof − 1|`
- The literature value for the dominant Ne Frank-Hertz period is **18.7 eV**
  (excitation to the $2p^5 3p$ state group)

## Working in Colab

All notebooks are designed to run in Google Colab without any local installation.
Upload or mount the data CSV files before running.
The Colab badge in cell 0 links directly to the GitHub-hosted notebook.
