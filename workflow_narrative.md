# From Sequence to Variant

### Protein Language Models on CFDE Data: A Narrative Summary

**CFDE Training Center, Community-Sourced Training ModuleAuthor:** Saba Nafees, Ph.D.
**Companion document to the module notebook and repository README**

---

## Why This Module Exists

The Common Fund Data Ecosystem (CFDE) connects data generated across independent NIH Common Fund research programs, each maintained by its own Data Coordinating Center (DCC). Individually, these DCCs provide powerful, single-purpose resources. The harder and more scientifically interesting problem is connecting them: asking a question that only makes sense when data from two separate programs is combined. This module was built to demonstrate exactly that kind of connection, using two CFDE resources that have never previously been paired in CFDE training content, MoTrPAC and GTEx, bridged through a protein language model.

The module also addresses two gaps identified in CFDE's existing training catalog: a lack of multi-omic data integration workflows, and an almost complete absence of machine learning content applied to CFDE data. It is, at the time of writing, the only proposed community module focused on protein-level biology rather than genomics alone.

### The Scientific Question

MoTrPAC (the Molecular Transducers of Physical Activity Consortium) tells us which genes change expression in response to structured exercise training. MoTrPAC measures multiple molecular layers, including both transcriptomics (RNA expression) and proteomics (protein abundance); this module uses its transcriptomics data specifically, for reasons explained below, not because proteomics data isn't available. GTEx (the Genotype-Tissue Expression project) tells us which genetic variants regulate expression of those same genes across human tissues. Neither dataset alone can answer the question this module poses: is exercise-responsiveness encoded in protein sequence itself, and do the sequence positions a language model learns to rely on correspond to places where real, independently discovered genetic variation already exists? Answering this requires a third component, ESM2, a protein language model, to translate between sequence-level and variant-level evidence.

---

## Three Molecular Layers, One Connected Pipeline

The module moves through three genuinely distinct sources of evidence, and understanding how they relate to one another is the key to understanding the whole notebook.

**MoTrPAC (the label).** The module uses MoTrPAC's rat six-month endurance training dataset, specifically its transcriptomics (RNA expression) measurements in liver tissue, comparing eight weeks of trained animals against sedentary controls. A gene is called "exercise-responsive" if its expression changed significantly in either direction, up or down, not upregulation specifically. This produces the labels the rest of the pipeline learns to predict.

**ESM2 (the input).** ESM2 is a protein language model, trained the same way large language models for text are trained, but on amino acid sequences instead of sentences. During training it repeatedly masks a random amino acid and learns to predict it from context, across millions of real protein sequences. In doing so it implicitly learns which substitutions are typically tolerated at each position in a sequence, a signal correlated with structural and functional importance. This module fine-tunes a simple classifier on top of ESM2's embeddings, using MoTrPAC's expression-based labels as the target, to ask whether sequence alone predicts exercise-responsiveness. It is worth stating plainly: the classifier's input comes entirely from protein sequence, nothing from MoTrPAC's expression data is used as a feature. Only the label comes from MoTrPAC.

**GTEx (the independent check).** Once the classifier identifies which sequence positions it relies on most, the module asks whether those positions, once translated from rat to human coordinates and from protein to genomic coordinates, sit near places where GTEx has already found real regulatory genetic variants (eQTLs) affecting the same gene. This is a genuinely independent line of evidence: GTEx's eQTLs were discovered entirely through population genetics, with no knowledge of ESM2, MoTrPAC, or exercise biology at all.

> A gene the model highlights as important, that also happens to sit near a real regulatory variant, is a case where two unrelated kinds of evidence, sequence-learned and population-genetic, converge on the same genomic neighborhood. That convergence is the notebook's central contribution.
> 

---

## Retrieving and Integrating the Data

Working with real, live CFDE data surfaced a number of genuine, non-obvious complications, each of which required a real fix rather than a workaround. These are worth mentioning, both because they shaped design decisions in the final notebook and because they are broadly representative of what real bioinformatics data integration looks like.

MoTrPAC's differential analysis tables are not standalone downloadable files. They are bundled as lazy-loaded objects inside the source code of an R package on GitHub, requiring the notebook to download and extract from the package's tarball directly rather than fetching a single file. The differential analysis table itself is structured "timewise," with one row per gene per training duration (one, two, four, or eight weeks) rather than a single overall test, and gene identity is stored only as an Ensembl ID, requiring a second lookup table to recover a readable gene symbol.

GTEx's eQTL endpoint does not accept a plain gene symbol as a query parameter at all. It requires a precise, versioned GENCODE identifier, resolved through a separate reference lookup first, a two-step pattern that is not obviously documented and required real trial-and-error, including several rounds of failed requests, to discover. Because MoTrPAC is a rat study and GTEx is human-only data, every gene also had to be translated to its human ortholog before any GTEx query could succeed at all, using MoTrPAC's own rat-to-human gene mapping table.

UniProt's protein sequence coverage for rat is considerably sparser than for human. Restricting sequence retrieval to UniProt's manually curated, "reviewed" entries alone returned usable sequences for only 27 of 74 candidate genes. Adding a fallback to UniProt's automatically predicted, "unreviewed" entries recovered the majority of the remainder, bringing the usable gene set to 57 of 74, with the explicit caveat that just over half of the final embedded set now comes from lower-confidence, non-manually-verified sequence predictions.

### A Note on Statistical Threshold

The notebook defines "exercise-responsive" using nominal, rather than multiple-testing adjusted, p-values. This is a deliberate, named trade-off rather than an oversight. At the scale of this dataset, thousands of genes tested across four timepoints and two sexes simultaneously, adjusted p-values are appropriately conservative for a real research claim but leave too few genes, roughly 90 at this timepoint, to build a workable labeled dataset for a teaching module. Nominal p-values yield roughly 2,800 candidate genes instead. This trade-off is stated explicitly in the notebook, along with the downside: the resulting gene list almost certainly includes some false positives.

---

## What the Pipeline Found

Working in liver tissue, comparing eight weeks of endurance training against sedentary controls, the pipeline identified 75 exercise-responsive genes by transcriptomics. Of these, 49 had a known human ortholog, and 13 of those 49 had at least one statistically significant GTEx eQTL in liver tissue. Fifty-seven of the 74 candidate genes had a usable protein sequence and were embedded with ESM2.

| 75 | 49 | 13 | 57 |
| --- | --- | --- | --- |
| responsive genes (transcriptomics) | have a human ortholog | have significant GTEx liver eQTLs | embedded with ESM2 |

### Fine-Tuning ESM2

A logistic regression classifier trained on ESM2 embeddings, mainly to test what the protein language model learns, using MoTrPAC's transcriptomics-based labels as the target, achieved an AUROC of 0.607 on 15 held-out test genes, meaningfully above chance but a modest signal, appropriate to a sample size this small. A small neural network trained on the same embeddings scored 0.643, a small but plausibly meaningful edge, consistent with a larger training set, 42 genes rather than the smaller sample used in earlier development runs, giving added model complexity a little more room to help.

### Connecting Sequence Signatures Back to Variants

For a representative gene, CBR1 (Carbonyl Reductase 1, an enzyme involved in clearing reactive metabolic byproducts, a biologically relevant choice given exercise increases metabolic stress), the fourteen sequence positions ESM2 relied on most heavily were translated from rat to human coordinates through direct sequence alignment (all fourteen mapped cleanly, reflecting near-total conservation between the rat and human versions of this protein), then from protein to genomic coordinates using Ensembl's exon-aware mapping tools. None of CBR1's thirty-one known GTEx eQTLs landed on an exact coding-sequence position, the expected outcome, since eQTLs act primarily through non-coding regulatory sequence rather than the coding sequence ESM2 was trained on. The closest eQTL sat 1,583 base pairs away*. 

*A note on reproducibility: the notebook's aggregate cross-reference (below) also processes CBR1 as one of several genes checked, using a slightly different number of sampled positions per gene than the single-gene walkthrough above. This produces a closest-distance figure for CBR1 in the aggregate run, 1,847 base pairs, that differs modestly from the 1,583 base pairs reported here. Both numbers are genuine computed outputs; the difference reflects which specific subset of CBR1's top positions each code path happened to sample, not a data error. The number quoted above reflects the single-gene walkthrough specifically.

Rather than rely on a single gene's result, which risks either an artificially clean or artificially null impression, the same pipeline was run across every gene with both eQTL coverage and an ESM2 embedding. An earlier successful run found that four of seven qualifying genes, 57 percent, had at least one eQTL within a defensible five-kilobase proximity window of a model-highlighted position. This aggregate view, rather than any single gene's number, is the notebook's central empirical finding.

> An exact positional match between a model-highlighted position and a known eQTL would be suggestive but not proof of shared biological mechanism, since eQTLs are regulatory and ESM2's signal is coding-sequence based. Proximity, not exact overlap, is the meaningful signal this pipeline can offer, and it should be read as hypothesis-generating rather than as a validated biological claim about any single gene.
> 

---

## A Direct Check: Does RNA-Level Responsiveness Match Protein-Level Responsiveness?

Because the module defines exercise-responsiveness using transcriptomics but goes on to model protein sequence with ESM2, a natural and important question is whether the two molecular layers actually agree. A direct comparison, using MoTrPAC's proteomics data for the same tissue and timepoint, found that they agree only about a quarter of the time.

| 2,593 | 2,917 | 679 |
| --- | --- | --- |
| responsive by transcriptomics | responsive by proteomics | responsive by both |

Of the 2,593 genes called responsive at the RNA level, only 679, about 26 percent, were also called responsive at the protein level. This is a substantial disagreement, and it directly qualifies how the classifier's results should be interpreted: its label is drawn from the transcriptomic layer, while its input, protein sequence, corresponds more directly to the proteomic layer. A gene ESM2 successfully predicts as responsive is being validated against an RNA-level definition that may not reflect what is actually happening at the protein level for that same gene.

---

## Limitations, Stated Directly

- Gene set size is capped at the top 75 exercise-responsive genes by effect size, for runtime reasons, not the full significant set across all tissues and timepoints.
- Nominal, not adjusted, p-values define responsiveness, a deliberate trade-off to keep a workable gene set for a teaching-scale module.
- The classifier trains and tests on a still modest sample, 42 training genes and 15 test genes; results should be read as directional rather than statistically robust.
- Results are specific to one tissue and one timepoint, liver at eight weeks, and should not be generalized without re-running the pipeline on other tissues or durations.
- Transcriptomics and proteomics responsiveness calls disagree substantially, only about a quarter of genes flagged as responsive by one layer are also flagged by the other.
- More than half of the embedded protein sequences come from lower-confidence, automatically predicted UniProt entries rather than manually curated ones, reflecting sparse rat proteome annotation.
- The sequence-to-variant cross-reference for any single gene depends on live calls to Ensembl's public API, which experienced genuine service instability during development; results should be reconfirmed with a fresh run before being presented as current.

---

## Conclusion and Generalizability

This module demonstrates a complete, reproducible pipeline connecting two CFDE Data Coordinating Center resources through a protein language model, retrieving and integrating real MoTrPAC and GTEx data programmatically, fine-tuning ESM2 on a phenotype label, and cross-referencing the model's learned sequence signal against independently discovered genetic variation. None of this is specific to exercise biology. Any CFDE dataset that pairs a labeled phenotype outcome with a protein-coding gene set could be substituted for MoTrPAC, and the same embed, classify, and cross-reference pipeline would run unchanged. MoTrPAC and GTEx are this just module's example.

The module's honest, stated limitations, small sample sizes, a deliberate statistical trade-off, and a real disagreement between RNA-level and protein-level definitions of responsiveness, are not incidental. They are part of what makes this a useful teaching tool: a learner who works through this notebook comes away understanding not just how to build a sequence-to-phenotype pipeline, but how to probe the results deeper and ways of extending them to answer other relevant biological questions.

---

*Companion materials: the full annotated Jupyter notebook, README with glossary and data source documentation, and answer key are available in the project repository at github.com/CFDETrainingCenter/multi-omics-plms.*