# multi-omics-plms
CFDE Multi-Omics and Protein Language Models
# From Sequence to Variant: Protein Language Models on CFDE Data

A [CFDE Training Center](https://www.orau.org/cfde-trainingcenter/training/e-learning.html) community-sourced training module connecting **MoTrPAC** (exercise multi-omics) and **GTEx** (expression quantitative trait loci) through the **ESM2** protein language model.

**Author:** Saba Nafees, Ph.D.
**Format:** Google Colab-compatible Jupyter notebook
**Level:** Intermediate — working Python familiarity, basic biology background, no prior deep learning experience required
**Estimated completion time:** ~2.5 hours

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/CFDETrainingCenter/multi-omics-plms/blob/main/From_Sequence_to_Variant_PLMs_on_CFDE_Data.ipynb)

---

## What this module does

This notebook walks learners through an end-to-end machine learning workflow connecting two CFDE Data Coordinating Center resources:

1. **MoTrPAC** (Molecular Transducers of Physical Activity Consortium) — identify which genes/proteins change expression across tissues in response to exercise
2. **GTEx** (Genotype-Tissue Expression) — retrieve expression quantitative trait loci (eQTLs), the regulatory genetic variants that control those same genes' expression
3. **ESM2** (a pre-trained protein language model) — embed protein sequences, fine-tune a classifier to predict exercise-responsiveness from sequence alone, then ask whether the sequence positions driving the model's predictions correspond to positions where GTEx has already found real genetic variation

**The core scientific question:** *Is exercise-responsiveness encoded in protein sequence and do the sequence signatures a language model learns correspond to positions where natural genetic variation exists?*

This workflow can be extended to any CFDE perturbation dataset with a labeled phenotype outcome.

## Module structure

| Part | Content | Runtime |
|---|---|---|
| 0 | CFDE / MoTrPAC / GTEx orientation + pre-module knowledge check | ~10 min |
| 1 | How protein language models encode sequence (concept only) | ~25 min |
| 2 | Retrieve & integrate MoTrPAC + GTEx data | ~30 min |
| 3 | Fine-tune ESM2 for exercise-responsiveness prediction | ~45 min |
| 4 | Cross-reference model-highlighted sequence positions against GTEx eQTLs | ~25 min |
| 5 | Wrap-up, post-module knowledge check, reflection prompt | ~5 min |

Each part includes inline concept checks, data/coding checkpoints, and commented code cells written for readers with different programming backgrounds.
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

All data retrieval happens live in the notebook and no data files are bundled in this repository.

## Example results (from a full run)

Using liver tissue, 8-week endurance-trained vs. sedentary comparison:

- 75 exercise-responsive genes identified in MoTrPAC (nominal p < 0.05)
- 49 / 75 genes had a known rat→human ortholog
- 13 / 49 orthologous genes had significant GTEx liver eQTLs
- 27 genes had usable rat protein sequences and were embedded with ESM2
- Logistic regression classifier: **AUROC 0.75** on held-out test genes
- Cross-reference example (gene *LIG1*): 0/10 GTEx eQTLs exactly matched a model-highlighted coding position; closest eQTL was 1,005 bp away

These numbers are illustrative of the workflow, not a validated biological finding and sample sizes at every stage are small by design, to keep the module runnable in a short amount of time without institutional compute.

## Known limitations

- **Gene set size:** capped at the top 75 exercise-responsive genes (by effect size) for runtime reasons; a full analysis would use all significant genes across all tissues/timepoints.
- **Significance threshold:** uses nominal (unadjusted) p-values rather than multiple-testing-corrected p-values, to yield a workable gene set for a teaching example. See inline notebook comments for the full rationale.
- **Small sample size:** the fine-tuning classifier trains on ~20 examples and tests on ~7 so the results are meant to show proof of concept and are not statistically robust.
- **Single tissue/timepoint:** results are specific to liver at the 8-week training timepoint and should not be generalized to other tissues without re-running the pipeline.
- **Part 4 cross-reference is capped at 15 positions per live run** for demo speed (Ensembl's REST API has real per-request latency); the notebook's saved outputs reflect a full uncapped run.

## Repository contents

```
.
├── From_Sequence_to_Variant_PLMs_on_CFDE_Data.ipynb   # the training module notebook
└── README.md                                           # this file
```

## Acknowledgments

Developed for the CFDE Training Center's community-sourced training module program. Built on open-access data from the MoTrPAC and GTEx Data Coordinating Centers.

## License

This notebook is released for reuse and adaptation by CFDE Training Center educators and the broader community. See individual data source licenses above for the underlying datasets.

## Contact

Saba Nafees, Ph.D. — for questions about this module, please open an issue in this repository.