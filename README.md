# Guarantee-Aware Medical Screening

Deep-learning project for melanoma screening with a finite-sample sensitivity guarantee. The project trains an image classifier and calibrates its deployment threshold with the Neyman–Pearson (NP) umbrella algorithm so that the false-negative rate (FNR) is controlled at the required confidence level.

## Objective

For a melanoma score `s(x)` and threshold `t`, minimize the false-positive rate (FPR) while targeting a sensitivity of at least 95%:

```text
minimize    FPR(theta, t)
subject to  FNR(theta, t) <= alpha = 0.05
```

The final threshold is selected on an independent calibration split to target the finite-sample guarantee `P(FNR(theta, t_hat) > alpha) <= delta`, with `delta = 0.05`.

## Method overview

1. Split ISIC 2020 data at patient level into training (60%), calibration (20%), and test (20%) sets.
2. Fine-tune ImageNet-pretrained ResNet-50 and ViT-S/16 classifiers using class-balanced sampling and image augmentation.
3. Compare BCE, weighted BCE/focal loss, partial-AUC, nominal-level Lagrangian, and conformal risk-training baselines.
4. Train the proposed model with a primal-dual objective at an effective FNR level `alpha_eff`.
5. Select the deployment threshold only from calibration positives with NP umbrella calibration.
6. Evaluate specificity, empirical guarantee violations, AUROC, guarantee cost, and unnecessary biopsies per 1,000 screened patients over repeated splits.

The calibration split must have at least 59 positive examples for `alpha = delta = 0.05`.

## Repository layout

```text
.
├── data/                 # Local datasets; never commit raw data
├── notebooks/            # Exploration and visual analysis
├── src/
│   ├── data/             # Dataset, patient-level split, transforms
│   ├── models/           # ResNet-50 and ViT model wrappers
│   ├── training/         # Baselines and primal-dual training loop
│   ├── calibration/      # NP umbrella threshold selection
│   ├── evaluation/       # Metrics, repeated-split experiments
│   └── visualization/    # Score plots and Grad-CAM
├── configs/              # Reproducible experiment configurations
├── tests/                # Unit tests for split, calibration, and metrics
├── outputs/              # Local runs, checkpoints, figures
├── plan.md               # Two-week execution plan
└── requirements.txt
```

## Environment setup

Python 3.10 or newer is required. A virtual environment is created separately; activate it and install the dependencies yourself:

```bash
source ~/.venv/bin/activate
pip3 install -r requirements.txt
```

Verify the active interpreter before running experiments:

```bash
python --version
python -c "import torch, timm; print(torch.__version__, timm.__version__)"
```

If using Jupyter, install and register a kernel after activating the environment:

```bash
python -m pip install ipykernel
python -m ipykernel install --user --name=medical-screening --display-name "Python (Medical Screening)"
```

Select **Python (Medical Screening)** in the notebook UI.

## Data and reproducibility

- Use ISIC 2020 as the primary dataset; MedMNIST v2 can be used for rapid debugging.
- Preserve patient-level isolation across train, calibration, and test splits.
- Never use the test split for model selection or threshold selection.
- Save split patient IDs, random seed, package versions, configuration, and git commit with each run.
- Treat the statistical guarantee as conditional on a fixed trained model and i.i.d. calibration and deployment examples; distribution shift can invalidate it.

## Planned experiments

- Backbones: ResNet-50 and ViT-S/16.
- Main comparison: conventional training versus guarantee-aware primal-dual training.
- Ablations: `alpha` versus `alpha_eff`, quantile-gap versus CVaR/no regularizer, backbone, and calibration-set size.
- Qualitative analysis: score distributions, Lagrange-multiplier trajectory, and Grad-CAM for false negatives.

See [plan.md](plan.md) for owners, deliverables, and the two-week schedule.
