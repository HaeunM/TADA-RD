This repository contains code for simulating a **realistic case–control dataset** used to illustrate the **RD model pipeline**.

### Overview

The simulation generates **de novo** and **inherited protein-truncating variants (PTVs)** for **probands (cases)** and **siblings (controls)** under a realistic gene- and variant-level model.

A detailed description of the simulation procedure is provided in:

> *“Realistic generation of proband–sibling data.docx”*

### Required Input Files

#### Core inputs (required for proband/sibling simulations)

- `gene-hg38-df-with-ccr.txt`
  Gene-level annotations
- `ptv-hg38-df-with-ccr.txt`
  PTV-level variant annotations

#### Additional inputs for proband simulation

- `denovo-signal-distribution.pdf`
- `inherited-signal-distribution.pdf`

### Simulation Modes

The simulation is controlled via the `SUBSET` option.

#### `SUBSET=PROBANDS`

Generates a synthetic **proband (case)** dataset and outputs:

1. `Probands-genes-with-signal.txt`
   Gene-level information for signal genes
2. `Probands-simulated-ptv-variants.txt`
   Variant-level PTV data
3. `Probands-simulated-ptv-variants-by-gene.txt`
   Gene-level aggregated variant counts

#### `SUBSET=SIBLINGS`

Generates a synthetic **sibling (control)** dataset and outputs:

1. `Siblings-simulated-ptv-variants.txt`
   Variant-level PTV data
2. `Siblings-simulated-ptv-variants-by-gene.txt`
   Gene-level aggregated variant counts

### Output Location

All simulation outputs are saved to the `output/` directory.

### Data Used for RD Analysis

For demonstration of the RD procedure, **only the variant-level files** are used:

- `Probands-simulated-ptv-variants.txt`
- `Siblings-simulated-ptv-variants.txt`

These files are copied into the `data/` directory for downstream RD analysis.

