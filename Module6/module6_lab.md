---
layout: tutorial_page
permalink: /HTSeq_2018_module6_lab
title: HTSeq Lab 6
header1: Workshop Pages for Students
header2: Informatics on High-Throughput Sequencing Data
image: /site_images/CBW_High-throughput_icon.jpg
home: https://bioinformaticsdotca.github.io/htseq_2018
---

# Genome Assembly for short and long reads

by [Jared Simpson](https://simpsonlab.github.io)

## Introduction

In this lab we will perform de novo genome assembly of a bacterial genome. You will be guided through the genome assembly starting with data quality control, through to building contigs and analysis of the results. At the end of the lab you will know:

1. How to perform basic quality checks on the input data
2. How to run a short read assembler on Illumina data
3. How to run a long read assembler on Pacific Biosciences or Oxford Nanopore data
4. How to improve the accuracy of a long read assembly using short reads
5. How to assess the quality of an assembly

## Data Sets

In this lab we will use a bacterial data set to demonstrate genome assembly. This data set consists of sequencing reads for Escherichia coli. E. coli is a great data set for getting started on genome assembly as it is a small genome (4.6 Mbp) with relatively few repeats, and has a high quality reference. We have provided Illumina, PacBio and Oxford Nanopore reads for this genome. The assemblies you will run later using `spades` and `canu` use a read set that is restricted to a one megabase region of the E. coli genome. This is to reduce the amount of compute time the assemblies take, so that they complete during this lab. Running a whole genome assembly uses the exact same set of commands and the results you will obtain are comparable to assembling the entire genome.

## Data Preparation

First, lets create and move to a directory that we'll use to work on our assemblies:

```
mkdir -p ~/workspace/HTseq/Module6
cd ~/workspace/HTseq/Module6
```

For convenience, we'll make symbolic links to the data sets that we'll work with. Run this command from the terminal, which will find all of the sequencing data sets we provided (using the `ls` command) and symlink those files into your current working directory.


```
ls ~/CourseData/HT_data/Module6/ecoli* | xargs -i ln -s {}
```

If you run `ls` you should now be able to see three files of sequencing data.

## Data Quality Control

In the lecture we heard about many factors that may result in a poor assembly: not having enough coverage, very high repeat content, very high heterozygosity, etc. In this section of the lab we will use the [sga preqc](https://academic.oup.com/bioinformatics/article/30/9/1228/237596/Exploring-genome-characteristics-and-sequence) program to explore our short read data set before starting the assembly. We have generated a report for two E. coli Illumina data sets - one at 15x coverage and one at 50x coverage. This will allow us to look at the effect of sequencing coverage on the results of our assembly. The preqc report for these two datasets, and a human genome and difficult-to-assemble fish dataset, can be found [here](https://github.com/bioinformaticsdotca/HTSeq_2017/blob/master/module6_data/preqc_report.pdf). We aren't generating the report as part of this exercise because it takes a few hours to run. If you'd like to see how it was generated you can find the commands [here](https://bioinformaticsdotca.github.io/HTSeq_2017_module6_lab_supplement).

Open the PDF report and try to interpret the results. Was the genome size estimated correctly? What differences do you notice between the 15x (blue) and the 50x (red) datasets? Which dataset do you expect to be easier to assemble (hint: preqc will perform a simulated genome assembly to estimate the contig sizes you might get).

## E. coli Genome Assembly with Short Reads

Now we'll assemble the E. coli 50x Illumina data using the [spades](http://bioinf.spbau.ru/spades) assembler. Parameterizing a short read assembly can be tricky and tuning the parameters (for example the size of the k-mer used) is often quite time consuming. Thankfully, spades will automatically select values for its parameters, making it particularly easy to use. You can start spades with this command (it will take a few minutes to run):

```
spades.py -o ecoli-illumina-50-spades/ -t 4 --12 ecoli.illumina.50x.fastq
```

After the assembly completes, let's move the results to a new directory that we'll use to keep track of all of our assemblies:

```
mkdir -p assemblies
cp ecoli-illumina-50-spades/contigs.fasta assemblies/ecoli.illumina.50x.spades-contigs.fasta
```

We can now start assessing the quality of our assembly. We typically measure the quality of an assembly using three factors:

- Contiguity: Long contigs are better than short contigs as long contigs give more information about the structure of the genome (for example, the order of genes)
- Completeness: Most of the genome should be assembled into contigs with few regions missing from the assembly (remember for this exercise we are only assembling one megabase of the genome, not the entire genome)
- Accuracy: The assembly should have few large-scale *misassemblies* and *consensus errors* (mismatches or insertions/deletions)

We'll use `abyss-fac.pl` to calculate how contiguous our spades assembly is. Typically there will be a lot of short "leftover" contigs consisting of repetitive or low-complexity sequence, or reads with a very high error rate that could not be assembled. We don't want to include these in our statistics so we'll only use contigs that are at least 500bp in length (protip: piping tabular data into `column -t` will format the output so the columns nicely line up):

```
abyss-fac.pl -t 500 assemblies/ecoli.illumina.50x.spades-contigs.fasta | column -t
```

The N50 statistic is the most commonly used measure of assembly contiguity. An N50 of x means that 50% of the assembly is represented in contigs x bp or longer. What is the N50 of the spades assembly? How many contigs were produced?

## E. coli Genome Assembly with Long Reads

Now, we'll use long sequencing reads to assemble the E. coli genome. Long sequencing reads are better at resolving repeats and typically give much more contiguous assemblies. Long reads have a much higher error rate than short reads though, so we need to use a different assembly strategy. In this tutorial, we'll use [canu](https://github.com/marbl/canu) to assemble the 25X PacBio dataset. The canu assembly of the pacbio data should take about 15 minutes on your computer (maybe take a break while it is running).  Run this command to generate the assembly:

```
canu gnuplotTested=true -p ecoli-pacbio-canu -d ecoli-pacbio-auto genomeSize=4.6m -pacbio-raw ecoli.pacbio.25x.fastq
```

When it completes, copy the Pacbio assembly to our results directory:

```
# Copy the pacbio assembly from canu's directory
cp ecoli-pacbio-auto/ecoli-pacbio-canu.contigs.fasta assemblies/ecoli.pacbio.25x.canu-contigs.fasta
```

Our data set also includes a higher-coverage Oxford Nanopore data set (50X). This assembly takes quite a bit longer to run so we won't run it now, you can just copy the results that the instructors generated earlier in the week:

```
# Copy the nanopore assembly that was provided by the instructors
cp ~/CourseData/HT_data/Module6/results/ecoli-nanopore-canu.contigs.fasta assemblies/ecoli.nanopore.50x.canu-contigs.fasta
```

The command used to generate the nanopore assembly can be found [here](https://bioinformaticsdotca.github.io/HTSeq_2017_module6_lab_supplement).

## Assessing the quality of your assemblies using a reference

The accuracy of the genome assembly is determined by how many misassemblies (large-scale rearrangements) and consensus errors (mismatches, insertions or deletions) the assembler makes. Calculating the accuracy of an assembly typically requires the use of a reference genome. We will use the [QUAST](http://quast.bioinf.spbau.ru/) software package to assess the accuracy of the assemblies.

Run QUAST on your three E. coli assemblies by running this command:

```
quast.py -R ~/CourseData/HT_data/Module6/references/ecoli_k12.fasta assemblies/*.fasta
```

Using the web browser for your instance, open the QUAST PDF report and try to determine which of the assemblies was a) the most complete b) the most contiguous and c) the most accurate.

## Improving the Accuracy of the Long Read Assemblies

As PacBio and Nanopore reads have a higher error rate than Illumina reads it is expected that the long read assemblies has errors in the consensus sequence (mostly insertions and deletions). We can increase the accuracy of our consensus sequence by using more coverage (up to around 100x), or by "polishing" the assembly (inferring a better consensus sequence) with high-accuracy Illumina reads. We'll use [Pilon](https://github.com/broadinstitute/pilon) to calculate a new consensus sequence for our PacBio assembly. First we have to map the Illumina reads to our assembly using bwa.

```
bwa index assemblies/ecoli.pacbio.25x.canu-contigs.fasta
bwa mem -t 4 -p assemblies/ecoli.pacbio.25x.canu-contigs.fasta ecoli.illumina.50x.fastq | samtools sort -T tmp -o ecoli.illumina.50x.mapped_to_pacbio_contigs.sorted.bam -
samtools index ecoli.illumina.50x.mapped_to_pacbio_contigs.sorted.bam
```

Now we can run pilon:

```
java -Xmx4G -jar /usr/local/pilon/pilon-1.22.jar --genome assemblies/ecoli.pacbio.25x.canu-contigs.fasta --frags ecoli.illumina.50x.mapped_to_pacbio_contigs.sorted.bam --output assemblies/ecoli.pacbio.25x.canu-contigs-pilon
```

Let's also polish the nanopore assembly:

```
bwa index assemblies/ecoli.nanopore.50x.canu-contigs.fasta
bwa mem -t 4 -p assemblies/ecoli.nanopore.50x.canu-contigs.fasta ecoli.illumina.50x.fastq | samtools sort -T tmp -o ecoli.illumina.50x.mapped_to_nanopore_contigs.sorted.bam -
samtools index ecoli.illumina.50x.mapped_to_nanopore_contigs.sorted.bam
java -Xmx4G -jar /usr/local/pilon/pilon-1.22.jar --genome assemblies/ecoli.nanopore.50x.canu-contigs.fasta --frags ecoli.illumina.50x.mapped_to_nanopore_contigs.sorted.bam --output assemblies/ecoli.nanopore.50x.canu-contigs-pilon
```

After running pilon on the PacBio and nanopore assemblies, re-run the Quast step to generate a new report including all five assemblies. Did pilon-polishing help the accuracy of the long read assemblies?
