# Phase 2 transformer (ours) results


## Setup

- Test split: 550 rows, 4 held-out genes (CUL3, HPRT1, NF2, TADA1)
- Architecture: 4 layers, d_model=64, 4 heads, FF=256, dropout=0.1, bio_hidden=16
- Parameter count: 202,641
- Inputs: 30-mer ACGT indices + 2 bio-context features (Percent Peptide, AA Cut position), z-scored on train stats
- Hyperparameters / loss / schedule: see header of `ml/scripts/train_ours.py`

## Metrics

All CIs are bootstrap (n=1000) 95% intervals.

| Metric | Reproduction | Azimuth V3 on same rows | Paper (Doench 2016) |
|---|---|---|---|
| Spearman rho | **+0.420 [+0.343, +0.488]** | +0.665 | ~0.55 |
| Pearson r | **+0.449 [+0.379, +0.513]** | +0.678 | — |
| AUC (>median) | **+0.669 [+0.613, +0.720]** | +0.783 | — |

## Stratified by protospacer GC bucket

| GC bucket | n | Spearman rho |
|---|---|---|
| low | 117 | +0.535 [+0.387, +0.643] |
| mid | 397 | +0.373 [+0.282, +0.464] |
| high | 36 | +0.164 [-0.167, +0.480] |

## Reproduction sanity check

How tightly our predictions track Azimuth V3 on the same test set
(closer to 1.0 means we are replicating Rule Set 2 rather than
fitting to noise):

- Spearman (ours vs. Azimuth): +0.731
- Pearson  (ours vs. Azimuth): +0.742

## Commentary

Phase 2 transformer with bio-context inputs equalised to the
baseline. Compare against `baseline_results.md` for the apples-to-
apples head-to-head on the same test split (CUL3, HPRT1, NF2, TADA1),
same bootstrap seed, same metric machinery.

Notes:
- The transformer has roughly the same input information set as the
  baseline now. Any remaining gap reflects architecture + capacity
  + corpus-size, not feature scope.
- Reproduction-vs-Azimuth Spearman tells us how much of Rule Set 2's
  ranking we recover. Values near 0.8 mean we replicate Rule Set 2;
  significantly lower means we learned a different mapping that
  happens to perform similarly.
