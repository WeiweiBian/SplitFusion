# SplitFusion - a fast pipeline for detection of gene fusion based on fusion-supporting split alignment.

Gene fusion is a hallmark of cancer. Many gene fusions are effective therapeutic targets such as BCR-ABL in chronic myeloid leukemia, EML4-ALK in lung cancer, and any of a number of partners-ROS1 in lung cancer. Accurate detection of gene fusion plays a pivotal role in precision medicine by matching the right drugs to the right patients.

Challenges in the diagnosis of gene fusions include poor sample quality, limited amount of available clinical specimens, and complicated gene rearrangements. The anchored multiplex PCR (AMP) is a clinically proven technology designed, in one purpose, for robust detection of gene fusions across clinical samples of different types and varied qualities, including RNA extracted from FFPE samples.

**SplitFusion** is a companion data pipeline for AMP, for the detection of gene fusion based on split alignments, i.e. reads crossing fusion breakpoints, with the ability to accurately infer in-frame or out-of-frame of fusion partners of a given fusion candidate. SplitFusion also outputs example breakpoint-supporting seqeunces in FASTA format, allowing for further investigations.

## Reference publication
[Zheng Z, et al. Anchored multiplex PCR for targeted next-generation sequencing. Nat Med. 2014](http://www.nature.com/nm/journal/v20/n12/full/nm.3729.html)

## How does SplitFusion work?  


The analysis consists of ## computational steps:

1. Retrive all alignments that have secondary alignments (the 'SA' tag in SAM format) from bam files generated by BWA MEM.

2. Remove alignments with low mapping quality (default 20).

3. ...

4. ...


Lastly, outputs a summary table and breakpoint-spanning reads.

## Installation

Required software, libraries and data dependency

- R
- samtools
- bedtools
- snpEff
- java

## Command line of Installation
>git clone git@github.com:Zheng-NGS-Lab/SplitFusion.git

>R CMD INSTALL SplitFusion

The dependency data (e.g. in 'data') should contain:

| Filename                   | Content                                                        |
|:--------------------------:|:--------------------------------------------------------------:|
| panel-name.target.genes    | Contains a list of targets (gene name, e.g. ALK, ROS1, etc.)   |
| fusion.gene-exon.filter.txt | Contains recurrent breakpoints identified as data accumulates, but are not of interest. |
| fusion.gene-exon.txt          | By default, SplitFusion only outputs fusions that are *in-frame* fusion of two different genes or when number of breakpoint-supporting reads exceed predefined threashold. This file contains known breakpoints that do not belong to the above two kinds, but are clinically relevant, e.g. "MET_exon13---MET_exon15" an exon-skipping event forms an important theraputic target. Many exon skipping/alternative splicing events are normal or of unknown clinical relevance and are thus not output by default. |
| fusion.partners.txt | Contains a list of known fusion partners of targets. |
| ENSEMBL.orientation.txt | Due to the lack of transcript orientation in snpEff annotation, so this file include two columns, Orientation (+ or –) and transcript ID (ENST*). |

The above files could be updated periodically as a backend supporting database that facilitates automatc filtering and outputing of fusion candidates.


## Input file
1.Sample information table:  Sample name (prefixed name in bam file), Cancer type or project name ( not used in script, just for user labeling ), Panel name (prefixed panel name in panel-name.target.genes), cpuBWA number

An example table (table separated) :


>AP7 Sample_ID Panel cpuBWA

>Lib009 LungFusion ITFTNA 2

>Lib010 LungFusion ITFTNA 2


2.Config file: you can set the path and parameters of depended tools in this file.

Example  table:
>SplitFusionPath= “/your path/SplitFusion”

>sampleInfo=”/your path/Sample information table file”

>runInfo=”/your path/Config file”

>bam_path=”/your bam file directory/”

>Panel_path=”/your directory of dependency data/”


>hgRef=”/your path/Homo_sapiens_assembly19.fasta”

>samtools=”/your path/samtools” Version: 1.9 (using htslib 1.9)

>bedtools=”/your path/bedtools” Version: v2.25.0

>java=”/your path/java” openjdk version "1.8.0_181", OpenJDK Runtime Environment (build 1.8.0_181-8u181-b13-1ubuntu0.16.04.1-b13), OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)

>snpEff=”/your path/snpEff.jar” Downloaded from http://snpeff.sourceforge.net/download_donate.html

>R=”/your path/R”

>bwa=”/your path/bwa” Version: bwa-0.7.17


>StrVarMinStartSite=2

>maxQueryGap=0

>minMapLength=25

>minExclusive=25

>maxOverlap=9

>minMQ=13


## Run

R command line:

>Library(SplitFusion)

>runSplitFusion(runInfo= inputfile2, output= output directory, sample.id=sample name)

## Output
An example brief output table:

| AP7         | GeneExon5'---GeneExon3'    | num_unique_reads | frame    | Gene_Exon_cDNA_5'_3'            |
|:-----------:|:--------------------------:|:----------------:|:--------:|:-------------------------------:|
| A01-P701    | KIF5B_exon15---RET_exon12  |                7 | in-frame | KIF5B exon15 c.1723 .NM_004521.---RET exon12 c.2138 .NM_020630. |
| A02-P702    | EML4_intronic---ALK_exon20 |                9 | _NA_     | EML4 intronic c.NA .NM_001145076.---ALK exon20 c.3171 .NM_004304. |
| A02-P702    | EML4_intronic---ALK_exon20 |               10 | _NA_     | EML4 intronic c.NA .NM_001145076.---ALK exon20 c.3173 .NM_004304. |
| A02-P702    | EML4_exon4---ALK_exon20    |               64 | in-frame | EML4 exon4 c.468 .NM_001145076.---ALK exon20 c.3171 .NM_004304. |

An example output fastq file for the KIF5B_exon15---RET_exon12 fusion of sample A01-P701 is:

 >CL100059760L2C005R002_288074
TTCCCACTTTGGATCCTCCTTTACATCATTATTTCCCACAGCAATTCCTATTTCTGCAAGGTCTTTTAGTAAAGATGC
 >CL100059760L2C008R017_227619
TTCCGAGGGAATTCCCACTTTGGATCCTCCTTTACATCATTATTTCCCACAGCAATTCCTATTTCTGCAAGGTCTTTT
 >CL100059760L2C005R026_187567
TTGACTGGAGTTCAGACGTGTGCTCTTCCGAAAGCCCTCCCCGGTGCGCATGTTGGCAGGCTCAGACAAGGCCCTGG
 >CL100059760L2C003R075_538109
TAGGAATTGCTGTGGGAAATAATGATGTAAAGGAGGATCCAAAGTGGGAATTCCCTCGGAAGAACTTGGTTCTTGGAA
 >CL100059760L2C006R047_14778
ATAGGAATTGCTGTGGGAAATAATGATGTAAAGGAGGATCCAAAGTGGGAATTCCCTCGGAAGAACTTGGTTCTTGGA
 >CL100059760L2C012R012_483791
GAATTGCTGTGGGAAATAATGATGTAAAGGAGGATCCAAAGTGGGAATTCCCTCGGAAGAACTTGGTTCTTGGAAAAA
 >CL100059760L2C013R006_484169
AATTGCTGTGGGAAATAATGATGTAAAGGAGGATCCAAAGTGGGAATTCCCTCGGAAGAACTTGGTTCTTGGAAAAAC
 >CL100059760L2C017R020_507274
ATAGGAATTGCTGTGGGAAATAATGATGTAAAGGAGGATCCAAAGTGGGAATTCCCTCGGAAGAACTTGGTTCTTGGA
 >CL100059760L2C017R091_315407
GAATTGCTGTGGGAAATAATGATGTAAAGGAGGATCCAAAGTGGGAATTCCCTCGGAAGAGCTTGGTTCTTGGAAAAA
 >CL100059760L2C003R015_252083
GGGAATTCCCACTTTGGATCCTCCTATGTTGGAATTCCCTCGGAAGAACTTGGTTCTTGGAAAAACTCTAAGATCGGA



## Visualization of fastq for the EML4_exon7---ALK_exon20

