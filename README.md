# Viral-Track

Viral-Track is a R-based computational software based on **STAR** and **samtools** developped to detect and identify viruses from single-cell RNA-sequencing (scRNA-seq) raw data. This tool was tested on various scRNA-seq datasets derived from mouse and human infected samples as described in our paper *'Detecting and studying viral infection at the single-cell resolution using Viral-Track'*.  Viral-Track was tested on a CentOS 7 cluster and on an Ubuntu 18.0.4 workstation. 


Installation
-------------

Before running Viral-Track, several dependencies must be installed :

1 . The first step is to install [**R software**] (https://www.r-project.org/). Once this is done, several Bioconductor packages  have to be installed too. To do so start a R session and type :


```r
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install(version = "3.10") 
BiocManager::install(c("Biostrings", "ShortRead","doParallel","GenomicAlignments","Gviz","GenomicFeatures))
```


2 . **S**pliced **T**ranscript **A**lignment to **A** **R**eference (**STAR**) has to be installed. The full installation process is described on the STAR [Github](https://github.com/alexdobin/STAR). On Ubuntu this can be done directly by typing :

```batch
sudo apt-get install rna-star
```
3 . **Samtools** suite is also required. The installation is described extensively [here](http://www.htslib.org/download/). On Ubuntu this is done by simply typing :

```batch
sudo apt-get install samtools
```

4 . Lastly, the transcript assembler **StringTie** is needed. This installation process is described [here](https://ccb.jhu.edu/software/stringtie/). Don't forget to add StringTie to your shell's PATH directory.

Creation of the  Index and of the annotation file 
----------

The first step consists in creating a **STAR** index that include both host and virus reference genomes.
To do so first download the **ViruSite**  [genome reference database](http://www.virusite.org/index.php?nav=download). Host genome has also to be downloaded from the [ensembl website](https://www.ensembl.org/info/data/ftp/index.html). 

The  **STAR** index can now be build by typing :

```batch
mkdir /path/to/index
STAR --runThreadN N --runMode genomeGenerate --genomeDir /path/to/index --genomeFastaFiles /path/to/Virusite_file.fa  path/to/Host_genome_chromosome*.fa 
```

This can take some time and requires large amount of memory and storage space : please check that you have at least 32 GB of RAM and more than 100GB of avaible memory.

Lastly you need to create a small file that list all the viruses included in the index and their genome length. We provide an example in the Github that corresponds to the VirusSite dataset.


Detection of viruses in scRNA-seq data
---------------

We can now start the real analysis. Viral-Track relies on two different text files to run : a file containing the values of all parameters (parameter file) and one containing the path to the sequencing files to analyze (target file). An exemple of each file is provided in the Github. 


The parameter file consists in a list of rows where the name of each variable is followed by an equal symbol and then the value of the parameter.

The target file contains the list of the files paths. The files can be either .fastq or .fastq.gz files.

Before running any scanning analysis check the parameter file and make sure that you have set the correct values for :

1. The output directory (Output_directory variable) : If the directory does not exist it will be created.
2. The path to the STAR index (Index_genome).
3. The path to the virus annotation file (Viral_annotation_file).
4. The number of cores to use (N_thread) : please be carefull as STAR and samtools can comsumme large amount of memory ! Runnning Viral-Track with a too high number of thread can trigger massive memory swapping....

Once this is done you can launch the analysis using :

```batch
Rscript Viral_Track_scanning.R Path/to/Parameter_file.txt Path/to/Target_file.txt
```
If you want to launch it in the background use instead :

```batch
R CMD BATCH Viral_Track_scanning.R Path/to/Parameter_file.txt Path/to/Target_file.txt &
```

Once the analysis is over, the results can be checked by looking at the output directory. A pdf called QC_report.pdf is automatically generated and described the most important results :
The three first panels describe the general quality of the mapping (percentage of mapped reads, mean length of the mapped reads....). The next panels describe the quality of the mapping of each individual virus : three different scatter plots show the number of uniquely mapped reads, the complexity of the sequences, the percentage of genome mapped and the length of the longest mapped contig. By default only viruses with at least 50 uniquely mapped reads, a mean sequence complexity of 1.3 and 10% of the genome mapped is considered as being present in the sample.
An example of QC file can be found in the Github.

Transcriptome assembly
---------------
