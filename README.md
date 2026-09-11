# Accelerating Low-Carbon Infrastructure Deployment — Evidence from Green Hydrogen

This repository contains the R/Jupyter analysis pipeline accompanying the project **“Accelerating Low-Carbon Infrastructure Deployment — Evidence from Green Hydrogen.”** The analysis combines annual International Energy Agency (IEA) Hydrogen Projects database vintages for 2021–2026 with project characteristics, country-level indicators and geographic data to examine hydrogen project progression, failure and revisions to announced completion schedules.

## Reproducibility workflow

Run **`Hydrogen-Figure-Pipeline.ipynb`** with the R kernel. The master notebook executes eight component notebooks in dependency order, each in a separate R kernel, using a common project/data directory.

| Order | Component notebook | Main role |
|---|---|---|
| 1 | `Data-Preparation.ipynb` | Read and harmonise source inputs; construct the project-year panel, transition datasets and sample-count audit. |
| 2 | `Figure-Tornado.ipynb` | Estimate the primary progression/failure models; generate Figure 2 and Supplementary Figure S2. |
| 3 | `Discrete-Time-Multinomial-Competing-Risk-Model.ipynb` | Estimate the competing-risk models used by Supplementary Figures S3–S5. |
| 4 | `Figure-Demand.ipynb` | Generate progression curves by end-use profile: Figure 3 and Supplementary Figure S3. |
| 5 | `Figure-Capacity.ipynb` | Generate progression curves by capacity quartile: Figure 4 and Supplementary Figure S4. |
| 6 | `Figure-Regions.ipynb` | Generate country-risk progression curves and country composition: Figure 5 and Supplementary Figure S5. |
| 7 | `Figure-Sankey.ipynb` | Generate Figure 1, Supplementary Figures S1/S6, descriptive tables and sample-count caption notes. |
| 8 | `Figure-Delays.ipynb` | Estimate first-delay models; generate Figure 6 and Supplementary Figure S7. |

**Expected runtime: up to 10 minutes**, depending on hardware and the local software environment. Package installation and obtaining source inputs are additional setup steps.

The `.rds` files used downstream are **generated intermediates**. A complete run creates them from the source inputs; they do not need to be supplied separately.

## Repository structure

The workflow supports separate notebook and data directories. A layout consistent with a `first-submission` release is:

| Path | Contents |
|---|---|
| `README.md` | Workflow, input requirements and output documentation. |
| `first-submission/Code/` | The master notebook and eight component notebooks listed above, retaining their exact filenames. |
| `first-submission/Data/` | The annual IEA workbooks, country-indicator workbooks and geographic raster inputs listed below. |
| `first-submission/` | Generated RDS, PDF, CSV, caption-note and execution-log files. |

The notebooks can also be kept directly alongside `Data/`. The master uses `notebook_dir` to locate notebooks and `project_dir` to locate `Data/` and write outputs. The data inventory below describes required files; intermediate RDS files and generated figures should be distinguished from source inputs.

## Data

### Annual hydrogen project data

| File | Contents | Used for |
|---|---|---|
| `H2_IEA_21_cleaned_v3.xlsx` | Cleaned 2021 IEA project vintage. | Initial project histories and identification of left-truncated origin spells. |
| `H2_IEA_22_cleaned_v3.xlsx` | Cleaned 2022 IEA project vintage. | Annual project histories, characteristics and schedules. |
| `H2_IEA_23_cleaned_v3.xlsx` | Cleaned 2023 IEA project vintage. | Annual project histories, characteristics and schedules. |
| `H2_IEA_24_cleaned_v3.xlsx` | Cleaned 2024 IEA project vintage. | Annual project histories, characteristics and schedules. |
| `H2_IEA_25_cleaned_v3.xlsx` | Cleaned 2025 IEA project vintage. | Annual project histories, characteristics and schedules. |
| `H2_IEA_26_cleaned_v3.xlsx` | Cleaned 2026 IEA project vintage. | Final project snapshot and the end of the observation window. |

These workbooks are read from `Data/`, sheet **`Projects`**. The import routine harmonises headers from rows 2–4 and reads project records below row 4. Fields include project reference, status, country, coordinates, electrolyser capacity, technology, electricity source, end uses and expected online year. Workbook names and original Excel row numbers are retained for sample-count investigation.

The pipeline starts from the named **cleaned** workbooks. Their preparation, project-identifier harmonisation and original-source documentation should accompany the data release.

### Country-level indicators

All eight workbooks below are read from `Data/`, sheet **`Output`**, range `B6:D10000`. The routine takes the country identifier from the first selected column and the indicator value from the third selected column.

| File | Contents | Use in the pipeline |
|---|---|---|
| `Country Risk Spread.xlsx` | Country risk spreads. | Main macro control and construction of spread-based rating groups. |
| `Governance Score.xlsx` | Country governance indicator. | Governance control in the current primary and competing-risk specifications. |
| `RISE.xlsx` | Regulatory Indicators for Sustainable Energy scores. | Main macro control. |
| `Ease of doing business.xlsx` | Business-environment indicator. | Prepared upstream; available as an alternative governance/institutional control. |
| `Anhydrous Ammonia Export.xlsx` | Country ammonia-export indicator. | Prepared upstream; outside the current main model controls. |
| `Competitive Industrial Performance.xlsx` | Industrial-performance indicator. | Prepared upstream; outside the current main model controls. |
| `Country Complexity Ranking.xlsx` | Country-complexity indicator. | Prepared upstream; outside the current main model controls. |
| `Freshwater Withdrawal.xlsx` | Freshwater-withdrawal indicator. | Prepared upstream; outside the current main model controls. |

The preparation notebook reads all eight files even when an indicator is not included in the selected regression specification. Country indicators are joined by standardised country codes and are constant across vintages in this implementation. Projects listing multiple countries receive equally weighted averages of the available country values. Source dates, definitions and any proxy-country substitutions belong in the workbook documentation.

### Geographic inputs

| Input | Contents | Used for |
|---|---|---|
| `hydrogen_kde_50km_*.tif` and/or `hydrogen_sum_50km_*.tif` | Precomputed geographic density/sum rasters. | Raster extraction for EU project locations and associated country-average fields. |
| Natural Earth world boundaries | Medium-resolution country geometries accessed through `rnaturalearth::ne_countries()`. | Supplementary Figure S1 project map. |

The raster search is recursive within `Data/`, so matching GeoTIFFs may be placed in subdirectories. At least one raster matching `^hydrogen_(kde|sum)_50km_.*\.tif$` is required by the preparation routine. The notebooks do not generate these raster inputs.

Separately, the preparation notebook computes a vintage-specific, leave-one-out project-density measure using project coordinates and a 50 km bandwidth. This measure is prepared but is not included in the current main core controls. Natural Earth map data must be available to `rnaturalearth`; offline execution requires the relevant packages and geographic data to be available locally.

## Generated analysis datasets

| File | Contents | Main consumers |
|---|---|---|
| `master_data.rds` | Complete reconstructed project-year panel, including harmonised statuses, observed/completed-row flags, project characteristics and country indicators. | Figure-Sankey and Figure-Regions. |
| `cloglog_data_full.rds` | Transition dataset before origin-spell left-truncation exclusion and model-specific preparation. It applies `prep_cloglog()` restrictions on identifiers, countries and valid current/previous states. | Figure-Sankey descriptive transition and duration calculations. |
| `cloglog_data_processed.rds` | Model-ready transition dataset after origin-spell left-truncation exclusion, transformations, standardisation and grouped end-use construction. | Primary progression/failure models, competing-risk models, curve notebooks and delays. |
| `cloglog_standardization_parameters.csv` | Means and standard deviations used for numeric model covariates. | Interpretation and reproducibility of transformations. |
| `waterfall_results.rds` | Fitted progression/failure models, estimation samples and inference information. | Figure-Demand, Figure-Capacity and Figure-Regions. |
| `multinomial_competing_risk_results.rds` | Fitted multinomial models and project-clustered covariance matrices. | Supplementary Figures S3–S5. |
| `delay_waterfall_results.rds` | First-delay risk-set summary and fitted delay-model results. | Delay results and reproducibility. |
| `sample_count_audit.rds` | Detailed sample-count audit and checksums of the prepared RDS inputs. | Figure-Sankey count reconciliation and caption-note generation. |

The main preparation stages are:

1. Read the annual workbooks and audit raw project-vintage identifiers.
2. Harmonise country codes and project status labels.
3. Apply the status-history rules and complete histories through 2026.
4. Identify observed versus reconstructed records, left-truncated origin spells and right-censored final spells.
5. Derive progression, failure, schedule revision and time-in-stage variables.
6. Join country indicators and geographic variables; construct grouped end uses and project characteristics.
7. Save the complete panel and full transition dataset.
8. Exclude transitions originating in left-truncated spells; transform and standardise model covariates.
9. Save the processed dataset, transformation parameters and sample-count audit.

## What produces what

### Main figures

| Figure | PDF output | Notebook | Main content |
|---|---|---|---|
| 1 | `figure_sankey.pdf` | Figure-Sankey | Project development flows, stage-duration distributions and project composition. |
| 2 | `figure_tornado.pdf` | Figure-Tornado | Project- and country-level correlates of progression and failure. |
| 3 | `figure_demand.pdf` | Figure-Demand | Model-based progression curves by recorded end-use profile. |
| 4 | `figure_capacity.pdf` | Figure-Capacity | Model-based progression curves by project capacity quartile. |
| 5 | `figure_country_risk.pdf` | Figure-Regions | Progression curves at rating-group median country-risk values and country composition of rating groups. |
| 6 | `figure_delay.pdf` | Figure-Delays | Pooled first-delay model coefficients. |

`figure_tornado_full.pdf` is the additional full coefficient-chart export associated with Figure 2.

### Supplementary figures

| Figure | PDF output | Notebook | Content |
|---|---|---|---|
| S1 | `figure_S1_project_map.pdf` | Figure-Sankey | Global hydrogen project map. |
| S2 | `figure_S2_variable_distribution.pdf` | Figure-Tornado | Distributions of model variables. |
| S3 | `figure_S3_competing_risk_demand.pdf` | Figure-Demand | End-use progression curves from the multinomial competing-risk model. |
| S4 | `figure_S4_competing_risk_capacity.pdf` | Figure-Capacity | Capacity progression curves from the multinomial competing-risk model. |
| S5 | `figure_S5_competing_risk_regions.pdf` | Figure-Regions | Country-risk progression curves from the multinomial competing-risk model. |
| S6 | `figure_S6_capacity_by_end_use.pdf` | Figure-Sankey | Capacity distributions by grouped end use. |
| S7 | `figure_S7_delays_by_stage.pdf` | Figure-Delays | Separately estimated stage-specific first-delay coefficients. |

### Source data, model summaries and diagnostics

| Output | Contents |
|---|---|
| `figure2_source_data.csv`, `figure2_full_source_data.csv` | Data used in the Figure 2 coefficient-chart exports. |
| `model_summary_table.csv`, `model_diagnostics.csv` | Primary model sample sizes, fit summaries and diagnostics. |
| `human_readable_effect_sizes.csv` | Capacity and country-risk contrasts derived from the progression/failure coefficients. |
| `multinomial_competing_risk_coefficients.csv` | Competing-risk coefficients and project-clustered inference. |
| `multinomial_competing_risk_model_statistics.csv` | Competing-risk sample sizes, outcome counts and convergence summaries. |
| `figure_S3_competing_risk_demand_source_data.csv` | Supplementary demand curves. |
| `figure_S4_competing_risk_capacity_source_data.csv` | Supplementary capacity curves. |
| `figure_S5_competing_risk_regions_source_data.csv` | Supplementary country-risk curves. |
| `delay_coefficients.csv`, `delay_model_statistics.csv`, `delay_model_diagnostics.csv` | Delay-model coefficients, samples, fit statistics and diagnostics. |
| `delay_human_readable_effect_sizes.csv` | Interpretable capacity and country-risk contrasts for delay models. |
| `figure_delay_source_data.csv`, `figure_S7_delays_by_stage_source_data.csv` | Main and supplementary delay-chart data. |
| `delay_plot_objects.rds` | Saved delay plot objects. |

The main Figure 3–5 notebooks also create their curve data in memory. Their current explicit curve CSV exports are for Supplementary Figures S3–S5, as listed above.

### Descriptive table outputs

Figure-Sankey writes the following CSVs. The `extended_data_table_*` filenames are retained in code even when the corresponding manuscript tables are presented as Supplementary Tables.

| File | Contents |
|---|---|
| `extended_data_table_project_sample.csv` | Projects, project-years and transitions overall and by stage, region and end use. |
| `extended_data_table_transitions_by_stage.csv` | Stage-specific progression/failure and full-spell transition counts. |
| `extended_data_table_transition_routes.csv` | Individual origin/destination transition routes, including stage jumps. |
| `extended_data_table_enduse_overlap_summary.csv` | Aggregate overlapping end-use membership. |
| `extended_data_table_enduse_multiplicity.csv` | Counts by number of recorded grouped end uses. |
| `overall_duration_distribution.csv` | Constructed overall duration distribution before display binning. |

Additional missingness checks, project-map summaries, duration quantiles, capacity-quartile counts and coefficient tables are printed in the executed notebooks.

## Sample-count reconciliation and caption notes

The count audit distinguishes **rows**, **valid distinct project-vintage pairs**, **projects** and **transitions**. It does not automatically change or deduplicate the estimation data.

`sample_count_reconciliation.csv` follows four datasets: raw IEA inputs, the reconstructed panel, the full transition dataset and the eligible dataset after origin-spell left-truncation exclusion. The eligible dataset precedes the stage-specific and complete-case restrictions used in model estimation.

For each dataset, the audit reports mutually exclusive exclusions for missing identifiers, empty identifiers and missing/invalid years, followed by additional rows belonging to repeated valid keys. It also reports conflicting duplicate attributes, whitespace-only identifiers, observed records and reconstructed records.

| Output pattern | Contents |
|---|---|
| `sample_count_{raw,panel,transitions,eligible}_summary.csv` | Dataset-specific count reconciliation. |
| `sample_count_{raw,panel,transitions,eligible}_by_year.csv` | Counts by annual vintage. |
| `sample_count_{raw,panel,transitions,eligible}_duplicate_keys.csv` | Repeated valid project-year keys, distinct payload counts and conflicting fields. |
| `sample_count_{raw,panel,transitions,eligible}_flagged_rows.csv` | Flagged records with attributes and workbook/Excel-row provenance. |
| `sample_count_plotting_only_pairs.csv` | Pairs added by annual status plotting beyond the counted panel. |
| `Figure1_ST1_ST2_sample_count_caption_notes.txt` | Run-specific sample-count text for the Figure 1 caption and Supplementary Tables 1–2 notes. |

Braces in the patterns above denote four separate filenames. The Figure-Sankey notebook checks the audit against the current RDS input checksums and verifies that its distinct project-year count reconciles. Caption numbers are generated from the current run rather than hard-coded. Review any flagged duplicate conflicts before incorporating the generated notes into the manuscript.

## Running the analysis

### Requirements

- R with Jupyter's **IRkernel**, registered under the kernel name `ir`. Notebook metadata records R **4.4.3**.
- Jupyter and `nbconvert`, with the `jupyter` executable available on `PATH`.
- System libraries supporting `sf` and `terra`, including GDAL, GEOS and PROJ.
- Cairo PDF support for the figure exports.
- The required source workbooks, raster inputs and Natural Earth map data.

The master preflight checks these R packages:

```r
c(
  "dplyr", "tidyr", "purrr", "tibble", "stringr", "readxl", "cellranger",
  "sf", "terra", "units", "rnaturalearth", "sandwich", "lmtest", "pROC",
  "ggplot2", "ggalluvial", "ggrepel", "ggsci", "scales", "patchwork",
  "MASS", "geepack", "nnet"
)
```

The notebooks also use standard R packages such as `stats`, `utils`, `tools`, `grid` and `grDevices`. Install missing packages before starting the master workflow; the preflight stops and lists missing requirements.

### Execution

For the `first-submission/Code/` layout, one way to start Jupyter is:

```bash
export H2_PROJECT_DIR="/absolute/path/to/repository/first-submission"
cd "$H2_PROJECT_DIR/Code"
jupyter notebook
```

1. Open `Hydrogen-Figure-Pipeline.ipynb` and select the R kernel.
2. Confirm that `notebook_dir` points to the directory containing all nine notebooks and `project_dir` points to the directory containing `Data/`.
3. Keep `run_data_preparation <- TRUE` for a complete rebuild.
4. Keep **`governance_control <- "Governance_Score"`** for the current specification.
5. Run all master-notebook cells from top to bottom.
6. Inspect the execution log, output checks and generated sample-count notes in `project_dir`.

When notebooks and `Data/` share one directory, the default configuration uses that directory for both paths. Alternatively, edit `notebook_dir` and `project_dir` directly in the master's configuration cell.

The master passes `H2_PROJECT_DIR` and `H2_GOVERNANCE_CONTROL` to each component kernel. All generated files are written to the common project directory. Later cells within a component notebook depend on objects created earlier in that notebook.

Setting `run_data_preparation <- FALSE` reuses existing prepared inputs. This requires the updated audit files as well as the panel and transition RDS files. After changing input data, preparation rules or the audit code, perform a complete rebuild.

### Execution logs and output checks

| File | Contents |
|---|---|
| `pipeline_execution_log.csv` | Successfully completed components, governance selection and start/finish times. |
| `main_output_check.csv` | Existence, non-empty status and freshness checks for Figures 1–6. |
| `supplementary_output_check.csv` | Corresponding checks for Supplementary Figures S1–S7. |

The master stops when a component execution fails or an expected output is missing, empty or stale. A successful file-freshness check confirms that an output was regenerated; model diagnostics and numerical results remain available in the component notebooks and diagnostic exports. Rerunning the pipeline overwrites its named generated outputs.

## Important implementation notes

- Primary progression, failure and first-delay models use the complementary log-log link, project-clustered inference and 90% confidence intervals.
- Progression/failure models are estimated separately for Concept, Feasibility study and FID/Construction. Grouped end-use indicators and the alternative any-recorded-end-use indicator are estimated in separate specifications on common stage-specific samples.
- The current macro controls include country risk spread, RISE, region and Governance Score. The master also supports `Ease_of_doing_business` or `none` as alternative governance settings.
- `prev_time_in_status_raw` retains duration in annual intervals; `prev_time_in_status` is its model-scale counterpart. Figure axes use the raw duration and label it “Time in stage.”
- Main progression curves accumulate the fitted progression hazards. Supplementary Figures S3–S5 use a separate multinomial model with mutually exclusive no-transition, progression and failure outcomes and propagate progression cumulative incidence with competing failure.
- Demand scenarios vary recorded end-use indicators across a common estimation sample. Country-risk scenarios assign the rating-group median risk to a common sample. Capacity curves average over observations belonging to each project-level capacity quartile; quartiles use median recorded project capacity.
- Curve averages use project-year records. Other covariates, including each record's calendar-year value, remain fixed while duration is advanced.
- The first-delay workflow identifies the first upward schedule revision within the prepared eligible histories and removes subsequent observations. The main pooled delay model omits stage indicators; S7 uses separately estimated stage models.
- Figure 1 flow totals include left-truncated origin spells and direct stage jumps. The compact flow diagram omits the direct-jump arrows, while the transition-route table retains them. Duration summaries use positive-progression spells with non-left-truncated origins.
- The overall-duration distribution sums the empirical stage-duration distributions under independence. It represents a constructed pathway through the three stages.
- Region/end-use descriptive classifications use the latest project record as of 2026. End-use groups overlap; project-year totals count distinct valid project-vintage pairs in the reconstructed panel.

## Reproducibility scope

The source workbooks and matching rasters must be supplied under `Data/`; the notebooks do not recreate those source files. A full run generates the analysis RDS objects, figures, CSVs and caption notes. Preserve workbook metadata, source dates, cleaning documentation and the R package environment alongside a release. No package lockfile is supplied in the notebook bundle.

## License and data attribution

Consult the repository-level `LICENSE` file, where provided, for code reuse terms. The notebook bundle does not itself declare a code licence.

IEA project data, country indicators, raster inputs and Natural Earth geometries retain their respective source attribution and usage terms. Consult the original providers and accompanying workbook metadata, definitions, references and proxy mappings before redistributing or adapting these inputs.
