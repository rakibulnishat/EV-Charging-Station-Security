# EV Charging Station Security: Multimodal Fusion for Intrusion Detection

Code and results for the study *Multimodal Fusion for Intrusion Detection in Electric Vehicle Charging Infrastructure: Accuracy Equivalence and Resilience to Channel Degradation* (under review at Computers and Electrical Engineering).

This repository contains analysis code and result files only.

## Data

All experiments use the public **CICEVSE2024** dataset from the Canadian Institute for Cybersecurity, University of New Brunswick:
https://www.unb.ca/cic/datasets/evse-dataset-2024.html

The dataset is not redistributed here. Download it from the link above and set the data path at the top of each notebook.

## Notebooks

| Notebook | Contents |
|---|---|
| `J_EVSE.ipynb` | Main pipeline: cleaning, per-modality baselines (Logistic Regression, Random Forest, XGBoost, MLP), early / late / intermediate fusion, ten-seed statistics, cross-device transfer, sensitivity checks, feature attribution, deployment cost |
| `EVSE_Fusion_Project.ipynb` | Earlier project notebook |
| `notebooks/EVSE_R1_revision.ipynb` | Revision analyses (Cells 1–7, listed below) |

### Revision analyses (`notebooks/EVSE_R1_revision.ipynb`)

| Cell | Analysis |
|---|---|
| 1 | Matched-sample comparison: every Part 1 arm, including network-only detection at two training budgets, scored on the same 656 test samples over 10 model seeds |
| 2 / 2b | Exact-duplicate audit of network feature vectors (rounded to 6 dp) and network-only performance on duplicate-free test rows |
| 3 | Part 2 equivalence design: 5 partitions × 5 pairing seeds × 5 model seeds (125 runs); TOST against a ±0.005 macro-F1 margin (±0.010 sensitivity), Holm adjustment, cluster bootstrap over partitions |
| 4 | Fusion under channel degradation: Gaussian noise (0.5, 1, 2 training SD) or a missing network channel at test time; uniform, validation-weighted, stacked, calibrated, gated, and availability-aware combiners; 5 pairing seeds |
| 5 | Per-condition comparison with host-only detection (Holm within condition); grouped feature removal (20 NFStream groups, leave-one-group-out and greedy cumulative) |
| 6 | Cross-device false-alarm rates with 0/5/10/20 target-device benign samples; leave-one-benign-file-out validation with Wilson intervals |
| 7 | Random Forest vs XGBoost equivalence over 10 paired seeds (TOST, ±0.005); power-model size and latency measurement |

## Results

`results/revision_R1/` holds the result files written by the revision notebook (CSV tables and logs).

## Reproducibility notes

- Model seeds, pairing seeds, and partition seeds are fixed in each notebook cell.
- Model size is the file written by `joblib.dump` (PyTorch state dictionary for the MLP); latency is the wall-clock time of 50 repeated full-test-set predictions divided by the number of predictions, on a Google Colab CPU runtime.
- Cross-modal pairing is class-conditional: the public release has no shared temporal key across power, host, and network data.

## License

MIT (see `LICENSE`).
