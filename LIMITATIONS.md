# Known Limitations — CRISPR Designer

This document is the deliberate "what would a careful reviewer catch?"
pass for the project. The methodological discipline that makes head-to-
head model comparisons honest (bootstrap CIs, identical evaluation
pipelines, negative results reported) is the same discipline that
surfaces these caveats.

Last updated: 2026-05-23 (initial scaffolding by CodeNya)

---

## Data limitations

- **Corpus snapshot dates**: Doench 2016 (5,310 mammalian sgRNAs) and
  crisprHAL WT-SpCas9 (33,743 *E. coli* sgRNAs). See `docs/data_card.md`
  for the exact versions used.
- **Doench 2016** is mammalian (human + mouse cell lines) and biased
  toward the gene set used in that study; the trained models inherit
  that bias for the mammalian-side comparisons.
- **crisprHAL** is *E. coli* K-12 strain MG1655 derivative; generalization
  to other bacterial backgrounds (let alone *B. subtilis*, archaea, etc.)
  is unverified.
- **No cross-organism corpus.** The cross-kingdom transfer experiments
  used mammalian-pretrained → bacterial fine-tune on the two existing
  corpora; a curated multi-organism corpus would strengthen the
  conclusion, but one wasn't available at v0.1 scale.

## Evaluation limitations

- **Leave-genes-out cross-validation** on Doench 2016 (matches the
  baseline paper's evaluation protocol). On crisprHAL, the held-out
  test set is sgRNA-disjoint but does include same-gene sgRNAs in
  train/test — the literature norm for that corpus.
- **Bootstrap CIs at n=1000** on Spearman ρ. The non-overlapping CI
  comparison on crisprHAL (xgboost +0.626 vs transformer +0.544) is
  the cleanest evidence in the project; the overlapping CIs on Doench
  2016 are reported as "statistically tied" rather than as a winner.
- **Metric choice — Spearman ρ.** Pearson r is also reported in
  `docs/baseline_results.md` but Spearman is the headline because
  Doench 2016 efficiency is a ranked outcome. Choice trade-offs
  documented in the model card.
- **Reproduction-vs-Azimuth V3 Spearman of +0.825** validates the Rule
  Set 2 baseline replication. Without that anchor, the +0.459 baseline
  number would be uninterpretable.

## Model limitations

- **Transformer scale**: 200k parameters. Small. The "ties baseline on
  mammalian, loses on bacterial" result is bounded to this scale; a
  larger model might (or might not) close the gap on crisprHAL. Phase 4
  scopes a full long-context transformer to test this directly.
- **Cross-kingdom transfer did not transfer.** Mammalian-pretrained →
  bacterial fine-tune produced no improvement over fresh bacterial
  initialization. This is reported as a *negative result*, not hidden.
  Possible explanations (organism-specific signal, corpus-size mismatch,
  scale-insensitivity at 200k parameters) are discussed in the technical
  report but not experimentally separated.
- **Hyperparameter search** budget was bounded. Both xgboost and the
  transformer received the same compute envelope; neither saw an
  exhaustive search.
- **No off-target search.** v0.1 is on-target scoring only. Cas-OFFinder
  integration is a v2 stretch goal.

## Scope limitations

- **In scope (v0.1)**: SpCas9 with NGG PAM, on-target efficiency scoring
  for human (Doench 2016) and *E. coli* (crisprHAL).
- **Out of scope**: other Cas variants (Cas12, Cas13, base editors, prime
  editors), other PAMs, genome-wide off-target search, *B. subtilis* and
  other organisms beyond the two corpora.
- **Not a substitute for**: experimental validation of designed sgRNAs.
  Predicted efficiency is a starting point for guide selection, not a
  guarantee.

## Reproducibility caveats

- **Pinned dependencies** in `requirements.txt`. Python 3.11+.
- **Random seeds**: pinned at `ml/scripts/train_ours.py` (training) and
  `ml/eval/bootstrap.py` (evaluation). Independent.
- **Doench 2016 data acquisition** uses the original supplementary
  material from Doench *et al.* (2016); if the upstream link breaks,
  see `docs/data_card.md` for the SHA256 of the file we used.
- **crisprHAL data acquisition**: similar — SHA256 archived in
  `docs/data_card.md`.

## Known bugs and open questions

- *No open bugs at v0.1.* Full test suite passes (44 tests). Bugs found
  post-publication are tracked as GitHub Issues and resolved in
  versioned releases.

## Things I'm uncertain about (honest section)

- The "transformer ties on mammalian, loses on bacterial" result is
  surprising and I'd want a second pair of expert eyes on the
  evaluation pipeline before drawing strong conclusions. The reproduction
  Spearman of +0.825 against Azimuth V3 is reassuring but only
  validates the baseline, not the comparison.
- The negative cross-kingdom transfer result is informative but
  bounded by the 200k-parameter scale. At GPT-scale a different
  pattern is plausible — but verifying that would require resources
  outside v0.1's scope.
- The choice of crisprHAL as the bacterial corpus was driven by data
  availability. A different bacterial corpus (especially one with more
  diverse organisms or different efficiency assay format) could shift
  the conclusion.
- I'd want to compare against a third baseline — DeepCRISPR or a
  similar deep-learning published model — before claiming the
  comparison is fully methodologically anchored. Currently it's
  trained-model-vs-engineered-features-baseline; trained-vs-trained
  baseline is the natural next step.

---

## Resolved limitations (kept for history)

*None yet — first publication.*
