# Transcript_analysis
This repository contains the RNA-seq transcript-level analysis workflow used in my MSc research project.   The workflow focuses on **isoform quantification**, **differential isoform usage (DIU)**, and **functional interpretation** — with a case study on the **ATM gene** in a patient with Common Variable Immunodeficiency (CVID).



---

## Overview

This pipeline performs:

1. **Quality Control (QC)** of raw RNA-seq reads  
2. **Trimming** of low-quality bases  
3. **Read Alignment** to the reference genome using STAR  
4. **Transcript Quantification**  

---

#Quality Control
fastqc *.fq.gz -o FastQC_out
multiqc FastQC_out -o MultiQC_out

#Trimming
trimmomatic PE \
  sample_R1.fq.gz sample_R2.fq.gz \
  sample_R1_paired.fq.gz sample_R1_unpaired.fq.gz \
  sample_R2_paired.fq.gz sample_R2_unpaired.fq.gz \
  ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 \
  SLIDINGWINDOW:4:20 MINLEN:36

#Quality Control
fastqc trimmed_reads
multiqc trimmed_reads

#Alignment with STAR

a)Build genome index
STAR --runThreadN 12 \
  --runMode genomeGenerate \
  --genomeDir path/to/index_dir \
  --genomeFastaFiles ../GRCh38.primary_assembly.genome.fa \
  --sjdbGTFfile ../gencode.v39.primary_assembly.annotation.gtf
  
b)Align reads
STAR --runThreadN 12 \
  --genomeDir path/to/index \
  --outSAMtype BAM SortedByCoordinate \
  --quantMode GeneCounts \
  --outFileNamePrefix sample_ \
  --readFilesCommand zcat \
  --readFilesIn path/to/input_R1_P.fq.gz path/to/input_R2_P.fq.gz
  
#Transcript Assembly and Quantification (StringTie)
stringtie sample_Aligned.sortedByCoord.out.bam \
  -p 8 -G /Ref_seq/gencode.v39.primary_assembly.annotation.gtf \
  -o sample.gtf -l sample

  #merge Transcripts across samples
  a)Create a merge list
  ls sample*.gtf > samplelist.txt
  
  b)merge GTF files
   stringtie --merge -p 8 \
  -G /Ref_seq/gencode.v39.primary_assembly.annotation.gtf \
  -o stringtie_merged.gtf \
  samplelist.txt

  #Quantify each sample using Merged annotation
stringtie /sample_Aligned.sortedByCoord.out.bam \
  -e -B -p 8 \
  -G stringtie_merged.gtf \
  -o sample_final.gtf

  #Feature counts
  featureCounts -T 8 -p -s 2 -t exon -g gene_id \
  -Homo sapiens.GRCh38.110.gtf \
  -o counts.txt \
  sample1_Aligned.sortedByCoord.out.bam \
  sample2_Aligned.sortedByCoord.out.bam

Downstream Analysis in R

  

