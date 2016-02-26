# FusionDirect

[![Build Status](https://travis-ci.org/OpenGene/FusionDirect.jl.svg?branch=master)](https://travis-ci.org/OpenGene/FusionDirect.jl)
[![Packagist](https://img.shields.io/packagist/l/doctrine/orm.svg)]()
[![FusionDirect](http://pkg.julialang.org/badges/FusionDirect_0.4.svg)](http://pkg.julialang.org/?pkg=FusionDirect)
[![FusionDirect](http://pkg.julialang.org/badges/FusionDirect_0.5.svg)](http://pkg.julialang.org/?pkg=FusionDirect)

detect gene fusion directly from fastq files

## Features
* no alignment needed, it just reads fastq files of pair sequencing
* output fusion pattern (gene and position), along with the reads support this fusion
* ultra sensitive, comparing to delly, factera or other tools
* output file is a standard fasta file, which can be used to verify fusions using blast or other tools
* very suitable for detecting fusions from cancer target sequencing data (exom seq or panel seq)

## Installation
```julia
# from Julia REPL
Pkg.add("FusionDirect")
```

## Use FusionDirect as a package
```julia
using FusionDirect

# the reference folder, which contains chr1.fa, chr2fa...
# download from http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/chromFa.tar.gz and gunzip it
ref = "/opt/ref/hg19chr"
# a gene list with their coordination intervals, see the example bed files in data folder
bed = Pkg.dir("FusionDirect") * "/data/test_panel.bed"
read1 = "R1.fq.gz"
read2 = "R2.fq.gz"
detect(ref, bed, read1, read2)
```

## Use FusionDirect as a standalone script from commandline
copy src/fusion.jl to anywhere you want, run
```shell
julia fusion.jl -f <REF> -b <BED> -l <READ1> -r <READ2> > output.fa
```

## Get the reference
Can be downloaded from http://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/chromFa.tar.gz   
You should run `gunzip chromFa.tar.gz` then pass the folder contains fa files to `-f <REF>`

## Prepare the bed
A bed file to give a gene list (chr, start, end, gene), it usually includes the gene panel of your target sequencing and other genes you have interest (like EML4). Here gives an example:
```
chr9    133588266   133763062   ABL1
chr14   105235686   105262088   AKT1
chr19   40736224    40791443    AKT2
chr2    29415640    30144432    ALK
chrX    66764465    66950461    AR
chr11   108093211   108239829   ATM
chr3    142168077   142297668   ATR
chr2    111876955   111926024   BCL2L11
chr7    140419127   140624564   BRAF
chr17   41196312    41277500    BRCA1
chr2    42396490    42559688    EML4
```

## Understand the output
* fasta: The output is a standard fasta, which can be directly used to double check these fusions with blast(http://blast.ncbi.nlm.nih.gov/Blast.cgi?PROGRAM=blastn&PAGE_TYPE=BlastSearch&LINK_LOC=blasthome)
* duplication number: the first nubmer after `>` is the number of duplicated reads (including this displaying read), so it is at least 1.
* fusion_site: The followed word can be `merged`, `read1`, `read2` or `crosspair`, which means the fusion is detected on merged sequence, read1, read2 or read1/read2 are not on same contig.
* conjunct_pos: the number after `fusion_site`, which means in which base the fusion happens. If `fusion_site` is `merged`, then the number is according to the merged sequence. If `fusion_site` is `crosspair`, then this value is set 0.
* fusion_genes: following `conjunct_pos`, the two fusion genes, intron/exon number and the global fusion coordinations are given. `+` or `-` means forward strand or reverse strand. Note that fusion is on double strand DNA, so both `+` and `-` can exist on same fusion.
* original_reads: original reads are given for read1/read2. See `/1` or `/2` in the tail of read name.
* merged_sequence: if the pair of reads can be merged automatically, the fusion detection is done on the merged sequence. In this case, merged sequence is given with `/merge` in the tail of its read name.
```
#Fusion:ALK-EML4 (total: 3, unique: 2)
>2_merged_120_ALK:intron:19|+chr2:29446598_EML4:exon:21|-chr2:42553364/1
AATTGAACCTGTGTATTTATCCTCCTTAAGCTAGATTTCCATCATACTTAGAAATACTAATAAAATGATTAAAGAAGGTGTGTCTTTAATTGAAGCATGATTTAAAGTAAATGCAAAGCTATGTCGTCCAATCAATGTCCTTACAATC
>2_merged_120_ALK:intron:19|+chr2:29446598_EML4:exon:21|-chr2:42553364/2
GCTGCAAACTAATCAGGAATCGATCGGATTGTAAGGACATTGATTGGACGACATAGCTTTGCATTTACTTAAAATCATGCTTCAATTAAAGACACACCTTCTTTAATCATTTTATTAGTATTTCTAAGTATGATGGAAATCTATCTTAA
>2_merged_120_ALK:intron:19|+chr2:29446598_EML4:exon:21|-chr2:42553364/merged
AATTGAACCTGTGTATTTATCCTCCTTAAGCTAGATTTCCATCATACTTAGAAATACTAATAAAATGATTAAAGAAGGTGTGTCTTTAATTGAAGCATGATTTAAAGTAAATGCAAAGCTATGTCGTCCAATCAATGTCCTTACAATCCGATCGATTCCTGATTAGTTTGCAGC
>1_merged_60_ALK:intron:19|+chr2:29446598_EML4:exon:21|-chr2:42553364/1
TAAAATGATTAAAGAAGGTGTGTCTTTAATTGAAGCATGATTTAAAGTAAATGCAAAGCTATGTCGTCCAATCAATGTCCTTACAATCCGATCGATTCCTGATTAGTTTGCAGCCATTTGGAATGTCCCCTTTAAATTTAGAAACAG
>1_merged_60_ALK:intron:19|+chr2:29446598_EML4:exon:21|-chr2:42553364/2
GTAAAAGTGGCTAGTTTGAATCAAGATGCACTTTCAAATACATTTGTACACAAGCACTATGATTATACTTCCTGTTTCTAAATTTAAAGGGGACATTCCAAATGGCTGCAAACTAATCAGGAATCGATCGGATTGTAAGGACATTGATT
>1_merged_60_ALK:intron:19|+chr2:29446598_EML4:exon:21|-chr2:42553364/merged
TAAAATGATTAAAGAAGGTGTGTCTTTAATTGAAGCATGATTTAAAGTAAATGCAAAGCTATGTCGTCCAATCAATGTCCTTACAATCCGATCGATTCCTGATTAGTTTGCAGCCATTTGGAATGTCCCCTTTAAATTTAGAAACAGGAAGTATAATCATAGTGCTTGTGTACAAATGTATTTGAAAGTGCATCTTGATTCAAACTAGCCACTTTTAC
```
