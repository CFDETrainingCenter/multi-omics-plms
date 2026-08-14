# From Sequence to Variant: Protein Language Models on CFDE Data

A [CFDE Training Center](https://www.orau.org/cfde-trainingcenter/training/e-learning.html) community-sourced training module connecting **MoTrPAC** (exercise multi-omics) and **GTEx** (expression quantitative trait loci) through the **ESM2** protein language model.

**Author:** Saba Nafees, Ph.D.
**Format:** Google Colab-compatible Jupyter notebook
**Level:** Intermediate — working Python familiarity, basic biology background, no prior deep learning experience required
**Estimated completion time:** ~2.5 hours

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/CFDETrainingCenter/multi-omics-plms/blob/main/From_Sequence_to_Variant_PLMs_on_CFDE_Data_UPDATED.ipynb)

**Slides:** [View presentation](https://docs.google.com/presentation/d/1UsSOS1zQ0F5G_REfiG-qufJh5_lTXjeBKdd2-oLfsb0/edit?usp=sharing)

---

## What this module does

This notebook walks learners through an end-to-end machine learning workflow connecting two CFDE Data Coordinating Center resources:

1. **MoTrPAC** (Molecular Transducers of Physical Activity Consortium) — identify which genes/proteins change expression across tissues in response to exercise
2. **GTEx** (Genotype-Tissue Expression) — retrieve expression quantitative trait loci (eQTLs), the regulatory genetic variants that control those same genes' expression
3. **ESM2** (a pre-trained protein language model) — embed protein sequences, fine-tune a classifier to predict exercise-responsiveness from sequence alone, then ask whether the sequence positions driving the model's predictions correspond to positions where GTEx has already found real genetic variation

**The core scientific question:** *Is exercise-responsiveness encoded in protein sequence — and do the sequence signatures a language model learns correspond to positions where natural genetic variation exists?*

This workflow is not exercise-specific — it generalizes to any CFDE perturbation dataset with a labeled phenotype outcome.

## Module structure

| Part | Content | Runtime |
|---|---|---|
| 0 | CFDE / MoTrPAC / GTEx orientation + pre-module knowledge check | ~10 min |
| 1 | How protein language models encode sequence (concept only) | ~25 min |
| 2 | Retrieve & integrate MoTrPAC + GTEx data | ~30 min |
| 3 | Fine-tune ESM2 for exercise-responsiveness prediction | ~45 min |
| 4 | Cross-reference model-highlighted sequence positions against GTEx eQTLs | ~25 min |
| 5 | Wrap-up, post-module knowledge check, reflection prompt | ~5 min |

Each part includes inline concept checks, data/coding checkpoints, and heavily commented code cells written for readers without a heavy programming background.

## Getting started

**Option 1 — Google Colab (recommended, no setup required):**
Click the "Open in Colab" badge above, or open `From_Sequence_to_Variant_PLMs_on_CFDE_Data.ipynb` directly in [Google Colab](https://colab.research.google.com/). The first code cell installs all required packages automatically.

**Option 2 — Local Jupyter:**
```bash
git clone https://github.com/CFDETrainingCenter/multi-omics-plms.git
cd multi-omics-plms
pip install pyreadr requests biopython fair-esm torch scikit-learn matplotlib pandas numpy jupyter
jupyter notebook From_Sequence_to_Variant_PLMs_on_CFDE_Data.ipynb
```

No GPU or institutional compute cluster is required — ESM2's smallest checkpoint (`esm2_t6_8M_UR50D`) runs on a standard CPU in a few minutes for the gene set used in this module.

## Data sources

| Dataset | Access | License |
|---|---|---|
| [MoTrPAC](https://motrpac-data.org/data-download) rat 6-month endurance training data | Public, via [GitHub package source](https://github.com/MoTrPAC/MotrpacRatTraining6moData), no registration | CC BY 4.0 |
| [GTEx](https://gtexportal.org/) v8 (API v2) | Public REST API, no login | See [GTEx data use policy](https://gtexportal.org/home/citations) |
| [UniProt](https://www.uniprot.org/) | Public REST API | Public domain / CC0 |
| [Ensembl](https://rest.ensembl.org/) REST API | Public, no login | Public domain |
| [ESM2](https://github.com/facebookresearch/esm) (`esm2_t6_8M_UR50D`) | Open weights, downloaded automatically via `fair-esm` | MIT License |

All data retrieval happens live in the notebook — no data files are bundled in this repository.
## Example results (from a full run)

Using liver tissue, 8-week endurance-trained vs. sedentary comparison:

- 75 exercise-responsive genes identified in MoTrPAC (nominal p < 0.05)
- 49 / 75 genes had a known rat to human ortholog
- 13 / 49 orthologous genes had significant GTEx liver eQTLs
- 57 / 74 genes had usable rat protein sequences and were embedded with ESM2 (27 from UniProt's manually curated "reviewed" entries, 30 from lower confidence automatically predicted "unreviewed" entries, see the Glossary below)
- Logistic regression classifier: **AUROC 0.607** on held-out test genes (42 train / 15 test); a small neural network scored **AUROC 0.643** on the same split
- Cross-reference, aggregated across 7 genes with valid eQTL coverage and an ESM2 embedding: **4/7 genes (57%) had at least one eQTL within 5,000 bp of a model-highlighted position** (exact coding-sequence matches were 0/7, as expected, eQTLs are predominantly regulatory/non-coding). This aggregate reflects an earlier, smaller run (27 embedded genes); it has not been recomputed against the full 57-gene set.
- Addendum check comparing transcript-level and protein-level responsiveness calls for the same tissue and timepoint: 2,593 genes responsive by transcriptomics, 2,917 by proteomics, only 679 responsive by both (about 26% of transcript-responsive genes and 23% of protein-responsive genes overlap with the other layer)

These numbers are illustrative of the workflow, not a validated biological finding, sample sizes at every stage are small by design, to keep the module runnable in ~2.5 hours without institutional compute.

## Known limitations

- **Gene set size:** capped at the top 75 exercise-responsive genes (by effect size) for runtime reasons; a full analysis would use all significant genes across all tissues/timepoints.
- **Significance threshold:** uses nominal (unadjusted) p-values rather than multiple-testing-corrected p-values, to yield a workable gene set for a teaching example. See inline notebook comments for the full rationale.
- **Small sample size:** the fine-tuning classifier trains on ~20 examples and tests on ~7 — results are directional, not statistically robust.
- **Single tissue/timepoint:** results are specific to liver at the 8-week training timepoint and should not be generalized to other tissues without re-running the pipeline.
- **Part 4 cross-reference is capped at 10–15 positions per gene** for runtime reasons (each position requires its own Ensembl API request); this is a real constraint on the saved results, not just a live-demo simplification — see inline notebook comments for the exact caps used.
- **One gene had a much larger real eQTL count than initially shown:** CPNE1's results table initially showed exactly 500 eQTLs, matching the API's page-size cap used in that run. Following up with GTEx's pagination metadata confirmed the true count is **709**. Re-checking the closest-distance calculation against the full 709 (not just the first 500) still gives 94 bp — the number itself holds up. The remaining caveat is interpretive, not numerical: CPNE1 was tested against far more eQTLs than most other genes in the aggregate (709 vs., e.g., GP5's 6), so a close result there is statistically less surprising by chance alone than an equally close result from a gene with few eQTLs. The aggregate 4/7 statistic does not depend on this number and is unaffected.
- **Transcriptomics and proteomics responsiveness calls disagree substantially.** A companion check (last part of the notebook) compared MoTrPAC's transcript-level and protein-level "responsive" gene calls for the same tissue and timepoint: only ~23-26% of genes flagged as responsive by one layer are also flagged by the other. Since this module defines "exercise-responsive" using transcriptomics but models *protein* sequence with ESM2, this can be a limitation; the classifier's label and its input come from two only weakly-agreeing molecular layers.

## Glossary

Terms are defined here in plain language, in the order a reader is likely to encounter them. Terms used only once and defined inline in the notebook aren't repeated here.

CFDE (Common Fund Data Ecosystem) — an NIH initiative connecting data generated across separate NIH Common Fund research programs, each run by its own Data Coordinating Center (DCC), so researchers can ask questions that span multiple programs' data.

Data Coordinating Center (DCC) — the organization responsible for collecting, curating, and distributing data for one NIH Common Fund program. MoTrPAC and GTEx are each run by their own DCC.

Transcriptomics — measurement of RNA (gene expression) transcript/read levels, typically via RNA sequencing, a proxy of how much a gene is "turned on".

Proteomics — measurement of protein abundance, typically via mass spectrometry. Tells you how much of the actual protein product is present. RNA and protein levels don't always move together (see "Known limitations" above).

Differential analysis (DA) — a statistical comparison of measurements (expression, protein abundance, etc.) between two conditions (e.g. exercise-trained vs. sedentary), producing a significance value and an effect size for each gene.

Log fold-change (logFC) — the size and direction of a change between two conditions, on a logarithmic scale. Positive = increased in the trained group; negative = decreased. This notebook uses the absolute value of logFC to define "responsive," since both increases and decreases count as a response.

p-value — the probability of seeing a result at least this extreme purely by chance, if there were truly no real effect. Lower = more statistically confident the effect is real.

Adjusted p-value / FDR correction — a stricter version of the p-value that accounts for testing many genes at once (multiple-testing correction). Necessary for rigorous research claims, but often leaves very few "significant" results when applied to thousands of simultaneous tests on a small, teaching-scale dataset — see "Known limitations" above for why this notebook uses nominal (unadjusted) p-values instead.

Ortholog — a gene in one species that evolved from the same ancestral gene as a gene in another species, and typically performs the same or a very similar function. Used here to translate rat genes to their human equivalents, since GTEx only has human data.

Protein language model — a machine learning model trained on large numbers of protein sequences to predict masked amino acids from context, the same way text language models predict masked words. In doing so, it implicitly learns which amino acid substitutions are typically tolerated at each position.

ESM2 — the specific protein language model used in this notebook (Evolutionary Scale Modeling, version 2), developed by Meta AI. Available in several sizes; this notebook uses the smallest (esm2_t6_8M_UR50D, 8 million parameters) so it runs quickly on a standard CPU.

Embedding — a list of numbers produced by a model to represent something (a protein, a word, an image) in a way that captures its meaningful properties. Similar proteins tend to have similar embeddings. The specific number of values in an embedding (its "dimension") is fixed by the model's architecture — ESM2's smallest checkpoint produces 320-number embeddings; larger ESM2 checkpoints produce wider embeddings (up to 2,560 for the largest).

Fine-tuning — starting from an already-trained model (like ESM2, pre-trained on millions of unlabeled protein sequences) and adapting it, or training something on top of it, using a smaller labeled dataset for a specific task. Different from training a model from scratch.

Zero-shot prediction — using a pre-trained model's existing knowledge directly, without any task-specific labeled training data. Contrasted with fine-tuning, which does use labeled data.

Classifier — a model that assigns inputs to one of a fixed set of categories (here: "exercise-responsive" or "not"). This notebook uses two: logistic regression (simple, linear) and a small neural network (MLP, more flexible).

AUROC (Area Under the ROC Curve) — a standard score for how well a classifier ranks true positives above true negatives, across every possible decision threshold. 0.5 = no better than random guessing; 1.0 = perfect separation.

Checkpoint (in the notebook) — a point in the notebook where learners are asked to pause, interpret their own output, and answer a question before continuing. Not to be confused with a model checkpoint (a saved, trained version of a model — e.g. esm2_t6_8M_UR50D is one such checkpoint).

eQTL (expression quantitative trait locus) — a genomic position where a DNA variant is statistically associated with the expression level of a nearby (or sometimes distant) gene. One of the main tools researchers use to interpret what non-coding genetic variants might functionally do.

GENCODE ID — a standardized, versioned identifier for a gene (e.g. ENSG00000134243.11), used by GTEx and Ensembl. Required by GTEx's API for eQTL lookups — a plain gene name like "SORT1" is not sufficient.

Ensembl — a public genomics database providing gene, transcript, and protein structure information, including tools to convert between protein positions and genomic (chromosome) coordinates — used in Part 4 to map ESM2's sequence positions to real genomic locations.

Canonical transcript — the single, representative version of a gene's transcript chosen as its "main" or default form, when a gene has multiple alternative transcripts. Ensembl's canonical protein ID is what's used for coordinate mapping in Part 4.

Reviewed vs. unreviewed (UniProt) — "reviewed" (Swiss-Prot) entries in UniProt are manually curated and high-confidence; "unreviewed" (TrEMBL) entries are automatically predicted and lower-confidence. Rat proteome coverage in the reviewed tier is much sparser than human's, which is why this notebook's UniProt sequence retrieval doesn't succeed for every gene.

Sequence alignment — lining up two sequences (here, a rat and a human version of the same protein) to identify which positions correspond to each other, accounting for the fact that the two sequences may not be exactly the same length due to evolutionary insertions or deletions.

BLOSUM62 — a standard scoring table used during sequence alignment, indicating how "similar" different amino acid substitutions are (e.g. two chemically similar amino acids score higher than two very different ones).

Residue — a single amino acid unit within a protein sequence; "residue position 50" means the 50th amino acid in the sequence.

## Repository contents

```
.
├── From_Sequence_to_Variant_PLMs_on_CFDE_Data_UPDATED.ipynb   # the training module notebook
├── workflow_narrative.md                                       # narrative summary (markdown version)
├── workflow_narrative.pdf                                      # narrative summary (PDF version)
└── README.md                                                   # this file
```
## Acknowledgments

Developed for the CFDE Training Center's community-sourced training module program, with guidance from Allissa Dillman and Laurel Steinfield (CFDE Training Center). Built on open-access data from the MoTrPAC and GTEx Data Coordinating Centers.

## License

This notebook is released for reuse and adaptation by CFDE Training Center educators and the broader community. See individual data source licenses above for the underlying datasets.

## Contact

Saba Nafees, Ph.D. — for questions about this module, please open an issue in this repository.