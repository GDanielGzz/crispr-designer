# Model card — bacterial CRISPR-Cas9 on-target scorer (Phase 3)

Hugging-Face-style model card for the bacterial scorer deployed in
the CRISPR Designer's *E. coli K-12* path.

**Headline:** Phase 3 ran a 3-way head-to-head on the crisprHAL
WT-SpCas9 corpus. xgboost over 870 hand-engineered Rule Set 2 features
beat both transformer variants by a clean, statistically significant
margin. The xgboost model is what the v1 designer uses for E. coli.

## Model details

- **Type:** XGBRegressor over 870 hand-engineered features extracted
  from a 43-nt sequence window (10 + 20 + 3 + 10).
- **Features:** Rule Set 2-style — position-specific mono/dinucleotide
  one-hots, position-independent counts, regional Tms via Biopython,
  GC content of the 20-mer protospacer, TTTT flag. Implemented in
  `backend/services/features_bacterial.py`.
- **Hyperparameters:** see `XGB_PARAMS` in
  `ml/scripts/train_baseline_bacterial.py`. Notable: n_estimators=500
  with early stopping (best at iteration 355), max_depth=6,
  learning_rate=0.08.
- **Output:** scalar predicted log-enrichment activity (unbounded,
  range ~[-9, +6]).
- **License:** MIT for the code; the trained weights are derivative
  of the crisprHAL dataset and inherit its terms (see
  `docs/data_card_bacterial.md`).

## Intended use

- On-target sgRNA efficiency prediction for SpCas9 with NGG PAM in
  *E. coli* and closely related Gram-negative bacterial contexts.
- Out of scope: *Bacillus subtilis* and other Gram-positives (Phase 4
  stretch goal), non-NGG PAMs, off-target prediction, eukaryotic
  contexts (use the mammalian path for those).

## Training data

See `docs/data_card_bacterial.md`. Summary: crisprHAL WT-SpCas9
corpus from Ham et al. 2023 (Nat Commun 14, 5514). 33,743 sgRNAs in
378-nt context windows, with the labeled target's PAM always at
positions 209-211. We crop to 43-mer windows at the canonical PAM
offset.

Splits:
- train: 24,295 rows (10% of the published train split held out for
  validation, seed=42)
- val: 2,699 rows
- test: 6,749 rows (crisprHAL's published test set, untouched)

## Evaluation

Test-set head-to-head against two transformer variants. All CIs are
bootstrap n=1000 95%. Source: `ml/eval_common.py`, consumed by
`eval_baseline_bacterial.py` and `eval_ours_bacterial.py`.

| Model | Spearman ρ | Pearson r | AUC (>median) |
|---|---|---|---|
| **xgboost baseline (deployed)** | **+0.626 [+0.608, +0.642]** | **+0.630 [+0.612, +0.647]** | **+0.826 [+0.816, +0.835]** |
| Transformer, Path B (fresh) | +0.544 [+0.526, +0.562] | +0.546 [+0.529, +0.565] | +0.787 [+0.777, +0.798] |
| Transformer, Path A (Phase-2 → bact) | +0.530 [+0.512, +0.548] | +0.533 [+0.514, +0.551] | +0.780 [+0.769, +0.790] |

CIs do not overlap between xgboost and either transformer — this is a
real, statistically significant performance gap of ~0.08 Spearman.

### Stratified by protospacer GC bucket

| GC bucket | n | xgboost | Path B | Path A |
|---|---|---|---|---|
| low (<0.40) | 985 | +0.552 | +0.475 | +0.462 |
| mid (0.40-0.70) | 5,416 | +0.625 | +0.544 | +0.528 |
| high (≥0.70) | 348 | **+0.711** | +0.555 | +0.547 |

Notable: xgboost is **strongest in the high-GC bucket** — the opposite
of what we saw in the mammalian model card, where high-GC was the
*weakest* slice. Bacterial CRISPR lacks the chromatin accessibility
confounder that hurts mammalian high-GC prediction.

## Why xgboost won (the interesting part)

Phase 2 found mammalian transformer and xgboost statistically tied.
Phase 3 found bacterial xgboost cleanly beats transformer. The contrast
points to three plausible factors:

1. **Bacterial signal is cleaner.** No chromatin, no expression-state
   confounders. Hand-engineered Rule Set 2 features capture more of
   the variance because the variance the features were designed for is
   most of what's there.
2. **Crop loss.** crisprHAL's published model uses the full 378-nt
   context. We cropped to 43-mer to fit a transformer that matches
   Phase 2's architecture. The 335 nt we threw away may carry real
   signal — the published paper's CNN+RNN was designed for long
   context for a reason.
3. **Transformer capacity sweet spot.** A 200k-parameter transformer
   on a clean ~24k-row corpus is plausibly in a regime where
   gradient-boosted trees over good features are simply harder to
   beat. crisprHAL's authors used CNN+RNN, not transformer, in their
   *Nature Communications* paper — they almost certainly hit the same
   wall and chose a different architecture for that reason.

Mammalian → bacterial transfer (Path A) did *not* help; Path A was
slightly worse than fresh init (Path B) at 0.530 vs 0.544, though
within bootstrap CI. The encoder representations the Phase-2 model
learned on Doench 2016 don't carry usable bacterial signal.

## Limitations

- **E. coli only.** Training data is E. coli K-12; Gram-positive
  (*Bacillus*, *Corynebacterium*) extrapolation is unvalidated.
- **SpCas9 / NGG only.** Other Cas variants and PAM conventions are
  out of scope.
- **Predicts log-enrichment rank, not absolute cleavage rate.**
- **No off-target search.** Designer outputs on-target scores only.
- **Cropped window (43-mer of 378-mer).** As discussed above, full
  long-context modeling is deferred.

## Phase 4 stretch goals

- *Bacillus subtilis* support, via fine-tuning the xgboost / training
  a CNN+RNN on a curated B. subtilis dataset.
- Full 378-mer transformer (or CNN+RNN replication of crisprHAL's
  architecture) to test whether the crop-loss hypothesis is right.
- Cas-OFFinder integration for off-target search on bacterial
  genomes.

## Citation

If you use this model, cite both Doench 2016 (for the methodology
heritage) and Ham 2023 (for the bacterial training corpus):

> Ham, D. T., Browne, T. S., Banglorewala, P. N., Wilson, T. L.,
> Michael, R. K., Gloor, G. B., & Edgell, D. R. (2023). A
> generalizable Cas9/sgRNA prediction model using machine transfer
> learning with small high-quality datasets. *Nature Communications*,
> 14, 5514. https://doi.org/10.1038/s41467-023-41143-7

---

*Phase 3 completed 2026-05-16. See `docs/bacterial_results.md` for
the raw eval outputs and `docs/data_card_bacterial.md` for the
training corpus details.*
