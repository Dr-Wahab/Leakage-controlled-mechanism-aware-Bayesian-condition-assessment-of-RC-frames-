================================================================
 Mechanism-Aware, Leakage-Controlled Bayesian SHM Framework
 Reproducible code package
================================================================

Reproduces the full analysis and figures for the paper
"Mechanism-Aware Bayesian Condition Assessment of RC Frames:
 Leakage-Controlled Finite-Element Training and Experimental Validation."

Contains: the numbered analysis pipeline (stages 01-07), the OpenSeesPy
finite element validation models (05a, 05b), the generated datasets
(so you can run without re-running OpenSees), and resumable runners for
the slow cross-validation step.

----------------------------------------------------------------
1. SETUP   (Python 3.11 or 3.12)
----------------------------------------------------------------
    python -m venv venv
    # Windows:        venv\Scripts\activate
    # macOS / Linux:  source venv/bin/activate
    pip install -r requirements.txt

Analysis only (skip openseespy):
    pip install numpy pandas scikit-learn matplotlib pgmpy

----------------------------------------------------------------
2. QUICK START  (reproduces all tables and figures)
----------------------------------------------------------------
The FE dataset is already in data/, so run the analysis directly:

    python run_all.py

Writes results/ (cv_results.csv, leakage_scores.csv,
table1_model_comparison.csv, table2_external_validation.csv,
hybrid_results.csv) and figures/ (fig1 ... fig9).

To also regenerate the 528-case FE dataset (slow; needs openseespy):
    python run_all.py --full

----------------------------------------------------------------
3. PIPELINE STAGES
----------------------------------------------------------------
config.py   shared paths, constants, plotting style, target names.

01_generate_fe_dataset.py            [needs openseespy; slow]
    528 nonlinear pushover analyses (fiber beam-column, Concrete02/Steel02)
    over full-scale, lab-scale, stilted frames, stratified by geometry,
    scale, mechanism.  -> data/fem_dataset_528.csv

02_labels_and_dataset_figures.py
    CODE-BASED target thresholds:
      DamageState      ASCE 41 drift   0.5 / 1 / 2 %  (none/minor/moderate/severe)
      StiffnessLoss    cracked-section 60 / 80 % loss (low/medium/high)
      ResidualCapacity binary failed / retained at 0.8 Vmax
    Builds categorical evidence incl. same-scalar leakage nodes
    (re-binned at the SAME code limit states as their target).
    -> data/fem_dataset_categorical.csv, figures/fig1, fig2

03_evaluate.py                       [slowest analysis step; minutes]
    Bayesian network (expert DAG, BDeu) vs tuned class-balanced baselines
    (random forest, HistGradientBoosting, logistic regression, dummy).
    Three evidence regimes; repeated 5-fold and leave-one-geometry-out.
    Computes the leakage score.
    -> results/cv_results.csv, leakage_scores.csv,
       table1_model_comparison.csv, figures/fig3,4,5,7

04_hybrid.py
    Gradient boosting augmented with Bayesian posteriors.
    -> results/hybrid_results.csv, figures/fig6

06_measured_features.py
    Scalar condition features from the four measured cyclic records.
    -> data/measured_frame_features.csv

07_external_validation.py
    Infers each specimen's condition; reports P(significant damage) and
    P(substantial stiffness loss).
    -> results/table2_external_validation.csv, figures/fig8, fig9

----------------------------------------------------------------
4. FINITE ELEMENT VALIDATION MODELS (OpenSeesPy)
----------------------------------------------------------------
05a_fe_existing_frame.py
    Existing deficient RC frame (EF, Wahab 2023). Full FEMA-461 cyclic
    protocol, robust recursive-bisection solver, genuine descending branch.
05b_fe_dj1_stilted.py
    Stilted frame DJ-1 (story-stiffness ratio 40, Li et al. 2023), backbone
    anchored to the digitized envelope and reported table values.
Run directly, e.g.:  python 05a_fe_existing_frame.py

----------------------------------------------------------------
5. RESUMABLE CROSS-VALIDATION HELPERS (optional)
----------------------------------------------------------------
If 03 is too slow to finish at once:
    python run_kfold_only.py    # -> results/_kfold_partial.csv (all 3 targets)
    python run_rc_only.py       # re-run only ResidualCapacity if needed
Then re-run 03 to assemble cv_results/leakage/table1 + figures from the
checkpoint plus the fast LOGO step.

----------------------------------------------------------------
6. DATA FILES
----------------------------------------------------------------
data/fem_dataset_528.csv            generated FE dataset (528 cases)
data/fem_dataset_categorical.csv    categorical encoding used by the models
data/measured_existing_frame.csv    EF measured cyclic record
data/measured_prestress_frame.csv   RF measured cyclic record
data/measured_DJ2_hysteresis.csv    DJ-2 measured cyclic record
data/DJ1_backbone_from_figure.csv   DJ-1 digitized backbone + table anchors
data/measured_frame_features.csv    extracted scalar features (stage 06)

----------------------------------------------------------------
7. ENVIRONMENT NOTES
----------------------------------------------------------------
- scikit-learn: f1_score uses zero_division=0; one-hot features cast to float.
- pgmpy 1.x: BDeu parameter learning; exact inference (VariableElimination).
- numpy: uses np.trapezoid (replace with np.trapz on older numpy).
- Reported numbers (code-based thresholds, tuned models, binary RC):
      leakage scores         ~ 0.17 / 0.19 / 0.17 (DS / SL / RC)
      GBM full -> controlled : 1.00 -> 0.84 / 0.80 / 0.83
      best baseline          : ~0.85 / 0.84 / 0.84
      Bayesian net (5-fold)  : 0.74 / 0.76 / 0.73
      Bayesian net (LOGO)    : 0.68 / 0.64 / 0.67
      hybrid                 : 0.84 / 0.82 / 0.83
      external validation    : P(damage) 0.61-1.00, P(stiffness) 0.83-0.89
================================================================
