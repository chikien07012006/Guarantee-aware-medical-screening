# Two-week project plan

**Period:** 19 November 2026–2 December 2026 (Asia/Seoul)  
**Goal:** Produce a reproducible baseline-to-guarantee-aware pipeline and initial experimental results for the project proposal.

## Ownership

Adapt the labels to the actual team members. The split is designed for a three-person team; for a smaller team, keep the ownership areas but combine roles.

| Role | Primary responsibility | Review responsibility |
| --- | --- | --- |
| A — Data & reproducibility | ISIC ingestion, patient-level splits, augmentation, experiment configs | Validate no patient leakage and data documentation |
| B — Models & training | ResNet/ViT baselines, imbalance handling, primal-dual loss | Reproduce a baseline from saved config |
| C — Calibration & evaluation | NP umbrella calibration, metrics, repeated splits, plots | Validate calibration/test separation and reported intervals |

## Week 1 — Working baselines and correct calibration

| Date | A — Data & reproducibility | B — Models & training | C — Calibration & evaluation | Shared deliverable |
| --- | --- | --- | --- | --- |
| Thu 19 Nov | Create data contract and directory structure; document ISIC download/source | Create package skeleton and CPU smoke-test model | Implement FNR, FPR, sensitivity, specificity, AUROC unit tests | Repository conventions, seed policy, and task board agreed |
| Fri 20 Nov | Implement patient-level 60/20/20 split; save patient IDs and class counts | Implement ResNet-50 BCE training loop with weighted sampler | Implement NP rank calculation and threshold API | Leakage test and calibration-unit tests pass |
| Sat 21 Nov | Add image transforms and data-loader tests | Train/debug MedMNIST smoke test; log metrics | Verify minimum-positive rule and edge cases for NP calibration | End-to-end toy-data run |
| Sun 22 Nov | Prepare ISIC manifest and metadata audit | Run ResNet-50 BCE and weighted-BCE pilot | Add conventional versus NP-calibrated threshold comparison | First pilot metrics table |
| Mon 23 Nov | Freeze split v1 and write data README | Add focal-loss baseline and checkpointing | Add specificity-at-guaranteed-threshold and biopsy/1k metrics | Reproducible baseline command/config |
| Tue 24 Nov | Audit duplicates, corrupt files, and patient overlap | Start ViT-S/16 fine-tuning pilot | Implement repeated calibration/test evaluation driver | Baseline results review |
| Wed 25 Nov | Support data fixes; tag dataset manifest v1 | Finish baseline runs or document compute limitations | Produce score-distribution and ROC plots | **Week-1 checkpoint:** baseline comparison and calibration validation |

## Week 2 — Guarantee-aware training and analysis

| Date | A — Data & reproducibility | B — Models & training | C — Calibration & evaluation | Shared deliverable |
| --- | --- | --- | --- | --- |
| Thu 26 Nov | Add configuration for calibration-size sweep | Implement differentiable FNR/FPR surrogates and dual update | Compute `alpha_eff` from calibration-positive count | Primal-dual toy test passes |
| Fri 27 Nov | Check class balance per split and sampler behavior | Implement nominal-level and `alpha_eff` constrained models | Validate threshold is selected only after training | First constrained-training pilot |
| Sat 28 Nov | Add deterministic-run and artifact manifests | Implement positive-score memory bank and soft quantile-gap loss | Add guarantee violation-rate aggregation with confidence intervals | Quantile-gap ablation ready |
| Sun 29 Nov | Launch/monitor configured runs; record resource use | Run no-regularizer, quantile-gap, and CVaR alternatives | Run repeated-split evaluation (start with a small debug count) | Preliminary ablation table |
| Mon 30 Nov | Verify all result artifacts map to configs and commits | Complete ResNet experiments; run ViT where compute permits | Scale repeated evaluation toward 500 splits; create plots | Results table and figures v1 |
| Tue 1 Dec | Curate final manifests and reproducibility instructions | Prepare Grad-CAM examples for false negatives | Analyse guarantee cost, failure cases, and limitations | Draft results narrative |
| Wed 2 Dec | Final data/leakage review | Code cleanup and rerun selected best configuration | Final metric verification and figure export | **Two-week milestone:** reproducible code, initial results, and report-ready figures |

## Acceptance criteria

- Patient IDs never overlap among training, calibration, and test sets.
- The trained model is fixed before NP threshold calibration begins.
- Each run logs seed, split IDs, `alpha`, `delta`, number of calibration positives, `alpha_eff`, selected threshold, and package/config versions.
- Baselines and the proposed method use the same evaluation protocol.
- Report both utility (specificity/FPR, AUROC, biopsies per 1,000) and safety (test FNR, sensitivity, guarantee violation rate).
- Clearly label exploratory, debug-scale results separately from the planned 500-split estimate.

## Risks and decisions

- **Too few positive calibration examples:** report the count; do not claim the stated guarantee when the minimum sample requirement is unmet.
- **GPU or time constraints:** prioritize ResNet-50, then run ViT-S/16 as a smaller pilot; do not mix protocols when comparing them.
- **Unstable constrained optimization:** tune temperature, dual learning rate, and slack on training/validation only; retain BCE and nominal-level baselines.
- **No gain from the regularizer:** report the negative result and ablations honestly.
