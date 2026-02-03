### Overview

This repository provides a **reproducible analysis pipeline** for gene-level risk inference using three Bayesian frameworks:

- **TADA-family** 
- **TADA-CC** 
- **TADA-RD** 

The pipeline integrates family-based PTV counts, simulated or user-provided case–control variant data and its probabilistic inference of mutation origin (de novo or inherited). It is designed to demonstrate how **TADA-RD** improves discovery power while remaining compatible with standard TADA analyses.

------

## Repository Structure

```
.
├── README.md
├── run_TADA_RD.R                  # Main analysis script
├── functions.R                    # Bayes factor and helper functions
├── data/
│   ├── genetable_info.xlsx        # Gene-level annotations and priors
│   ├── fu_suppl_table5.xlsx       # ASC family-based PTV counts (Fu et al., 2022)
│   ├── PROBANDS-simulated-ptv-variants.txt
│   ├── SIBLINGS-simulated-ptv-variants.txt
│   ├── ClassDn_rusboost.Rdata # Trained de novo inference model
│   └── accuracy_rusboost      # Sensitivity/specificity estimates
├── output/
│   └── (analysis results written here)
```

------

## Key Scripts

- **`run_TADA_RD.R`**
  Executes the full analysis pipeline:
  - merges family-based and case–control PTV data,
  - infers de novo vs. inherited mutation status in cases,
  - computes Bayes factors for **TADA-family**, **TADA-CC**, and **TADA-RD**,
  - estimates Bayesian FDR and outputs gene-level results.
- **`functions.R`**
  Contains Bayes factor implementations and helper functions, including DN, CC, and RD likelihood components.

------

## Analysis Workflow

1. **Load gene annotations and family-based PTV counts**
   Published ASC de novo and inherited PTV counts (Fu et al., 2022) are merged with gene-level covariates and mutation-rate priors.
2. **Aggregate simulated or user-provided case–control variants**
   Variant-level PTV data are collapsed into gene-level counts for cases and controls.
3. **Infer de novo vs. inherited variants in cases**
   A trained probabilistic classifier assigns a posterior probability of de novo origin to each variant.
   Variants exceeding a fixed probability threshold (default: 0.7) are classified as de novo.
4. **Construct gene-level mutation counts**
   For each gene:
   - `case`: total PTVs in cases
   - `ldn`: inferred de novo PTVs
   - `lin`: inferred inherited PTVs
5. **Compute Bayes factors**
   - `BF_TADA_family`: family-based TADA using ASC data
   - `BF_TADA_CC`: classical TADA-CC using case–control data
   - `BF_TADA_RD`: TADA-RD combining family-based evidence with inferred mutation origin in cases
6. **Bayesian FDR estimation**
   Bayesian FDR is computed for each method under a fixed non-risk prior, yielding:
   - `qval_TADA_family`
   - `qval_TADA_CC`
   - `qval_TADA_RD`

------

## Parameters You Must Adjust (Sample Sizes)

`run_TADA_RD.R` requires users to specify **sample sizes**, because these directly affect Bayes factor computations.

In the current script these appear under:

```r
### Setting sample sizes
n_asc_trio_case    = 6430
n_asc_trio_control = 2179
n_case    = 8000
n_control = 2500
```

### What these mean

- `n_asc_trio_case`, `n_asc_trio_control`: number of probands and siblings **in the family-based (ASC trio) dataset** used for `fu_suppl_table5.xlsx`. These parameters are used in the **family-based TADA** and therefore affect :`BF_TADA_family`, `BF_TADA_CC` and `BF_TADA_RD`
- `n_case`, `n_control`: number of individuals **in your case–control dataset** corresponding to the variant-level files you provide (or the simulator output). These parameters are used **only in the TADA-CC component**.

### How to set them for your data

- If you replace the ASC family table with a different trio dataset, update `n_asc_trio_case` and `n_asc_trio_control` accordingly.
- If you run the pipeline on your own case–control study, set `n_case` and `n_control` to your cohort sizes.

**Important:** Do *not* infer `n_case`/`n_control` from the number of variants in the input files—these parameters refer to the **number of individuals**, not variant rows.

------

## Input Data Requirements

To apply the pipeline to new data, users should provide:

1. **Gene-level annotation table** (`genetable_info.xlsx`)
   Gene ID, mutation rates, prior parameters, and optional covariates.
2. **Variant-level case–control PTV data**
   Two tab-delimited files with at minimum a gene identifier column:
   - Cases: `PROBANDS-...txt`
   - Controls: `SIBLINGS-...txt`
3. **De novo inference model**
   - trained classifier (`ClassDn_rusboost.Rdata`)
   - sensitivity/specificity calibration (`accuracy_rusboost`)

------

## Quick Tutorial

### Step 1: Prepare input files

Place the following in `data/`:

```text
genetable_info.xlsx
PROBANDS-ptv-variants.txt
SIBLINGS-ptv-variants.txt
ClassDn_rusboost.Rdata
accuracy_rusboost
```

### Step 2: Update sample-size parameters

Open `run_TADA_RD.R` and set:

- `n_case`, `n_control` to match your case–control cohort sizes
- (optional) `n_asc_trio_case`, `n_asc_trio_control` if you change the family-based dataset

### Step 3: Run the pipeline

```r
setwd("path/to/repository")
source("run_TADA_RD.R")
```

### Step 4: Inspect results

The output results table includes:

- Bayes factors and q-values from TADA-family, TADA-CC, and TADA-RD,
- inferred de novo/inherited counts (`ldn`, `lin`),
- a comparison of genes discovered by TADA-RD vs. TADA-CC.

------

## Minimal Required Columns for Variant-Level Input Files

The variant-level input files for **cases** and **controls** must be **tab-delimited text files**.
Only a small subset of columns is required by `run_TADA_RD.R`.

### Required for **case (proband) variants**

The case file (e.g. `PROBANDS-ptv-variants.txt`) must contain the following columns:

| Column name  | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| `gene`       | Gene identifier (must match `Gene_ID` in `genetable_info.xlsx`) |
| `ccr_pct`    | CCR percentile (used as a variant-level covariate; missing values allowed) |
| `gnomAD.MAF` | Population allele frequency (used by the de novo classifier) |

**Notes**

- Rows correspond to individual PTVs.
- Missing values in `ccr_pct` are automatically set to 0.
- Additional columns may be present but are ignored by the pipeline.

------

### Required for **control (sibling) variants**

The control file (e.g. `SIBLINGS-ptv-variants.txt`) must contain:

| Column name | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| `gene`      | Gene identifier (must match `Gene_ID` in `genetable_info.xlsx`) |

**Notes**

- Only gene-level aggregation (`n()` per gene) is used for controls.
- Variant-level covariates are not required for controls.
- Sibling (control) data are used only for the TADA-CC component, and therefore inferring de novo mutation status is not required 

------

### Gene identifier consistency

- The `gene` column in both files must match the `Gene_ID` column in `genetable_info.xlsx`.
- Genes not present in `genetable_info.xlsx` are automatically excluded.

## Notes on Reproducibility

- Thresholds (e.g., de novo posterior cutoff `c = 0.7`) and priors are explicit in code.
- The analysis is modular and can be adapted to alternative cohorts by replacing the input files and updating sample sizes.
- RUSboost can be replaced by Underbagging model. 

