---
title: "Trial Run with CNVer"
author: "Simon Renny-Byfield"
date: "`r Sys.Date()`"
output: rmarkdown::html_vignette
vignette: >
  %\VignetteIndexEntry{Vignette Title}
  %\VignetteEngine{knitr::rmarkdown}
  \usepackage[utf8]{inputenc}
---

This document tracks my first attempt at running [CNVer](http://compbio.cs.toronto.edu/CNVer/) on a test data set. CNVer is a program that estiamtes presence absence variants and copy number variants using mate pair/paired end Illumina sequence data. The mapping is done like this: 

```
#!/bin/bash
#SBATCH -D /home/sbyfield/CNVer
#SBATCH -o /home/sbyfield/CNVer/logs/out_log-%j.txt
#SBATCH -e /home/sbyfield/CNVer/logs/err_log-%j.txt
#SBATCH -J bowtie

echo "Starting Job: "
date

cmd="gunzip test_data/TIL01_3510_3807_3510_N_TIP521_4_R1.fastq.gz"
echo $cmd
eval $cmd

cmd="bowtie AGPv3 test_data/TIL01_3510_3807_3510_N_TIP521_4_R1.fastq -p 12 -S -v 3 -a -m 600 --best --strata | samtools view -b -S -o bams/R1.bam -"
echo $cmd
eval $cmd

cmd="gunzip test_data/TIL01_3510_3807_3510_N_TIP521_4_R2.fastq.gz"
echo $cmd
eval $cmd

cmd="bowtie AGPv3 test_data/TIL01_3510_3807_3510_N_TIP521_4_R2.fastq -p 12 -S -v 3 -a -m 600 --best --strata | samtools view -b -S -o bams/R2.bam -"
echo $cmd
eval $cmd

module load gcc/4.5 cnver

cmd="interleave_bam bams/R1.bam bams/R2.bam bams/interleaved.bam"
echo $cmd
eval $cmd

echo "Ending Job: "
date
```

Next It is important to estiamte the mean and SD of insert size in your mapping data so that CNVer can do a good job of identifying discordant mate pairs. This can be achived using a program distributed with CNVer called `find_mean_sd` and the command look likes this:
```
#!/bin/bash -l
#SBATCH -D /home/sbyfield/CNVer
#SBATCH -J MeanL
#SBATCH -o /home/sbyfield/CNVer/out-%j.txt
#SBATCH -e /home/sbyfield/CNVer/error-%j.txt
#SBATCH --mem=20000

module load gcc/4.5 cnver

echo "Starting Jog: "
date

cmd="find_mean_sd bams/interleaved.bam bam 50 1000 75"
echo $cmd
eval $cmd

echo "End Job: "
date
```
arguments are <mapping file> <format> <min length> <max length> <seq length>

