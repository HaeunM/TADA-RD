## Data Description

This directory contains data files used to implement and illustrate the **TADA-RD** method.

### Data required to run the TADA-RD method

- **`genetable_info`**
  Gene-level annotations and prior parameters required for TADA-family, TADA-CC, and TADA-RD analyses.
- **`ClassDn_rusboost`**
  A de novo–inference classifier trained on the SPARK family-based dataset using the RUSBoost algorithm.
- **`ClassDn_underbagging`**
  A de novo–inference classifier trained on the SPARK family-based dataset using the UnderBagging algorithm.
- **`accuracy_rusboost`**
  Sensitivity and specificity estimates for `ClassDn_rusboost` across probability thresholds.
- **`accuracy_underbagging`**
  Sensitivity and specificity estimates for `ClassDn_underbagging` across probability thresholds.

*Note:* De novo inference is required **only for proband (case) data** in TADA-RD.
Sibling (control) data are used **only for the TADA-CC component** and do not require de novo inference.

------

### Example datasets provided

- **`fu_supplementary_table`**
  An example family-based dataset from Supplementary Table 5 of Fu et al. (2022), containing published de novo and inherited PTV counts.
- **`PROBANDS-simulated-ptv-variants`**
  Simulated proband (case) PTV variants generated using the procedure implemented in the `realistic_data_generation/` folder.
- **`SIBLINGS-simulated-ptv-variants`**
  Simulated sibling (control) PTV variants generated using the procedure implemented in the `realistic_data_generation/` folder.

------

