# Bacterial Phase 3 results

Head-to-head comparison on the crisprHAL WT-SpCas9 held-out test
split (6,749 rows). All CIs are bootstrap n=1000
95%; all models scored on identical predictions vs. raw
log-enrichment via `ml/eval_common.py`.

## Test-set metrics

| Model | Spearman ρ | Pearson r | AUC (>med) |
|---|---|---|---|
| Baseline (xgboost, 870 features) | +0.626 [+0.608, +0.642] | +0.630 [+0.612, +0.647] | +0.826 [+0.816, +0.835] |
| Ours, Path A (Phase-2 mammalian → bacterial) | +0.530 [+0.512, +0.548] | +0.533 [+0.514, +0.551] | +0.780 [+0.769, +0.790] |
| Ours, Path B (fresh transformer) | +0.544 [+0.526, +0.562] | +0.546 [+0.529, +0.565] | +0.787 [+0.777, +0.798] |

## Stratified by protospacer GC bucket

Spearman ρ per GC bucket. Buckets are low (<0.40), mid (0.40–0.70), high (≥0.70).

| Model | low (n / ρ) | mid (n / ρ) | high (n / ρ) |
|---|---|---|---|
| Baseline (xgboost, 870 features) | 985 / +0.552 [+0.505, +0.598] | 5416 / +0.625 [+0.608, +0.642] | 348 / +0.711 [+0.646, +0.762] |
| Ours, Path A (Phase-2 mammalian → bacterial) | 985 / +0.462 [+0.411, +0.515] | 5416 / +0.528 [+0.509, +0.548] | 348 / +0.547 [+0.465, +0.621] |
| Ours, Path B (fresh transformer) | 985 / +0.475 [+0.425, +0.524] | 5416 / +0.544 [+0.525, +0.563] | 348 / +0.555 [+0.474, +0.628] |

## Notes

- Test split is crisprHAL's published test set, verbatim from
  Ham et al. 2023.
- Each transformer has ~{'spearman': MetricCI(value=0.5439558945801248, lo=0.5260325583682686, hi=0.5615025505822943), 'pearson': MetricCI(value=0.5462328195571899, lo=0.5286111548542977, hi=0.5645326405763627), 'auc': MetricCI(value=0.7872180289358712, lo=0.7770247476432888, hi=0.7979024841633894)} … see ours_bacterial_*_meta.json for hyperparams.
- Path A weights are initialised from `ml/checkpoints/ours_best.pt`
  (Phase 2 mammalian model); only the encoder + embedding tensors
  whose shapes match are transferred. Positional embedding (30→43)
  and head are re-initialised. The bio_mlp branch is dropped.
- Target during training was rank-normalised to [0,1]. Metrics
  here are computed against raw log-enrichment because Spearman
  and AUC are rank/threshold-based.

## Citations

Ham DT et al. (2023) *Nature Communications* 14, 5514.
DOI: 10.1038/s41467-023-41143-7
