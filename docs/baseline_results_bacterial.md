# Bacterial xgboost baseline results (Phase 3)


## Setup

- Test split: 6,749 rows (crisprHAL's published test set, verbatim)
- Feature count: 870 (Rule Set 2 features over the 43-mer window)
- Target: raw log-enrichment activity (unbounded, range ~[-9, +6])
- AUC binary label: score above test-set median
- Hyperparameters: see `XGB_PARAMS` in `train_baseline_bacterial.py`

## Metrics

All CIs are bootstrap (n=1000) 95% intervals.

| Metric | Reproduction | Azimuth V3 on same rows | Paper (Doench 2016) |
|---|---|---|---|
| Spearman rho | **+0.626 [+0.608, +0.642]** | — | — |
| Pearson r | **+0.630 [+0.612, +0.647]** | — | — |
| AUC (>median) | **+0.826 [+0.816, +0.835]** | — | — |

## Stratified by protospacer GC bucket

| GC bucket | n | Spearman rho |
|---|---|---|
| low | 985 | +0.552 [+0.505, +0.598] |
| mid | 5416 | +0.625 [+0.608, +0.642] |
| high | 348 | +0.711 [+0.646, +0.762] |

## Commentary

Non-deep-learning bacterial baseline. crisprHAL's own published
model is a CNN+RNN trained on the same data; this xgboost number
is here as a head-to-head 'simple-features baseline' alongside
our transformer (Phase 3 Path A and Path B).

Notes:
- We did NOT rank-normalise the target. xgboost is scale-
  agnostic and Spearman is rank-based, so raw log-enrichment
  is fine for the metrics we report.
- GC stratification uses the 20-mer protospacer (positions
  189-208 in the 378-mer, equivalently positions 10-29 in the
  43-mer crop).
