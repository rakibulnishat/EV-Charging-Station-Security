# EV Charging Station Multi-Modal Attack Detection — Journal-Tier Final Summary

**Generated:** 2026-07-22 03:56
**Status:** Experimental work complete (Phases 1–4). Literature review expansion and manuscript writing remain.

---

## 1. What This Project Is

A controlled ablation study testing whether fusing three EVSE sensing modalities
(Power consumption, Host HPC/Kernel events, Network traffic) improves attack
detection over the best single modality alone, on the official CICEVSE2024
dataset (Canadian Institute for Cybersecurity, UNB). Upgraded from a conference-
tier evaluation (3 seeds, 2 models, 2 fusion strategies) to journal-tier depth
(10 seeds, 4 model families, 3 fusion strategies, formal statistical validation,
cross-device generalization, deployment-relevance analysis).

## 2. Why This Gap Existed

No peer-reviewed CICEVSE2024 study fuses all three modalities with an explicit
best-single-modality anchor under statistically rigorous multi-seed testing,
tests cross-device generalization, or reports deployment-relevant latency and
model size alongside accuracy. This project supplies all three.

## 3. Dataset

| Modality | Raw Rows | Clean Rows | Features | Classes |
|---|---|---|---|---|
| Power | 115,298 | 114,198 | 4 | 5 (all) |
| Host | 8,474 | 6,166 | 903 | 4 (no Backdoor) |
| Network | 2,744,700 | 274,519 | 40 | 3 (no Backdoor/Cryptojacking) |

Only Benign/Recon/DoS are visible to all three modalities. Backdoor is
Power-exclusive; Cryptojacking is Power+Host only.

## 4. How the Data Was Cleaned (full disclosure)

- **Power**: 1,100 exact-duplicate rows removed (data-collection artifact).
- **Host**: 6 corrupted/column-shifted rows removed; 2,302 dead/dropout rows
  (`State=='0'`) removed after HPC-counter analysis proved these were genuine
  measurement gaps, not mislabeled Benign traffic.
- **Network**: 2.74M raw flows aggregated into 274,519 windows; a construction
  artifact (`n_flows_in_window` + `_std` columns trivially separating classes
  by window size) was found and removed; a within-session leakage pattern was
  found and fixed via session/file-grouped splitting (documented Benign
  exception: only 2 source files).

## 5. Core Findings

### Finding 1 — Fusion does not beat the best single modality
| Config                            |     Mean |         Std |
|:----------------------------------|---------:|------------:|
| Network (anchor)                  | 0.999972 | 1.14912e-05 |
| Early fusion                      | 1        | 0           |
| Late fusion                       | 0.998314 | 2.34056e-16 |
| Intermediate fusion (autoencoder) | 0.999273 | 0.000766436 |

Early fusion ties the Network anchor exactly. Late fusion underperforms
consistently (10/10 seeds). Intermediate (autoencoder) fusion shows a bimodal
seed-dependent outcome. No fusion strategy improves on the anchor.

### Finding 2 — Model-family insights
| modality   | model              |     mean |      std |      min |      max |
|:-----------|:-------------------|---------:|---------:|---------:|---------:|
| host       | LogisticRegression | 0.990565 | 0        | 0.990565 | 0.990565 |
| host       | MLP                | 0.988855 | 0.001367 | 0.985736 | 0.990805 |
| host       | RandomForest       | 0.987455 | 0.000612 | 0.987165 | 0.988616 |
| host       | XGBoost            | 0.986385 | 0.00169  | 0.983546 | 0.989353 |
| network    | LogisticRegression | 0.999896 | 0        | 0.999896 | 0.999896 |
| network    | MLP                | 0.970484 | 0.017369 | 0.946101 | 0.999965 |
| network    | RandomForest       | 0.999972 | 1.2e-05  | 0.999948 | 0.999983 |
| network    | XGBoost            | 0.999983 | 0        | 0.999983 | 0.999983 |
| power      | LogisticRegression | 0.530877 | 0        | 0.530877 | 0.530877 |
| power      | MLP                | 0.695216 | 0.001223 | 0.69323  | 0.697072 |
| power      | RandomForest       | 0.71515  | 0.000822 | 0.714179 | 0.716946 |
| power      | XGBoost            | 0.715711 | 0.000972 | 0.714813 | 0.717974 |

Power's attack signal is NOT linearly separable (Logistic Regression 0.531 vs.
tree ensembles 0.715–0.716). Host's IS largely linear (LogReg 0.991, competitive
with or beating tree ensembles).

### Finding 3 — Cross-device generalization (novel)
| direction   |     mean |      std |      min |      max |
|:------------|---------:|---------:|---------:|---------:|
| A_to_B      | 0.99983  | 5e-06    | 0.999825 | 0.999834 |
| B_to_A      | 0.999632 | 0.000271 | 0.999287 | 0.999842 |

Network's Recon/DoS attack signatures transfer near-perfectly (≥0.9996 macro-F1)
between a real EVSE-A charger and its Raspberry-Pi-emulated EVSE-B counterpart.
Benign-class transfer is untestable (EVSE-B has zero Benign samples), disclosed
explicitly. Formal significance testing (Table below) shows the two directions
are not confidently distinguishable from each other, though both differ
trivially (<0.05 percentage points) from the pooled baseline.

| comparison       | test           |   statistic |    p_value |
|:-----------------|:---------------|------------:|-----------:|
| A_to_B vs B_to_A | Welch t-test   |     2.3118  | 0.0460842  |
| A_to_B vs B_to_A | Mann-Whitney U |    40       | 0.460807   |
| A_to_B vs Pooled | Welch t-test   |   -34.6524  | 4.6162e-13 |
| B_to_A vs Pooled | Welch t-test   |    -3.96534 | 0.00325188 |

### Finding 4 — Deployment relevance (novel)
| modality   | model              |   test_macro_f1 |   latency_ms_per_sample |   model_size_kb |
|:-----------|:-------------------|----------------:|------------------------:|----------------:|
| host       | LogisticRegression |        0.990565 |             0.00287613  |       22.0615   |
| host       | MLP                |        0.990805 |             0.00862449  |      985.926    |
| host       | RandomForest       |        0.987165 |             0.108344    |     2119.89     |
| host       | XGBoost            |        0.987182 |             0.0197484   |      643.299    |
| network    | LogisticRegression |        0.999896 |             0.000198486 |        1.82715  |
| network    | MLP                |        0.965451 |             0.000378439 |       26.4775   |
| network    | RandomForest       |        0.999983 |             0.00492088  |      762.079    |
| network    | XGBoost            |        0.999983 |             0.0033005   |      513.474    |
| power      | LogisticRegression |        0.530877 |             0.000180749 |        0.983398 |
| power      | MLP                |        0.695374 |             0.000181996 |        9.80957  |
| power      | RandomForest       |        0.716946 |             0.0410756   |   153091        |
| power      | XGBoost            |        0.714813 |             0.0264199   |     3899.4      |

Power's RandomForest reaches only 0.002 higher F1 than XGBoost at 39x the disk
size (150MB vs. 3.9MB) — an unexpected finding: fewer features can produce
LARGER trees, not smaller ones, since there's no clean early split available.

### Finding 5 — Backdoor (Power-exclusive), 10-seed rigor
| model        |   test_macro_f1_mean |   test_macro_f1_std |   f1_Backdoor_mean |   f1_Backdoor_std |
|:-------------|---------------------:|--------------------:|-------------------:|------------------:|
| RandomForest |             0.64351  |         0.000803195 |           0.717678 |       0.000818494 |
| XGBoost      |             0.657029 |         0.00104767  |           0.727709 |       0.00144331  |

Confirms the original 3-seed conference finding (0.7173) almost exactly.
New finding at this resolution: XGBoost (0.7277) is measurably, consistently
better than RandomForest (0.7177) for Backdoor detection specifically.

### Finding 6 — Robustness confirmed
Network's aggregation window size (5–50 flows) barely affects results
(0.00005 macro-F1 spread across a 10x range). The fixed hyperparameter
configuration used throughout was already near-optimal in 5/6 tested cases.

## 6. Methodological Rigor: Self-Corrected Issues

Six real issues were found and corrected during this notebook's own
self-checking, each disclosed rather than silently fixed:
1. Within-session leakage in Network's original train/test split (conference tier)
2. Accidental timestamp leak in Host's feature set (conference tier)
3. An overgeneralized "directionally consistent" claim about intermediate
   fusion, contradicted by its own 5/10-vs-5/10 seed split
4. A metric-averaging artifact in cross-device generalization's first result
   (inconsistent `f1_score` label denominators across seeds)
5. An overgeneralized "less stable" claim about a searched hyperparameter
   config, actually driven by one outlier seed
6. An undertrained MLP baseline in the inference-latency table (no early
   stopping), inconsistent with the project's own established MLP results

## 7. Assessment: Publication Tier

Realistically positioned for Q1/solid-Q2 venues (IEEE Access, IEEE IoT
Journal, Sensors, Computers & Security tier) given the rigor demonstrated.
Not automatically "high impact" top-tier (IEEE TDSC/TIFS) because: (a) the
central finding is a null result, harder to position as high-impact than a
working new method; (b) single-dataset scope limits generalization claims;
(c) no new fusion architecture is proposed, only evaluated. Two paths to
raise impact: a second EVSE/IoT-security dataset to test generalization, or
a fusion variant designed around the Power non-linear-separability finding.

## 8. Remaining Work

- [ ] Expand literature review from 20 to ~30–35 verified sources
- [ ] Full journal-length manuscript incorporating all findings above
