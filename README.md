# Model Fixation Analysis

Version: 0.2.0  
Date: 2026-05-23  

## Overview

This project estimates fixation probabilities for a beneficial allele lineage under three offspring distributions:

- Poisson
- Negative binomial
- Fractional reproduction, implemented as zero-inflated Poisson reproduction where each copy fails to reproduce with probability `f`; reproducing copies draw offspring from a Poisson distribution with adjusted mean `mu / (1 - f)`.

The simulation starts with one copy of allele `A`. Each `A` copy has fitness `1 + s` and produces replicate copies according to the selected offspring distribution. Runs continue until the allele is lost (`0` copies) or reaches the copy-number threshold. The resident allele is not simulated explicitly, so selection enters through the expected reproductive output of the `A` lineage.

Throughout this project, a replicate means one copy of allele `A` in the current generation. Family-size variance and effective population size are calculated from the neutral offspring distribution before introducing the beneficial allele, so those quantities use `s = 0` even when the fixation simulation uses `s > 0`.

The main analysis is written as a Jupyter notebook so the code, results table, and figure descriptions stay together.

## Files

`testm2.ipynb` is the primary analysis notebook. Run its cells in order:

1. Helper functions and setup.
2. Model inputs and simulation run.
3. CSV output and figures.

`model_results_summary.csv` is the default CSV output written by the final notebook cell. The notebook does not save PNG, PDF, or other image files.

Other notebooks and data files in this folder are older or adjacent working files and are not required to run `testm2.ipynb`.

## Environment

The current working environment used for this notebook was:

- Operating system: macOS 26.2 arm64
- Python: 3.12.0
- NumPy: 2.4.2
- pandas: 3.0.0
- Matplotlib: 3.10.8
- Jupyter Notebook or another notebook runner such as Positron, VS Code, or JupyterLab

If Matplotlib warns that its default cache directory is not writable, set `MPLCONFIGDIR` to a writable folder before running the notebook.

## Running the Analysis

Open `testm2.ipynb` in a notebook environment and run all cells from top to bottom.

The default run uses:

- Selection coefficient: `s = 0.02`
- Initial copies: `1`
- Fixation threshold: `250` copies
- Census population size: `N = 200,000`
- Replicate batches: `100`
- Simulations per batch: `10,000`
- Fractional zero probabilities: `0.4, 0.8`
- Negative-binomial exemplar: `r = 2`
- Fail-safe maximum generations: `100,000`
- Random seed: `12345`

Each simulation stops when allele `A` is lost or reaches the fixation threshold. `MODEL_MAX_GEN` is a fail-safe for unresolved runs, not the intended fixed stopping time.

The final cell writes `model_results_summary.csv`, displays Figure 1, and recomputes the negative-binomial sweep for Figure 2. Figure 2 is intentionally negative-binomial only.

## Configuration

The notebook can be configured with environment variables before launching the notebook process:

- `MODEL_S`: selection coefficient
- `MODEL_THRESHOLD`: threshold copies for fixation
- `MODEL_N_CENSUS`: census population size
- `MODEL_INITIAL`: initial allele copies
- `MODEL_MAX_GEN`: maximum generations before an unresolved run fails
- `MODEL_SIMS`: simulations per replicate batch
- `MODEL_REPS`: replicate batches
- `MODEL_FRAC_ZEROS`: comma-separated fractional zero probabilities, for example `0.4,0.8`
- `MODEL_R_NB`: negative-binomial `r` value for Figure 1
- `MODEL_SEED`: random seed
- `MODEL_RESULTS_CSV`: CSV output path
- `MODEL_NB_SWEEP_REPS`: replicate batches for the negative-binomial sweep
- `MODEL_NB_SWEEP_SIMS`: simulations per sweep batch
- `MODEL_NB_SWEEP_GEN`: maximum generations for the sweep
- `MODEL_LOG_LEVEL`: Python logging level

## Methods Notes

For the model comparison, offspring counts are simulated directly until each run reaches extinction or the copy-number threshold. The fractional model records the first-generation loss frequency for fractional rows.

The sampling functions use these parameterizations:

- Poisson: each copy samples offspring from a Poisson distribution with mean `mu = 1 + s`.
- Negative binomial: each copy samples offspring from a negative binomial distribution with mean `mu = 1 + s` and dispersion controlled by `r`. Under the neutral distribution used for variance calculations, per-copy variance is `1 + 2/r`; family-size variance is twice that value, `2 + 4/r`.
- Fractional: each copy has probability `f` of producing zero offspring. Active copies draw from a Poisson distribution with adjusted mean `mu / (1 - f)`, preserving the overall mean `mu = 1 + s`.

Effective population size is computed from family-size variance as:

`Ne = ((4N) - 4) / (Vk + 2)`

The plotted effective size ratio is:

`Ne / N`

The negative-binomial sweep uses:

`Vk = 2 + 4 / r`

and compares simulated fixation probabilities to the effective-population-size approximation. The sweep varies `r` from `0.4` to `100`, corresponding to neutral per-copy variance from `6.0` to `1.02` and neutral family-size variance from `12.0` to `2.04` under this parameterization.

## Output Columns

`selection` is the selection coefficient used in the fixation simulation.

`distribution` is the offspring distribution used for the row: Poisson, negative binomial, or fractional.

`threshold_copies` is the lower copy-number bound used to count threshold fixation.

`frac_zero` is the fraction of copies that produce zero offspring in the fractional distribution. This does not include zeros that occur from the subsequent Poisson draw among active copies.

`r` is the negative-binomial dispersion parameter.

`n` is the number of replicate batches used to summarize fixation probability.

`mean_pfix` is the mean fixation probability across replicate batches.

`theoretical_pfix` is the theoretical fixation probability used for comparison. Figure 1 uses a branching-process extinction fixed-point calculation for the specified offspring distribution. Figure 2 uses the effective-population-size approximation for the negative-binomial sweep.

`sd_pfix` is the standard deviation of fixation probability across replicate batches.

`first_gen_loss_freq` is the first-generation loss frequency for fractional rows, averaged across replicate batches.

`variance` is the analytical neutral family-size variance for the offspring distribution. This is the theoretical variance described in the methods notes, retained as `variance` in the CSV for compatibility with the analysis notebook.

`N_census` is the census population size used in the effective-size calculation.

`Ne` is the effective population size inferred from neutral family-size variance.
