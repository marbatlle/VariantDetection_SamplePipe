# Overview of the case study: SNVs Analysis

**Goal:** Create a pipeline for the variant analysis of Whole-Exome Sequencing data.

**Data:** Whole-exome sequencing data from two samples from the patient (data was modified to reduce the computational analysis time), including:
* Human Genome: Alignment on a consensus genome reference
* Intervals: Predefined genomic regions of interest
* Raw Data: Patient's sample data, generated by the sequencing machine
    * Tumor sample
    * Matched normal sample (healthy tissue) from epithelium



**Library protocol:** Agilent SureSelect V5 Human All Exons.

## 1. Quality Control
Performed by **FASTQ software**
* Input: FastQ files
* Output report: html with plots

Script: 

    $ fastqc {fastq path} --outdir=out/fastqc

Manual: http://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/

## 2. Alignment
### 2.1. Indexing of Reference Genome
Performed by **Bowtie software**
* Input: Reference genome Fasta FastQ file
* Output report: Different files are generated during the indexing with the same prefix as the fasta file (hg19_chr17.fa*)

Script:

    $ bwa index [path to Human_genome]/hg19_chr17.fa

### 2.2. Alignment
Performed by **Bowtie software**
* Input: Reference genome and fastq sample files
* Output: Alignment sam file

Script: 

    $ bwa mem -R '@RG\tID:OVCA\tSM:sample' \
    [path to Human_genome]/hg19_chr17.fa \
    [path to Raw_data]/WEx_sample_R1.fastq \
    [path to Raw_data]/WEx_sample_R2.fastq > [path to Alignment]/sample.sam

## 3. Refinement of Alignment
Performed by **Samtools software**
* Input: Alignment sam file
* Output: Alignment refinement bam file

Script: 

    $ samtools fixmate -O bam [path to Alignment]/sample.sam [path to Alignment]/sample_fixmate.bam
    $ samtools sort -O bam -o [path to Alignment]/sample_sorted.bam [path to Alignment]/sample_fixmate.bam
    $ samtools rmdup -S [path to Alignment]/sample_sorted.bam [path to Alignment]/sample_refined.bam
    $ samtools index [path to Alignment]/sample_refined.bam


## 4. Variant Calling
### 4.1. Sample Calling and filtering
Performed by **BCFtools software**
* Input: Alignment refinement bam file
* Output: Variant vcf file

Script: 

    $ bcftools mpileup -Ou -f [path to Human_genome]/hg19_chr17.fa [path to Alignment]/sample_refined.bam | bcftools call -vmO z -o [path to Calling]/sample_rawcalls.vcf.gz
    $ bcftools index [path to Calling]/sample_rawcalls.vcf.gz

### 4.1. Intersection of Variants
Performed by **BCFtools software**
* Input: vcf samples files
* Output: intersection readme

Script: 

    $ bcftools isec –i 'DP>10' [path to Calling]/Tumour_rawcalls.vcf.gz [path to Calling]/Normal_rawcalls.vcf.gz -p [path to Calling]/Intersection
    $ cat [path to Calling]/Intersection/README.txt

## 5. Visualitzation
Performed by **IGV: Genome Browser software**
* Input: BAM files from each sample and BED file (the intervals)

