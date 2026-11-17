# Data and code: salamander occupancy response to wildfire and timber harvest

Data and R code for:

> Williamson JM, Twining JP, Homyack JA, Garcia TS. Rare and common salamanders show resilience to increasing wildfires but are sensitive to forest management. *Ecological Applications* (in review).

We fit hierarchical multi-scale occupancy models (site occupancy ψ, plot use θ, detection p) in a Bayesian framework (NIMBLE) to detection/non-detection data for Oregon Slender salamander (*Batrachoseps wrighti*, "OSS") and Ensatina (*Ensatina eschscholtzii*, "ENES"). The surveys were done in the western Oregon Cascades in 2013–2019 (before the 2020 Labor Day fires) and 2023–2024 (after them).

Contact: Jasmine Williamson, jasmine.williamson@oregonstate.edu

## How to run

1. Open `multi-scale-occu-oss.Rproj` in RStudio, or set the working directory to this folder. All paths are relative to this folder.
2. Run the scripts in `code/` in numerical order. Steps 1 and 2 can be skipped: their outputs are included in `data/`.
3. Figures are written to `figures/` and tables to `output/`. These folders start empty.

| Script | What it does | Inputs | Outputs |
|---|---|---|---|
| `1-model-data-multiscale-prepost-merge.R` | Merges pre- and post-fire detection and covariate data into one subplot × survey matrix per species | `dets.*.csv`, `site_level_matrix.csv`, `subplot.complete.new.csv`, `geo-data-all-plots-prefire.csv` | `data/enes.prepost.multiscale.occu.csv`, `data/oss.prepost.multiscale.occu.csv` |
| `2-model-code-multi-scale-occu-4D-V3-current.R` | Formats the data into arrays and fits the multi-scale occupancy model in NIMBLE (3 chains). Set `species <- "enes"` or `"oss"` at the top and run once per species. **This is slow (2.5 million iterations per chain).** | `*.prepost.multiscale.occu.csv` | `data/msc-data-workspace-V3.RData`, `data/multiscale_output_121225_<species>_small.RData`, `data/multiscale_output_and_data_121225_<species>_full.RData` |
| `3-marginal-coeff-plots.R` | Coefficient plot and marginal-effect plots for covariates on ψ, θ, and p (Figure 5) | model outputs | `figures/o-coeff-plot.png`, `figures/marginal-effects-combined-2.png` |
| `4-both-occu-preds-treatment-plot.R` | Predicted occupancy by treatment for both species, all years combined (Figure 6) | full model outputs | `output/posterior-preds-both-V3.csv`, `figures/both-spp-trt-psi-preds.png` |
| `5-posterior-distribution-plots.R` | Posterior distributions of occupancy covariate effects (Figure 4) | full model outputs | `figures/both-posterior-dist-facet.png` |
| `6-msc-occu-yearly-plot.R` | Predicted occupancy by year and treatment (Figure 3) | small model outputs | `figures/facet-yearly-preds-v3.png` |
| `7-posterior-probs-for-pub.R` | Predicted occupancy by treatment, parameter estimates with 95% credible intervals, and P(β > 0) (Table 2; Appendix S1: Tables S2–S3) | full model outputs | `output/beta_coefficients_table.docx` |

`code/attach-nimble.R` is a helper function used by script 7. It came from J. Stewart's Bayesian statistics course materials.

### Model output files

- `*_full.RData`: object `a`, an `mcmc.list` of 3 chains with 45,001 saved samples each (thinned by 50, burn-in removed). Also holds the NIMBLE model code, constants, data, and initial values.
- `*_small.RData`: object `a2`, a heavily thinned version (451 samples per chain) used for diagnostics and the yearly plots.
- `msc-data-workspace-V3.RData`: R workspace saved by script 2 after data formatting. It holds the treatment indicator arrays used by the prediction scripts.

## Data files (`data/`)

Treatment codes used throughout:

| Code | Treatment |
|---|---|
| UU | control (unharvested, unburned) |
| HU | harvest-only |
| BU | burn-only |
| HB | harvest then burn |
| BS | burn then salvage harvest |

Species codes: OSS = *B. wrighti*, ENES = *E. eschscholtzii*. Other amphibian columns in `subplot.complete.new.csv` are PLDU = *Plethodon dunni*, TAGR = *Taricha granulosa*, AMGR = *Ambystoma gracile*, ANFE = *Aneides ferreus*.

| File | Rows | Description |
|---|---|---|
| `dets.o.pre.csv`, `dets.e.pre.csv` | 3,379 | Pre-fire (2013–2019) detection data for OSS and ENES (from Garcia et al. 2020). One row per stand × year × subplot. `V1`–`V3` are detection (1) / non-detection (0) on repeat surveys 1–3. Also has downed wood (`DW`), Julian date, air temperature, owner, and treatment group. |
| `dets.o.post.csv`, `dets.e.post.csv` | 889 | Post-fire (2023–2024) detection data. One row per site × year × subplot; `V1`–`V3` as above. |
| `site_level_matrix.csv` | 126 | Post-fire site-level covariates: landowner, treatment (`trt`), survey date, coordinates, weather, temperature, humidity, cover, soil moisture, downed wood, char class, and site-level counts (`oss`, `enes`). |
| `subplot.complete.new.csv` | 889 | Post-fire subplot-level covariates (cover, soil moisture, downed wood counts and size/decay/char/length classes, temperature, humidity), observer initials (`obs`), and counts per species. |
| `geo-data-all-plots-prefire.csv` | 3,379 | Pre-fire subplot coordinates (`lat`, `long`) and elevation (`elev`). |
| `oss.prepost.multiscale.occu.csv`, `enes.prepost.multiscale.occu.csv` | 4,268 | Merged model input (output of script 1). `V1`–`V3` detections; treatment indicators `HB`, `BU`, `BS`, `HU`, `UU`; `temp` (°C), `lat`, `long`, `elev`, `DW` (downed wood count), `jul_date`, `owner`, `mgmt_type` (0 = private, 1 = public), `char_cl`. Covariates are identical in the two files; only the detection columns differ. |

Units: temperature in °C in the model inputs (converted from °F in script 1); elevation in feet; coordinates in decimal degrees. Continuous covariates are standardized (mean 0, SD 1) inside script 2 before model fitting.

## Software

R 4.4.2 with: nimble 1.3.0, coda 0.19-4.1, runjags 2.2.2-5, dplyr 1.1.4, tidyr 1.3.1, tibble 3.2.1, data.table 1.16.4, ggplot2 4.0.0, patchwork 1.3.1, ggh4x 0.3.1, ggtext 0.1.2, cowplot 1.1.3, flextable 0.9.10.
