# Model card — CRISPR Designer on-target scorer

Hugging Face-style model card for the CRISPR Designer transformer.
Phase 2 smoke train (single seed, no hyperparameter sweep), trained
on the Doench 2016 corpus alongside a reproduced Rule Set 2 baseline
for head-to-head comparison.

## Model details

- **Architecture**: encoder-only transformer over one-hot encoded 30-mer
  sgRNA contexts, fused with a small MLP over two biological-context
  inputs (Percent Peptide, AA Cut position) before the regression head.
  - 4 layers, 4 heads, d_model=64, dim_feedforward=256, dropout=0.1
  - bio_hidden=16
  - mean-pool over positions, LayerNorm, concat with bio MLP output,
    linear head, sigmoid
- **Parameters**: ~204k
- **Input**:
  - 30 nt ACGT sequence (4 nt 5' flank + 20 nt protospacer + 3 nt NGG
    PAM + 3 nt 3' flank)
  - Optional: Percent Peptide (0–100), AA Cut position (1–~2800),
    both z-scored using training-set statistics persisted in
    `ours_meta.json`. Missing inputs default to the training mean.
- **Output**: scalar predicted on-target activity in [0, 1]
- **License**: MIT (training code) + CC-BY (model weights, per the
  upstream Doench 2016 supplementary licensing chain)

## Intended use

- **Primary use**: on-target sgRNA efficiency prediction for SpCas9
  with NGG PAM, in mammalian cell contexts comparable to Doench's
  screens.
- **Out of scope**:
  - Off-target prediction
  - Non-NGG PAMs / Cas variants beyond SpCas9
  - Bacterial, plant, or yeast genomes (training data is human/mouse;
    Phase 3 will address bacterial separately with a retrained model)
  - Absolute cleavage rate (target is a within-gene rank)

## Training data

See `docs/data_card.md`. Summary: Doench 2016 corpus, 5,310 sgRNAs
across 17 human/mouse genes, 4 drug-selection conditions. Splits:
leave-genes-out 9/4/4 with greedy LPT bin-packing, seed=42, documented
in `ml/data/splits/SPLIT_NOTES.md`.

## Training procedure

- **Optimizer**: AdamW, lr=3e-4, weight_decay=1e-4
- **Schedule**: linear warmup (5 epochs) → cosine decay to 0 over 60
  epochs cap
- **Batch size**: 64
- **Loss**: MSE on rank-normalised target
- **Gradient clipping**: max norm 1.0
- **Early stopping**: patience=15 epochs on val Spearman
- **Hardware**: CPU (Intel laptop), training time ~4 min
- **Seed**: 42
- **Actual training**: early-stopped at epoch 39, best val_spearman at
  epoch 24

Hyperparameters were not tuned. A sweep is deferred — the smoke train
landed in a defensible "comparable to baseline" regime and the
honest portfolio story preserves that without overfitting.

## Evaluation

Head-to-head against the reproduced Rule Set 2 baseline on the same
leave-genes-out test split (CUL3, HPRT1, NF2, TADA1; 550 rows). All
CIs are bootstrap n=1000 95% intervals; bootstrap seed shared between
models for fair comparison. Source: `ml/eval_common.py` consumed by
both `eval_baseline.py` and `eval_ours.py`.

| Metric | Baseline (xgboost + bio) | **Ours (transformer)** | Paper (Doench 2016) |
|---|---|---|---|
| Spearman ρ | +0.459 [+0.389, +0.527] | **+0.420 [+0.343, +0.488]** | ~0.55 |
| Pearson r | +0.471 [+0.404, +0.531] | **+0.449 [+0.379, +0.513]** | — |
| AUC (>median) | +0.670 [+0.616, +0.719] | **+0.669 [+0.613, +0.720]** | — |
| Reproduction-vs-Azimuth | +0.825 | +0.731 | — |

CIs heavily overlap on every metric; AUC is identical to three
decimals. The result is "statistically indistinguishable from the
baseline" rather than a clean win for either model.

### Stratified by protospacer GC bucket

| GC bucket | n | Baseline Spearman | Ours Spearman |
|---|---|---|---|
| low (<0.40) | 117 | +0.519 | **+0.535** |
| mid (0.40–0.70) | 397 | **+0.402** | +0.373 |
| high (≥0.70) | 36 | **+0.552** | +0.164 |

The transformer slightly outperforms baseline on AT-rich gRNAs,
matches it in the bulk middle, and underperforms substantially on
high-GC. The high-GC bucket's CI is enormous (n=36 only), so the
central estimate may be partly noise, but it's the model's clearest
documented weakness.

## Limitations

- **Small training corpus** (~4.2k training rows after splits). Variance
  on held-out test is correspondingly large; bootstrap CIs are wide.
- **Mammalian only.** Trained on human/mouse cell-line data; predictions
  for bacterial, plant, or yeast genomes are out-of-distribution.
- **No hyperparameter sweep.** Architecture defaults were not tuned;
  a proper sweep might close the central-estimate gap to baseline.
- **High-GC weakness.** Substantially worse than baseline on the 36
  test rows with protospacer GC ≥0.70. Likely related to corpus
  imbalance + small parameter count.
- **Sequence + minimal context only.** Chromatin accessibility,
  expression level, and off-target genomic context are not represented.

## Citation

If you use this model, cite Doench 2016 (DOI 10.1038/nbt.3437) and
this repository.

## Publishing checklist (when ready)

- [ ] Hugging Face Hub username confirmed (`SleepyKomodo`)
- [ ] Weights exported in `safetensors` format
- [ ] Repository created under that namespace
- [ ] This model card mirrored to the HF Hub repo
- [ ] README links to the HF Hub repo

---

*Phase 2 smoke train completed 2026-05-16. See `docs/baseline_results.md`
and `docs/ours_results.md` for the source numbers feeding the table
above.*
