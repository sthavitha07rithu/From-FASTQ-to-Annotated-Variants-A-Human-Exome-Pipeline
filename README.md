# From-FASTQ-to-Annotated-Variants-A-Human-Exome-Pipeline
End-to-end human exome variant calling and annotation pipeline starting from raw FASTQ reads through QC, alignment, variant calling, filtering, and functional annotation using BWA-MEM2, samtools, bcftools, and snpEff.


## Overview
This repository documents an end-to-end human exome sequencing analysis pipeline, starting from raw paired-end FASTQ files and ending with a functionally annotated VCF.  
The workflow follows standard best practices for short-read variant calling on GRCh38 and is implemented entirely on Ubuntu using widely adopted bioinformatics tools.

The goal of this project is to demonstrate practical, reproducible execution of a complete variant calling workflow, including quality control, alignment, variant calling, filtering, and annotation.

---

## Pipeline Overview

1. **Raw data acquisition**
   - Download paired-end FASTQ files from ENA (SRR098401)

2. **Quality control of raw reads**
   - Assess base quality, GC content, and adapter contamination

3. **Read trimming**
   - Remove low-quality bases and residual adapters

4. **Post-trimming QC**
   - Verify improvements after trimming

5. **Reference preparation**
   - GRCh38 indexing and FASTA indexing

6. **Read alignment**
   - Align trimmed reads to GRCh38 using BWA-MEM2

7. **Alignment processing & QC**
   - Sorting, indexing, and mapping statistics

8. **Variant calling**
   - Generate raw VCF using bcftools mpileup + call

9. **Variant filtering**
   - Filter low-quality and low-depth variants

10. **Variant annotation**
    - Functional annotation using snpEff (GRCh38.86)

---

## Tools Used

| Step | Tool |
|----|----|
| Data download | wget (ENA FTP) |
| QC | FastQC, MultiQC |
| Trimming | fastp |
| Alignment | BWA-MEM2 |
| BAM processing | samtools |
| Variant calling | bcftools |
| Variant annotation | snpEff |
| Compression & indexing | bgzip, tabix |

---

## Outputs

## Results Snapshot

- **Mapping rate:** ~99.92% reads mapped to GRCh38  
- **Properly paired reads:** ~99.21%  
- **Final annotated variants:** ~129,000 VCF entries after filtering

---

## Next Steps

- Convert annotated VCF to tabular format for downstream analysis
- Variant prioritization based on functional impact
- Visualization and statistical analysis in **R** or **Python**
- Integration with population frequency and clinical databases (e.g., gnomAD, ClinVar)

---

## Notes
This repository focuses on pipeline execution and reproducibility.  
Large intermediate files (SAM/BAM/FASTQ) are intentionally excluded from version control.
