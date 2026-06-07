# CRISPR Designer

> **Showcase repository — research code, not production.**
> This public repository presents the methodology, headline results, and
> technical report for the CRISPR Designer project. The **full source
> (backend, ML pipeline, frontend, tests) is private and available on
> request** (see Contact). Results are reproducible from the private
> repository against the corpus snapshots in [`docs/data_card.md`](docs/data_card.md)
> with pinned random seeds. Known limitations are in [`LIMITATIONS.md`](LIMITATIONS.md).

A web-based CRISPR-Cas9 single-guide-RNA (sgRNA) designer where the on-target
efficiency scorers are **two models trained head-to-head against the published
Rule Set 2 baseline** on mammalian and bacterial corpora. Paste a gene sequence,
get a ranked list of candidate sgRNAs scored by models whose training curves and
bootstrap CIs are documented in `docs/`.

## Headline results

Bootstrap 95% CIs from n=1000 resamples on the held-out test splits.

| Phase | Corpus | Baseline (xgboost) | Ours (transformer) | Outcome |
|---|---|---|---|---|
| 1+2 | Doench 2016 (5,310 mammalian sgRNAs) | ρ **+0.459** [+0.389, +0.527] | ρ +0.420 | **tied** (CIs overlap) |
| 3 | crisprHAL WT-SpCas9 (33,743 *E. coli* sgRNAs) | ρ **+0.626** | ρ +0.544 | **xgboost wins cleanly** (non-overlapping CIs) |

**Reproduction validation:** the Rule Set 2 reproduction reaches Spearman ρ +0.825
against the published Azimuth V3 scorer, confirming the baseline replication is
faithful. The +0.459 baseline falls inside the bootstrap CI of the paper's
reported ~0.55.

**Cross-kingdom transfer learning** (mammalian-pretrained → bacterial fine-tune)
produced **no improvement** over fresh initialisation, suggesting CRISPR
efficiency signal is largely organism-specific at the 200k-parameter scale
tested. The deployed designer routes by organism accordingly.

Full breakdown in `docs/`: `baseline_results.md` (Rule Set 2 reproduction),
`model_card.md` / `model_card_bacterial.md` (architecture + per-corpus
performance), `bacterial_results.md` (cross-kingdom transfer), `data_card.md` /
`data_card_bacterial.md` (corpora + SHA256 provenance).

## Why this project exists

CRISPR-Cas9 sgRNA efficiency prediction is a useful in-silico screen for
narrowing guide candidates before wet-lab validation. The Doench 2016 Rule Set 2
model is the standard mammalian baseline, but the literature contains few rigorous
head-to-head comparisons against newer architectures on controlled splits, and
bacterial sgRNA scoring is underexplored. This project addresses both gaps with
three methodological choices:

- **Faithful Rule Set 2 reproduction.** XGBoost over 612 hand-engineered features
  at Spearman ρ +0.825 against the published Azimuth V3 scorer on leave-genes-out
  splits — the baseline replication has to hold before any head-to-head can mean
  anything.
- **Head-to-head on identical splits across two corpora.** A 200k-parameter
  encoder-only transformer trained against the baseline: statistically tied on
  Doench 2016 mammalian (overlapping bootstrap CIs); cleanly outperformed by
  xgboost on the larger crisprHAL *E. coli* corpus (non-overlapping CIs). **Both
  results are reported**, including the one where the transformer loses.
- **Cross-kingdom transfer, tested and negative.** Mammalian-pretrained →
  bacterial fine-tune gave no improvement over fresh initialisation. Reported
  honestly rather than dropped.

## What's in the full project

The private source repository contains:

```
backend/        FastAPI app — /health, /score, /design
ml/             data, scripts, features, eval, checkpoints
frontend/       React + Vite + TypeScript designer UI
tests/          pytest unit + route tests (44 passing)
docs/           data cards, baseline + model cards, results (mirrored here)
report/         technical report (PDF in this repo)
```

End-to-end reproduction runtime: ~3 h on CPU, ~30 min on GPU.

**Source available on request.** The full code, trained checkpoints, and
reproduction pipeline can be shared with hiring teams or collaborators — email
below. It is kept private for now because the Phase-3 bacterial training code is
still being cleaned up for release; the methodology and results here are complete.

## Technical report

The full write-up — methodology, every result with CIs, and the negative results
— is in [`report/technical_report.pdf`](report/technical_report.pdf).

## In scope / out of scope

**In scope (v0.1):** SpCas9 with NGG PAM; on-target efficiency scoring for human
(Doench 2016) and *E. coli* (crisprHAL); Rule Set 2 reproduction as baseline; a
200k-parameter encoder-only transformer trained head-to-head.

**Out of scope (v0.1):** genome-wide off-target search (v2); Cas variants beyond
SpCas9 (Cas12/13, base/prime editors); organisms beyond the two corpora;
LLM-generated explanation panel (v1.5).

See [`LIMITATIONS.md`](LIMITATIONS.md) for the full caveats.

## Citation

If you use this work, please cite the technical report:

```
González Lozano, D. (2026). CRISPR Designer: Trained sgRNA efficiency scorers
against measured Rule Set 2 baselines on mammalian and bacterial corpora.
Zenodo (DOI pending deposit; added on first tagged release).
```

A machine-readable citation lives in [`CITATION.cff`](CITATION.cff).

## Acknowledgments

- Doench, J. G. *et al.* (2016). *Optimized sgRNA design to maximize activity and
  minimize off-target effects of CRISPR-Cas9.* Nature Biotechnology, 34, 184–191.
  <https://doi.org/10.1038/nbt.3437>
- crisprHAL — the WT-SpCas9 *E. coli* corpus used for the bacterial comparison.
  Acquisition and SHA256 archived in `docs/data_card.md`.

## License

MIT. See [`LICENSE`](LICENSE).

## Contact

Daniel González Lozano · `sleepy.komodo@protonmail.com` ·
[ORCID](https://orcid.org/0009-0002-1737-276X)

Postdoctoral researcher, Tecnológico de Monterrey × Technical University of Denmark.
