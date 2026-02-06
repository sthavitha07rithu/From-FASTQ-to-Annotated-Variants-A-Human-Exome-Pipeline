# From FASTQ to Annotated Variants: A Human Exome Pipeline

## Project Summary

This repository presents a **complete, single-sample human exome variant-calling pipeline** executed from raw paired-end FASTQ files to a functionally annotated VCF on **GRCh38**.

The project is designed to demonstrate **end-to-end NGS Variant Calling workflow**, with emphasis on:

* sound **bioinformatics reasoning** behind each step
* correct handling of **large, noisy sequencing data**
* **reproducible command-line execution** on Ubuntu

This pipeline is intended for learning and is not a clinical or cohort-scale production workflow.

**Dataset:** SRR098401 (public human exome)
**Final output:** compressed, indexed, functionally annotated VCF

---

## Design Philosophy and Scope

Key design principles:

* **End-to-end completeness:** covers all major stages of an exome variant-calling workflow
* **Reasoned tool choice:** each tool addresses a specific biological or computational need
* **Reproducibility:** every step is explicitly runnable via command line on Ubuntu

Scope clarifications:

* Single-sample analysis (no joint genotyping)
* Hard filtering instead of machine-learning-based recalibration
* Functional annotation, not clinical interpretation

These constraints are explicit and deliberate, ensuring the workflow remains interpretable and reproducible for a first full pipeline.

---

## Pipeline Overview and Reasoning

### 1. Raw Data Acquisition

**Source:** ENA FTP (SRR098401) using `wget`

Direct FASTQ download from ENA avoids the overhead and instability of SRA Toolkit conversion. Starting from raw reads ensures that all downstream quality control and preprocessing decisions are based on the actual sequencing output rather than pre-curated inputs.

---

### 2. Raw Read Quality Control

**Tools:** FastQC, MultiQC

FastQC is used to evaluate base quality distributions, GC content, adapter contamination, and duplication levels. These metrics determine whether preprocessing is required and what type of artifacts are present.

MultiQC aggregates QC outputs into a single report, reinforcing scalable best practices and enabling consistent interpretation as pipelines grow beyond a single sample.

**Rationale:** QC establishes whether the fundamental assumptions of short-read alignment are satisfied before proceeding.

---

### 3. Read Trimming and Filtering

**Tool:** fastp

Trimming removes low-quality bases and residual adapter sequences that would otherwise inflate mismatches and spurious variant calls. `fastp` was selected because it provides adapter detection, quality filtering, and QC reporting in a single, actively maintained tool with sensible defaults.

**Rationale:** Preprocessing restores the statistical assumptions required by the aligner while minimizing toolchain complexity for a first-time end-to-end run.

---

### 4. Post-trimming Quality Control

QC is repeated after trimming to confirm that read quality has improved and that adapters have been effectively removed. This step provides evidence that preprocessing decisions were beneficial rather than destructive.

---

### 5. Reference Genome Preparation

**Reference:** GRCh38
**Tools:** BWA-MEM2, samtools

The GRCh38 reference was selected for compatibility with modern annotations and databases. Explicit indexing of the FASTA ensures deterministic behavior across systems and avoids implicit dependency resolution.

**Rationale:** Reference preparation is a computational prerequisite and a frequent source of silent errors if handled implicitly.

---

### 6. Read Alignment

**Tool:** BWA-MEM2

BWA-MEM2 aligns trimmed reads to the reference genome using a widely trusted algorithm for short-read human data. It offers performance improvements over BWA-MEM while maintaining algorithmic equivalence and downstream compatibility.

**Rationale:** Accurate alignment is foundational; alignment errors directly propagate into false variant calls.

---

### 7. Alignment Processing and QC

**Tool:** samtools

Aligned reads are sorted and indexed to enable random access. `samtools flagstat` is used to assess mapping rate, pairing consistency, and overall alignment sanity.

Observed mapping statistics (~99.9% mapped, ~99.2% properly paired) are consistent with expectations for high-quality human exome data and serve as biological validation of upstream steps.

---

### 8. Variant Calling

**Tool:** bcftools (mpileup + call)

Variant calling is performed using a pileup-based approach to retain transparency in how evidence accumulates at each genomic position. This avoids black-box abstraction and exposes core concepts such as read depth and genotype likelihoods.

**Rationale:** For a single-sample educational pipeline, interpretability and control are prioritized over cohort-optimized callers.

---

### 9. Variant Filtering

**Method:** Hard filters on QUAL and DP

Raw variant calls include sequencing and alignment artifacts. Hard filtering applies explicit minimum confidence thresholds, producing a set of variants suitable for downstream functional interpretation.

**Rationale:** VQSR is not applicable in a single-sample context; hard filters are transparent, reproducible, and interpretable.

---

### 10. Variant Annotation

**Tool:** snpEff (GRCh38.86)

Annotation assigns predicted functional consequences to variants, translating genomic coordinates into gene-level biological meaning. snpEff was chosen for its offline execution, deterministic behavior, and compatibility with GRCh38.

**Rationale:** Annotation converts a technical artifact (VCF) into a biologically interpretable dataset.

---

### 11. Compression and Indexing

**Tools:** bgzip, tabix

The final annotated VCF is compressed and indexed to enable efficient storage and region-based querying, ensuring compatibility with downstream tools.

---

## Handling Large and Messy Data

NGS data are inherently large and noisy. This repository adopts the following practices:

* Large intermediate files (FASTQ, SAM, BAM) are **excluded from version control**.
* Only lightweight, information-dense artifacts (reports, logs, VCFs) are tracked.
* QC reports are preserved to justify preprocessing and alignment decisions.
* Each step produces inspectable outputs, allowing failures to be diagnosed rather than obscured.

This approach reflects real-world bioinformatics workflows, where data scale and disk usage must be managed explicitly.

---

## What Went Wrong and How It Was Resolved

Several issues were encountered during pipeline execution:

* **Slow and unreliable downloads via SRA Toolkit:** Resolved by switching to direct FASTQ downloads from ENA FTP.
* **Alignment failures due to limited RAM:** Mitigated by controlling thread usage and optimizing resource allocation.
* **Variant annotation errors:** Resolved by upgrading Java and explicitly setting heap memory for snpEff.
* **Missing compression/indexing utilities:** Addressed by installing `htslib` via Bioconda.

Documenting these issues reflects real-world pipeline development, where debugging and environment management are integral to reproducibility.

---

## Outputs and Results Snapshot

Key results from this run:

* **Mapping rate:** ~99.92% reads aligned to GRCh38
* **Properly paired reads:** ~99.21%
* **Final variants:** ~129,000 annotated variants after filtering

These metrics are consistent with expectations for a high-quality human exome sample and serve as sanity checks validating upstream QC, trimming, and alignment decisions.

---

## Why Not GATK?

This pipeline deliberately uses **bcftools** instead of the GATK Best Practices workflow. This choice is methodological because GATK is optimized for **production-scale, cohort-based analyses**, where:

* joint genotyping is performed across many samples
* Variant Quality Score Recalibration (VQSR) can be trained
* extensive metadata and is available

In contrast, this project is:

* **single-sample**
* intended for **learning and transparency**
* focused on understanding how evidence accumulates at each genomic position

Using bcftools allows:

* explicit inspection of pileup-based evidence
* deterministic behavior with minimal configuration
* reduced computational and conceptual overhead for a first pipeline


## Next Steps

* Convert annotated VCF to tabular formats (TSV/CSV) for downstream analysis
* Visualization and statistical analysis in R or Python
* Variant prioritization based on predicted functional impact
* Integration with population frequency and clinical databases (e.g., gnomAD, ClinVar)

---

## Notes

This repository prioritizes **execution correctness, reasoning transparency, and reproducibility**. It is intended as a demonstrable, inspectable example of how raw sequencing data are transformed into biologically meaningful variants through a reasoned bioinformatics workflow.


---

