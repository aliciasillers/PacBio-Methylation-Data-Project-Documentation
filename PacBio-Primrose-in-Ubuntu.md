---
title: "PacBio Primrose in Ubuntu"
author: "Alicia Sillers"
date: "2022-10-10"
output: 
  html_document:
    keep_md: yes
---



# PacBio Primrose in Ubuntu

Official PacBio Primrose documentation: https://github.com/PacificBiosciences/primrose 

## Installation

### If using FARM

Primrose, ccs, and minimap2 are already installed in FARM under the module bio3. This module will need to be loaded and activated every time you open FARM using the following code.

```bash
module load bio3
source activate pbio-2022-09-29 
```

### If not using FARM
Step 1: conda

ccs and primrose are installed using conda. conda can be installed on Ubuntu as part of anaconda or miniconda. Here we will be using miniconda, but the process for installing anaconda is similar. This example is for miniconda3 for Linux 64-bit with Python 3.8. For other options visit this website: https://docs.conda.io/en/latest/miniconda.html#linux-installers


```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-py38_4.12.0-Linux-x86_64.sh

chmod -v +x Miniconda*.sh

echo "3190da6626f86eee8abf1b2fd7a5af492994eb2667357ee4243975cdbb175d7a *Miniconda3-py38_4.12.0-Linux-x86_64.sh" | shasum --check

./Miniconda3-py38_4.12.0-Linux-x86_64.sh
```

After installing, you will need to activate miniconda using the following code. While installation only needs to happen once, this activation needs to happen every time you load Ubuntu.


```bash
source ~/.bashrc 
```

Step 2: ccs & primrose


```bash
conda install -c bioconda pbccs
conda install -c bioconda primrose
```

## Running ccs

Official ccs documentation: ccs.how     

On large subread data sets, ccs takes a long time to run. As a result, it is helpful to run it in chunks, which you can run in parallel. The first step towards this is to index the data with the following code. In the following ccs and primrose code, "movie" is used as an example name for some data. 


```bash
pbindex movie.subreads.bam
```

After the data is indexed, you can run ccs with the following command. You can decide how many pieces you want to split the data into based on how much there is and how fast you need ccs to run. In the following example, it is split into 10 chunks, as indicated at the end of the command. After running this command, you would run the same thing 9 more times while replacing "1" with each subsequent number until you reach "10." If running on FARM, requesting more cores for your job can also help speed things up. 


```bash
ccs movie.subreads.bam movie.hifi_reads.1.bam --hifi-kinetics --chunk 1/10
```

The resulting hifi_reads.bam files can then be used as input for Primrose. 

## Running Primrose

The actual running of Primrose only requires one line of code

```bash
primrose movie.hifi_reads.bam movie.5mc.hifi_reads.bam
```

Primrose will assess probability of CpG context DNA methylation and apply SAM tags to the data that indicate methylation. When viewing the output file with samtools view, these tags will show up towards the end of the printed data. The numbers after the MM tag represent the number of unmethylated cytosines between each methylated cytosine. The numbers after the ML tag represent the probability that each of these methylated cytosines are truly methylated. This data will be used for later steps in the workflow. Find more information about the SAM tags here: https://samtools.github.io/hts-specs/SAMtags.pdf 

## PBMM2 or Minimap2

After obtaining the hifi reads with methylation tags, the next step is to align these reads to a reference genome. Typically PBMM2 is used for this, but we will use minimap2, which works better for the strawberry genome.     

Official minimap2 documentation: https://github.com/lh3/minimap2 

### for FARM


```bash
module load minimap2/2.24
source activate minimap2-2.24

#Call minimap2, specify reference genome file, output file, and type of sequence data
minimap2 -a ref.fa -t 4 -o movie_aligned.bam -x map-hifi

conda deactivate

module load samtools/1.15.1

#Get alignments with 'Primary' tag and over 1500 base pairs in length
samtools view -h movie_sorted.bam | awk 'substr($0,1,1) == "@" || $2 < 2048 && length($10) > 1500' | samtools view -bS - > movie_Primary1500.bam

# Download, use msamtools to get reads aligned 80% of their length
#samtools sort Mojo_Primary1500_msam.bam -o Mojo_Primary1500_msam_resort.bam
#samtools index Mojo_Primary1500_msam_resort.bam
```

### not for FARM


```bash
git clone https://github.com/lh3/minimap2
cd minimap2 && make
```

```bash
conda install -c bioconda samtools
```

## PacBio CpG Tools

Official documentation: https://github.com/PacificBiosciences/pb-CpG-tools


```bash
# create conda environment
$ conda env create -f conda_env_cpg.yaml

# activate environment
$ conda activate cpg
```


```bash
python aligned_bam_to_cpg_scores.py -b input.bam -f ref.fasta -o label [options] # label is a string which results in [label].bed/bw
#additional optional arguments detailed in official documentation
```

## Visualization   
