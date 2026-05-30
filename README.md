# Model Fixation Analysis

Version: 1.0.0  
Date: 2026-05-26

## Overview

This repository contains the notebook used to simulate fixation probabilities for a beneficial allele lineage under three offspring distributions:

- Poisson
- Negative binomial
- Fractional

The main workflow is in `Model.ipynb`. It runs the simulations, writes the CSV results table, and displays the figures. It does not save PNG or PDF figure files.

## Files

`Model.ipynb` is the primary analysis notebook. Run its cells in order:

1. Helper functions and setup.
2. Model inputs and simulation run.
3. CSV output and figures.

`model_results_summary.csv` is the default CSV output from the notebook.

Other notebooks and data files in this folder are older or adjacent working files and are not required to run `Model.ipynb`.

## Environment

The current working environment used for this notebook was:

- Operating system: macOS 26.2 arm64
- Python: 3.12.0
- NumPy: 2.4.2
- pandas: 3.0.0
- Matplotlib: 3.10.8
- Jupyter Notebook, JupyterLab, VS Code, or Positron

If Matplotlib warns that its cache directory is not writable, set `MPLCONFIGDIR` to a writable folder before running the notebook.

## Running

Open `Model.ipynb` and run all cells from top to bottom.

The default run uses:

- `s = 0.02`
- `initial = 1`
- `threshold = 250`
- `n_census = 200000`
- `reps = 100`
- `sims = 10000`
- `frac_zeros = (0.4, 0.8)`
- `r_nb = 2`
- `max_gen = 100000`
- `seed = 12345`

Each simulation stops when the allele is lost or reaches the threshold copy number. `max_gen` is only a fail-safe for unresolved runs.

## Configuration

Set these environment variables before launching the notebook to override defaults:

- `MODEL_S`: selection coefficient.
- `MODEL_THRESHOLD`: threshold copies counted as fixation.
- `MODEL_N_CENSUS`: census population size used for effective-size calculations.
- `MODEL_INITIAL`: initial allele copies.
- `MODEL_MAX_GEN`: fail-safe generation limit.
- `MODEL_SIMS`: simulations per replicate batch.
- `MODEL_REPS`: replicate batches.
- `MODEL_FRAC_ZEROS`: comma-separated fractional zero probabilities, for example `0.4,0.8`.
- `MODEL_R_NB`: negative-binomial `r` value for Figure 1.
- `MODEL_SEED`: random seed.
- `MODEL_RESULTS_CSV`: CSV output path.
- `MODEL_NB_SWEEP_REPS`: replicate batches for the negative-binomial sweep.
- `MODEL_NB_SWEEP_SIMS`: simulations per sweep batch.
- `MODEL_NB_SWEEP_GEN`: generation limit for the negative-binomial sweep.
- `MODEL_LOG_LEVEL`: Python logging level.

## Model Notes

`mu` is the selected mean offspring count:

`mu = 1 + s`

The simulation follows only copies of allele `A`. A replicate means one copy of `A` in the current generation.

The neutral family-size variance is calculated before adding selection, so variance and `Ne` use the neutral mean of `1`.

Distribution parameterizations:

- Poisson: offspring are drawn from `Poisson(mu)`.
- Negative binomial: offspring have mean `mu` and dispersion parameter `r`. Neutral per-copy variance is `1 + 1/r`; neutral family-size variance is `2 + 2/r`.
- Fractional: each copy has probability `f` of producing zero offspring. The fractional mean is `mu_frac = mu / (1 - f)`, and active copies draw offspring from `Poisson(mu_frac)`.

For the fractional draw:

`E[X] = (1 - f) * mu_frac`

`Var(X) = (1 - f) * mu_frac * (1 + f * mu_frac)`

Neutral fractional family-size variance:

`V = 2 / (1 - f)`

Effective population size:

`Ne = (4 * N_census) / (V + 2)`

Figure 1 uses branching-process theoretical fixation probabilities for the plotted offspring distributions. Figure 2 is negative-binomial only. It computes `Ne` from `Ne = (4 * N_census) / (V + 2)` and uses:

`pfix = 1 - exp(-2 * s * (Ne / N_census))`

## Output Columns

`selection`: selection coefficient.

`distribution`: offspring distribution.

`threshold_copies`: threshold copy number counted as fixation.

`frac_zero`: fractional zero-offspring probability, blank for non-fractional rows.

`r`: negative-binomial dispersion parameter, blank for non-negative-binomial rows.

`n`: replicate batches used to summarize fixation probability.

`mean_pfix`: mean fixation probability across replicate batches.

`theoretical_pfix`: theoretical fixation probability for the row.

`sd_pfix`: standard deviation of fixation probability across replicate batches.

`first_gen_loss_freq`: first-generation loss frequency for fractional rows.

`variance`: analytical neutral family-size variance.

`N_census`: census population size used in effective-size calculations.

`Ne`: effective population size inferred from neutral family-size variance.
