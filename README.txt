# Leakage-controlled, mechanism-aware Bayesian condition assessment of RC frames using finite-element training data and experimental demonstration

This repository provides the reproducible code package for the manuscript:

**Leakage-controlled, mechanism-aware Bayesian condition assessment of RC frames using finite-element training data and experimental demonstration**

The package reproduces the finite-element-derived dataset, categorical evidence tables, Bayesian-network and machine-learning evaluations, leakage-control analysis, hybrid-model results, external experimental demonstration, and manuscript figures/tables.

The generated finite element dataset is included under `data/`, so the main analysis can be reproduced without rerunning OpenSeesPy. OpenSeesPy is required only to regenerate the 528-case FE dataset from scratch or to rerun the two finite element validation models.

---

## 1. Repository contents

The repository is organized as a numbered analysis pipeline.

```text
config.py                              Shared paths, constants, target names, and plotting settings

01_generate_fe_dataset.py              Generates the 528-case nonlinear FE dataset
02_labels_and_dataset_figures.py       Defines targets, evidence states, and dataset figures
03_evaluate.py                         Runs Bayesian-network and baseline-model evaluations
04_hybrid.py                           Runs the hybrid Bayesian-posterior + gradient-boosting model
05a_fe_existing_frame.py               OpenSeesPy validation model for the existing deficient frame
05b_fe_dj1_stilted.py                  OpenSeesPy validation model for the DJ-1 stilted frame
06_measured_features.py                Extracts scalar features from measured cyclic records
07_external_validation.py              Performs external experimental condition assessment

run_all.py                             Runs the full reproducible workflow
run_kfold_only.py                      Optional resumable repeated k-fold helper
run_rc_only.py                         Optional helper for rerunning residual-capacity evaluation
requirements.txt                       Python dependency list
```

---

## 2. Software requirements

The code was tested with **Python 3.11-3.12**.

Create and activate a virtual environment:

```bash
python -m venv venv
```

On Windows:

```bash
venv\Scripts\activate
```

On macOS or Linux:

```bash
source venv/bin/activate
```

Install all dependencies:

```bash
pip install -r requirements.txt
```

For analysis only, without regenerating the OpenSeesPy finite element models, the essential packages are:

```bash
pip install numpy pandas scikit-learn matplotlib pgmpy
```

OpenSeesPy is required only for:

1. regenerating the FE dataset using `01_generate_fe_dataset.py` or `run_all.py --full`; and
2. running the FE validation scripts `05a_fe_existing_frame.py` and `05b_fe_dj1_stilted.py`.

---

## 3. Quick start

Because the generated FE dataset is included in `data/`, the main results can be reproduced directly:

```bash
python run_all.py
```

This command writes the main outputs to:

```text
results/
figures/
```

Typical result files include:

```text
results/cv_results.csv
results/leakage_scores.csv
results/table1_model_comparison.csv
results/table2_external_validation.csv
results/hybrid_results.csv
```

To regenerate the 528-case FE dataset from scratch, run:

```bash
python run_all.py --full
```

This option is slower and requires OpenSeesPy.

---

## 4. Pipeline stages

### Stage 01: Finite element dataset generation

```bash
python 01_generate_fe_dataset.py
```

Generates the 528-case nonlinear pushover dataset using fiber beam-column models with Concrete02 and Steel02 material models. The dataset spans full-scale regular frames, laboratory-scale regular frames, and vertically irregular stilted frames.

Output:

```text
data/fem_dataset_528.csv
```

This stage requires OpenSeesPy and may take substantially longer than the analysis-only stages.

### Stage 02: Target labels and categorical evidence

```bash
python 02_labels_and_dataset_figures.py
```

Defines the three condition targets and categorical evidence variables.

Target definitions:

| Target | Defining response quantity | Classes |
|---|---|---|
| Damage state | Peak inter-story drift ratio | none, IO, LS, CP |
| Stiffness loss | Fractional secant-stiffness loss | low, medium, high |
| Residual capacity | Post-peak strength retention | failed, retained |

Thresholds used in the manuscript:

- Damage state: 0.5%, 1.0%, and 2.0% drift thresholds.
- Stiffness loss: 60% and 80% fractional stiffness-loss thresholds.
- Residual capacity: binary classification at 0.8 Vmax.

Outputs:

```text
data/fem_dataset_categorical.csv
figures/
```

### Stage 03: Bayesian network, baselines, and leakage control

```bash
python 03_evaluate.py
```

Evaluates the expert-specified Bayesian network, logistic regression, random forest, gradient boosting, and majority-class baseline.

The stage evaluates three evidence regimes:

1. full evidence;
2. leakage-controlled evidence; and
3. design-only evidence.

The leakage-controlled regime removes the same-scalar evidence node that directly re-bins the response quantity used to define each target.

Outputs:

```text
results/cv_results.csv
results/leakage_scores.csv
results/table1_model_comparison.csv
figures/
```

### Stage 04: Hybrid model

```bash
python 04_hybrid.py
```

Runs the hybrid model in which Bayesian posterior probabilities are used as additional structured features for gradient boosting. Posterior probabilities used as gradient-boosting inputs are generated within the training folds to avoid using posterior estimates fitted on the same test cases.

Outputs:

```text
results/hybrid_results.csv
figures/
```

### Stage 05: Finite element validation models

Two OpenSeesPy validation models are provided.

```bash
python 05a_fe_existing_frame.py
python 05b_fe_dj1_stilted.py
```

`05a_fe_existing_frame.py` models the existing deficient RC frame from Wahab et al. (2023) using a FEMA-461 cyclic loading protocol and a robust recursive-bisection solver.

`05b_fe_dj1_stilted.py` models the DJ-1 stilted frame with a story-stiffness ratio of 40, based on the Li et al. (2023) experimental program. The backbone response is anchored to the digitized experimental envelope and reported tabulated values.

### Stage 06: Measured experimental features

```bash
python 06_measured_features.py
```

Extracts scalar condition features from four measured cyclic records:

```text
data/measured_existing_frame.csv
data/measured_prestress_frame.csv
data/measured_DJ2_hysteresis.csv
data/DJ1_backbone_from_figure.csv
```

Output:

```text
data/measured_frame_features.csv
```

### Stage 07: External experimental demonstration

```bash
python 07_external_validation.py
```

Infers the condition of four physically tested RC frames and reports the probability of significant damage and the probability of substantial stiffness loss.

Outputs:

```text
results/table2_external_validation.csv
figures/
```

---

## 5. Data files

The principal data files are:

```text
data/fem_dataset_528.csv              Generated 528-case finite element dataset
data/fem_dataset_categorical.csv      Categorical evidence table used by the models
data/measured_existing_frame.csv      Existing deficient frame measured cyclic record
data/measured_prestress_frame.csv     Retrofitted frame measured cyclic record
data/measured_DJ2_hysteresis.csv      DJ-2 measured cyclic record
data/DJ1_backbone_from_figure.csv     DJ-1 digitized backbone and tabulated anchors
data/measured_frame_features.csv      Extracted scalar features for physical specimens
```

The measured experimental data are included only in processed form for reproducibility and should be cited to the corresponding original experimental sources.

---

## 6. Resumable cross-validation helpers

The repeated cross-validation stage is the slowest analysis step. If it does not finish in one run, use:

```bash
python run_kfold_only.py
```

This creates:

```text
results/_kfold_partial.csv
```

To rerun only the residual-capacity evaluation:

```bash
python run_rc_only.py
```

After generating the partial checkpoint, rerun:

```bash
python 03_evaluate.py
```

This assembles the final cross-validation results, leakage scores, model-comparison table, and related figures.

---

## 7. Implementation notes

Finite element analyses were performed using OpenSeesPy. Data processing, Bayesian-network inference, machine-learning classification, validation, and plotting were implemented in Python.

Important implementation details:

- Bayesian-network learning and inference use `pgmpy`.
- Conditional probability tables are estimated using BDeu parameter learning.
- The BDeu equivalent sample size is 10.
- Posterior inference is performed using exact variable elimination.
- Machine-learning baselines are implemented using `scikit-learn`.
- Macro F1 is computed with `zero_division=0`.
- One-hot encoded categorical features are cast to floating-point values.
- `numpy.trapezoid` is used for numerical integration; use `numpy.trapz` only if running an older NumPy version.

---

## 8. Main reproduced results

The package reproduces the following manuscript-level numerical results.

| Result | Values |
|---|---|
| Gradient boosting full evidence macro F1 | 1.000 / 1.000 / 1.000 |
| Gradient boosting leakage-controlled macro F1 | 0.840 / 0.803 / 0.833 |
| Leakage scores | 0.160 / 0.197 / 0.167 |
| Bayesian network repeated 3x5-fold macro F1 | 0.741 / 0.757 / 0.732 |
| Bayesian network leave-one-geometry-out macro F1 | 0.677 / 0.642 / 0.674 |
| Hybrid model macro F1 | 0.841 / 0.820 / 0.834 |
| External P(significant damage) | 0.61-1.00 |
| External P(substantial stiffness loss) | 0.83-0.89 |

The target order is:

```text
Damage state / Stiffness loss / Residual capacity
```

---

## 9. Citation

If you use this repository, cite the associated manuscript:

```text
Wahab, A. G., Tao, Z., & Clementi, F. Leakage-controlled, mechanism-aware Bayesian condition assessment of RC frames using finite-element training data and experimental demonstration.
```


---

## 10. Contact

For questions about the repository or manuscript, contact the corresponding author listed in the manuscript.
