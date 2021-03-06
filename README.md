
<H2>
  
### 0. Overview
  
DNA-seq：

0. create an environment for DNA

   install trimmomatic, bwa, samtools and snakemake
    
1. use trimming script

    $ vim sk_trim # to revise the script
    
    $ snakemake -j 8 -s sk_trim

### Main process
  1. Quality assessment
  2. Alignment against GRCh38
  3. Duplicate removal
  4. Variant calling (germline or somatic mode)
  5. Variant annotation


<H2>

### 1. Preparation

#### 0.1 install Miniconda3 in Linux
```
cd /tempwork173/qiyoyou/
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
source /home/qiyoyou/.bashrc
```

#### 0.2 create an environment
```
conda create -n py3.7 python=3.7
source activate py3.7
deactivate
```

#### 1.1 create an environment named 'DNA' and activate this environment
```
conda create -n DNA python=3.7
conda activate DNA
```

#### 1.2 install 'fastqc', 'multiqc', 'trimmomatic', 'bwa', 'samtools' and 'snakemake' in DNA enviroment
```
conda install -c bioconda fastqc
conda install -c bioconda multiqc
conda install -c bioconda trimmomatic
conda install -c bioconda bwa
conda install -c bioconda samtools
conda install -c bioconda snakemake
conda install -c bioconda htslib ## for bgzip and tabix
```

<H2>

### 2. Quality Check

#### 2.1 Make sure you activate the enviromment since we installed 'fastqc' within and change direction to the folder containing raw .fastq data
```
conda activate DNA
cd /tempwork173/qiyoyou/Liu_WES_201903/RawData/
```

#### 2.2 Quality check with 'fastqc'. Apply fastqc to all file with '.gz' pattern (It usually takes a lot of time if sequence data being large)
```
fastqc *.gz
```

#### 2.3 Quality check with 'multiqc'. Apply multiqc to the output (.html) in this folder (./). This process integrate all the seperate output .html files into one.
```
multiqc ./
```

<H2>

### 3. Quality trimming and adapter removal using TRIMMOMATIC (tentative)

[[link to TRIMMOMATIC website]](http://www.usadellab.org/cms/?page=trimmomatic)


<H2>

### 4. Alignment against the GRCh38 reference assembly

[[link to the search page of hg38]](https://console.cloud.google.com/storage/browser/genomics-public-data/resources/broad/hg38/v0?pli=1&prefix=Homo_sapiens_assembly38.fasta)

#### 4.1 Build a GRCh38 genome index (bwa)

Build a folder for storing the index and cd to this folder.
```
cd /tempwork173/qiyoyou/db/
mkdir grch38_bwa
cd grch38_bwa/
```

Download hg38 .fasta from the link and store it to this folder.
```
wget https://storage.cloud.google.com/genomics-public-data/resources/broad/hg38/v0/Homo_sapiens_assembly38.fasta?_ga=2.56607659.-1064644800.1560758967
```

Making bwa index

[[link to bwa manual]](http://bio-bwa.sourceforge.net/bwa.shtml)
```
bwa index Homo_sapiens_assembly38.fasta
```

#### 4.2 Use alignment script to perform align, SAM to BAM and stats

Remember to revise the script to adapt to your own filenames and path names via 'vim' command.
```
cd /tempwork173/qiyoyou/Liu_WES_201903/scripts/
vim sk_align
snakemake -j 8 -s sk_align   ## use 8 threads
```

<H2> 

### 5. Remove duplicate and mpileup


<H2> 

### 6. Variant calling (germline) with VarScan

[[link to VarScan manual]](http://varscan.sourceforge.net/using-varscan.html)

Here is a really flexible settings for those parameters.
(parameters:--min-coverage 20 --min-var-freq 0.2 --min-reads2 4 --min-avg-qual 20 --p-value 0.05)


### 7. Variant annotation

#### 7.1 Apply 'bgzip' and 'tabix' on .vcf file

```
cd /tempwork173/qiyoyou/Liu_WES_201903/calling/
parallel bgzip {} ::: *.vcf
parallel tabix -p vcf {} ::: *.vcf.gz
```

```
# or one by one
# For each VCF file:
bgzip Variants_sample_A.raw.vcf
tabix -p vcf Variants_sample_A.raw.vcf.gz
```
#### 7.2 Create an folder and download VCFtools 

[[VCFtools download link]](https://sourceforge.net/projects/vcftools/files/)

```
cd /tempwork173/qiyoyou/
mkdir tools
cd tools/
tar -xvf vcftools_0.1.13.tar.gz
```

#### 7.3 Go to the decompressed folder and merge all the .vcf.gz into one .vcf file

Merge seperate .vcf files into one via VCFtools:vcf-merge. After this procedure, 'Variants_all_samples.vcf' can be uploaded to [wANNOVAR](http://wannovar.wglab.org/) to perform functional annotation.

```
cd /tempwork173/qiyoyou/tools/vcftools_0.1.13/perl/
perl vcf-merge /tempwork173/qiyoyou/Liu_WES_201903/calling/temp/*.vcf.gz >| /tempwork173/qiyoyou/Liu_WES_201903/calling/temp/Variants_all_samples.vcf
```

#### 7.4 Create a combined .vcf file without header by using 'awk' (optional)

Remove the header lines containing '##'.

```
awk '! /\##/' Variants_all_samples.vcf > Variants_all_samples_no_header.vcf
```


### Supplementary

[[How to Download hg38/GRCh38 FASTA Human Reference Genome]](https://www.gungorbudak.com/blog/2018/05/16/how-to-download-hg38-grch38-fasta-human-reference-genome/)

### Reference

(Analysis workflow)

1. Lee CY, Chiu YC, Wang LB, et al: Common applications of next-generation sequencing technologies in genomic research. Translational Cancer Research 2013;2:33-45. http://tcr.amegroups.com/article/view/962/html

