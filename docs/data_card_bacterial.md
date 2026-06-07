# Data card — crisprHAL WT-SpCas9 bacterial corpus

Phase 3 training data. Same shape of card as `docs/data_card.md`
(Doench 2016 mammalian corpus) so the two are directly comparable.

## Source

Ham, D. T., Browne, T. S., Banglorewala, P. N., Wilson, T. L.,
Michael, R. K., Gloor, G. B., & Edgell, D. R. (2023). *A
generalizable Cas9/sgRNA prediction model using machine transfer
learning with small high-quality datasets.* Nature Communications,
14, 5514. DOI: [10.1038/s41467-023-41143-7](https://doi.org/10.1038/s41467-023-41143-7).

Distributed via Tyler Browne's maintained crisprHAL repository:
https://github.com/tbrowne5/crisprHAL — the WT-SpCas9 train/test CSVs
under `data/`.

## License

The crisprHAL repository does not currently ship a LICENSE file, so
the data falls under default GitHub terms (all rights reserved). The
underlying *paper* is open access under the standard Nature
Communications [CC-BY 4.0 license](https://creativecommons.org/licenses/by/4.0/).
Use for research, reproduction, and portfolio publication is clearly
within the spirit of the open-science release; we redistribute only
inside the working repo (gitignored) and cite Ham et al. 2023
prominently in every downstream artefact.

## Schema

Each CSV is comma-delimited, **no header row**, two columns:

| Column | Type | Description |
|---|---|---|
| (0) | string | 378-nt sequence context, ACGT alphabet |
| (1) | float  | log-enrichment activity score, unbounded [-15, +15] sanity band |

## Fixed sequence layout

Verified across all 33,743 rows. Every sequence has the same internal
structure with the labeled target's PAM at the same position:

    pos:  0 .. 188              | 189 .. 208    | 209 | 210..211 | 212 .. 377
          \---- 189 nt 5' ------/ \--protospacer/ \-N-/ \---GG---/ \--166 nt 3'-/

So position 209 is the variable PAM nucleotide (N) and positions
210-211 are the conserved GG of the NGG SpCas9 PAM.

## Phase 3 extracted window

Our transformer trains on a **43-mer window** extracted at the
canonical PAM offset:

    seq[179:222]   →   10 nt 5' flank | 20 nt protospacer | 3 nt PAM | 10 nt 3' flank

Why 43 instead of the full 378: the long context contains many
off-target NGG sites (16-29 per row in our spot-check), so a
fixed-window crop forces the transformer to focus on the labeled
target. Matches the bacterial-CRISPR window crisprHAL uses for their
TevSpCas9 variant (3+20+14 = 37 nt) but with a few more flanking
nucleotides per side.

## Size and distribution

- **Total rows**: 33,743 (26,994 train + 6,749 test, pre-split by
  Ham et al.)
- **Sequence alphabet**: ACGT only, no Ns
- **Activity score**:
  - Train: range [-9.05, +6.21], mean -0.367
  - Test: range [-8.88, +6.01], mean -0.302
- **Score normalization (Phase 3 path)**: rank-normalize to [0, 1]
  within train and test independently. Lets our Phase-2 transformer
  (sigmoid output, MSE loss) drop in unchanged.

## Splits used by this project

The corpus is **already split** by Ham et al.: train and test are
distributed as separate files. For Phase 3 we:

- Keep the published `WT-SpCas9_testing_data.csv` as the test set
  (untouched, used only for the final eval).
- Carve a deterministic 10% of `WT-SpCas9_training_data.csv` as the
  validation split for early stopping during training, seed=42.

Whether the train and test sets in Ham et al. share targets or genes
in common is not directly stated in the paper, so we treat them as
nominally independent and report bootstrap CIs to capture residual
uncertainty.

## Known biases and limitations

1. **E. coli only.** All measurements come from a two-plasmid
   positive-selection assay in E. coli. *Bacillus subtilis* and other
   industrial bacteria are extrapolation; that's the Phase 4 stretch.
2. **SpCas9 / NGG only.** Single Cas9 variant, single PAM convention.
3. **Log-enrichment, not absolute cleavage rate.** The model predicts
   relative ranking of sgRNAs, not absolute cut efficiency.
4. **Different biology from Doench 2016.** Bacterial CRISPR is
   chromatin-free; the mammalian-trained Phase-2 model is, by
   construction, out-of-distribution here. Path A (transfer
   learning) is the experiment that tests how much transfers.
5. **Self-targeting toxicity is a known confounder** in bacterial
   CRISPR assays. Ham et al.'s positive-selection design addresses
   this for the high-quality TevSpCas9 subset (279 sgRNAs) but the
   WT-SpCas9 corpus uses a different assay where toxicity may still
   bleed into the activity score.

## Citation

If you use this corpus, cite:

> Ham, D. T., Browne, T. S., Banglorewala, P. N., Wilson, T. L.,
> Michael, R. K., Gloor, G. B., & Edgell, D. R. (2023). A generalizable
> Cas9/sgRNA prediction model using machine transfer learning with
> small high-quality datasets. *Nature Communications*, 14, 5514.
> https://doi.org/10.1038/s41467-023-41143-7

---

*Data card revision 1, 2026-05-16. Update when adding new bacterial
variants (TevSpCas9, eSpCas9, SaCas9) or organisms (Bacillus, etc.).*
