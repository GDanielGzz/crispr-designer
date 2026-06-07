# Data card — Doench 2016 CRISPR-Cas9 efficacy corpus

Hugging-Face-style data card for the training corpus used by the
CRISPR Designer baseline and (eventually) our transformer.

## Source

Doench, J. G., Fusi, N., Sullender, M., Hegde, M., Vaimberg, E. W.,
Donovan, K. F., Smith, I., Tothova, Z., Wilen, C., Orchard, R., Virgin,
H. W., Listgarten, J., & Root, D. E. (2016). *Optimized sgRNA design to
maximize activity and minimize off-target effects of CRISPR-Cas9.*
Nature Biotechnology, 34, 184–191.
DOI: [10.1038/nbt.3437](https://doi.org/10.1038/nbt.3437)

Distributed via Microsoft Research's Azimuth repository:
https://github.com/MicrosoftResearch/Azimuth

## License

The Azimuth repository is BSD-3-Clause (Copyright Microsoft Research,
2015). See `ml/data/raw/AZIMUTH_LICENSE.txt` for the full text. Doench
et al.'s underlying paper supplementary tables are CC-BY per Nature's
standard supplementary policy.

When redistributing, this project keeps the Microsoft Research copyright
notice in the data directory and credits Doench 2016 prominently in
the project README and model card.

## Schema

| Column                       | Type    | Description |
|---                           |---      |--- |
| `30mer`                      | string  | 30-nt sequence context. Layout: 4 nt 5' flank · 20 nt protospacer · 3 nt PAM (NGG) · 3 nt 3' flank. ACGT alphabet only. |
| `Target gene`                | string  | Gene symbol. 17 unique values. Used for leave-one-gene-out splits. |
| `Percent Peptide`            | float   | Position of the cut within the coding sequence as a percentile (0–100). |
| `Amino Acid Cut position`    | float   | Position of the cut within the protein, in residues. |
| `score_drug_gene_rank`       | float   | **Primary regression target.** Within-gene-within-drug rank of measured activity, normalized to [0, 1]. |
| `score_drug_gene_threshold`  | int (0/1) | Binary label: 1 if `score_drug_gene_rank` is above the within-gene-within-drug median. Used for AUC. |
| `drug`                       | string  | Selection condition. 4 levels: `nodrug`, `6TG_2ug/mL`, `PLX_2uM`, `AZD_200nM`. |
| `predictions`                | float   | Rule Set 2 model's predictions for that 30mer (as published with Azimuth). Useful as a sanity check on our baseline reproduction — see `docs/baseline_results.md`. |

## Size and distribution

- **Rows**: 5,310 (validated; matches the paper's reported corpus size)
- **Unique target genes**: 17 (CCDC101, CD13, CD15, CD28, CD33, CD43,
  CD45, CD5, CUL3, H2-K, HPRT1, MED12, NF1, NF2, TADA1, TADA2B, THUMPD3)
- **Drug conditions**: 4
- **30mer alphabet**: ACGT only, no Ns
- **PAM distribution at positions 25–27**: AGG, CGG, GGG, TGG present (all SpCas9-compatible NGG)
- **Activity target range**: 0.001–1.000 (mean ≈ 0.503; the rank
  normalization gives a near-uniform distribution by design)

The full validation report is reproduced by running:

```bash
python ml/scripts/validate_data.py
```

## Splits used by this project

We use **leave-genes-out** for the test split, the same protocol Doench
used to evaluate Rule Set 2. With 17 genes available, we hold out:

- 2 genes for the **test** split (≈12% of rows)
- 2 genes for the **val** split (≈12% of rows)
- 13 genes for the **train** split (≈76%)

Genes are partitioned deterministically with `seed=42` in
`ml/scripts/make_splits.py`. The exact genes assigned to each split are
recorded in `ml/data/splits/SPLIT_NOTES.md`.

Within-gene random splits inflate reported metrics because adjacent
gRNAs in the same gene share biological context (chromatin state,
expression level, off-target landscape). Holding out by gene is the
honest protocol; we use it.

## Known biases and limitations

1. **Mammalian only.** Doench's screens were on human and mouse cell
   lines. The model trained on this corpus is **out-of-distribution for
   bacterial, plant, or yeast genomes.** Phase 2 may add a bacterial
   dataset for *B. subtilis* support; until then, the v1 designer
   advertises human (GRCh38) only.
2. **SpCas9 / NGG only.** Other Cas variants (Cas12a, SaCas9) and
   non-NGG PAMs are not represented.
3. **Limited gene panel.** 17 genes; broad sequence diversity is
   present at the 30-mer level, but gene-level diversity is modest.
   This is exactly why leave-genes-out evaluation matters.
4. **Activity is a rank, not a rate.** Predictions calibrate to
   relative ranking within a gene/drug, not absolute cleavage rate. The
   model can rank gRNAs against each other reliably; it cannot tell you
   "this gRNA will cut X% of the time."
5. **No accounting for chromatin or expression.** The model sees
   sequence only. In situ activity depends on chromatin accessibility,
   expression, and other context the model is blind to.

## Citation

If you use this corpus, cite Doench 2016 (DOI 10.1038/nbt.3437) and
the Azimuth repository (Microsoft Research, 2015).

---

*Data card revision 1, 2026-05-15. Update when adding new data sources
or splits.*
