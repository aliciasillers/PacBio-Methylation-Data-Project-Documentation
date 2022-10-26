---
title: "ccsmeth in Ubuntu"
author: "Alicia Sillers"
date: "2022-10-10"
output: 
  html_document:
    keep_md: yes
---



# ccsmeth in Ubuntu

ccsmeth can be used as an alternative to Primrose. Both detect DNA methylation using PacBio circular consensus sequencing. They also were both trained using human genome data. However, Primrose has been officially tested on plant genome data and has shown to have similar results to bisulfite sequencing, while ccsmeth has not been officially tested on plant genome data. That said, the current documentation on ccsmeth is much more complete that that on Primrose, and almost the entire workflow is self-contained. Most of the documentation here comes from the official documentation, which is very comprehensive and can be found at https://github.com/PengNi/ccsmeth 

## Installation

If using FARM, replace step 1 with running the following code before step 2 code:

```bash
module load bio3
source activate pbio-2022-09-29
```


Step 1: conda     

conda can be installed on Ubuntu as part of anaconda or miniconda. Here we will be using miniconda, but the process for installing anaconda is similar. This example is for miniconda3 for Linux 64-bit with Python 3.8. For other options visit this website: https://docs.conda.io/en/latest/miniconda.html#linux-installers


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

Step 2: ccsmeth   

Within the following code, there are a few different options for installing ccsmeth. My first instinct was to use the conda method since we are using conda for many other commands anyway, but it is easy to run into problems with this method if you don't have all the necessary dependencies loaded. For example, the base python version used in FARM is 2.7, and ccsmeth installation requires 3.8. As a result, I recommend using the first method listed in the code, as I believe it is the most likely to run successfully under a variety of conditions. 

```bash
conda create -n ccsmethenv python=3.8
# activate
conda activate ccsmethenv

# install ccsmeth after activating ccsmethenv
# install ccsmeth from github (latest version)
git clone https://github.com/PengNi/ccsmeth.git
cd ccsmeth
python setup.py install
# OR, install ccsmeth using python pip
pip install ccsmeth
# OR, install ccsmeth using conda
conda install ccsmeth -c bioconda
```

```bash
conda install bedtools -c bioconda
conda install pbccs pbmm2 samtools -c bioconda
```

## Call Hifi Reads with Kinetics

Be sure to have ccsmethenv activated for all of the following steps so the necessary packages will be available. If you ever have a problem activating the environment while using FARM, make sure you have pbio-2022-09-29 activated first. Also, if using FARM, the number of threads you request in the below code should be double the number of cores you request in your script header. 

```bash
ccsmeth call_hifi --subreads /path/to/subreads.bam \
  --threads 10 \ 
  --output /path/to/output.hifi.bam
```

## Align Reads
In order to align your reads, you will need to have a reference genome. If you do not already have the reference genome downloaded, you can look for a file online and download it with the following code.

```bash
wget http://url.org/ref.zip
```

```bash
unzip ref.zip #this step is only necessary if the data is downloaded in a zipped file
```

Inputs for the following code are the output file from the call_hifi step and your reference genome.

```bash
ccsmeth align_hifi \
  --hifireads /path/to/output.hifi.bam \
  --ref /path/to/genome.fa \
  --output /path/to/output.hifi.pbmm2.bam \
  --threads 10
```


## Call Modifications

Inputs for this step are the output from the align_hifi step, your reference genome, and a model, which you can find in the models directory within your ccsmeth directory.

```bash
CUDA_VISIBLE_DEVICES=0 ccsmeth call_mods \
  --input /path/to/output.hifi.pbmm2.bam \
  --ref /path/to/genome.fa \
  --model_file /path/to/ccsmeth/models/model_call_mods.ckpt \
  --output /path/to/output.hifi.pbmm2.call_mods \
  --threads 10 --threads_call 2 --model_type attbigru2s \
  --rm_per_readsite --mode align
```


## Call Modification Frequency

There are two modes for calling modification frequency: aggregate mode and count mode. Aggregate mode is usually more accurate but requires an aggregate model, while count mode does not. We do have access to the default aggregate model, but it is unclear whether this will increase or decrease accuracy for plant data given that the model is trained on human data. As a result, I will be trying both modes and will compare results.    

Here is the code for aggregate mode:

```bash
ccsmeth call_freqb \
  --input_bam /path/to/output.hifi.pbmm2.call_mods.modbam.bam \
  --ref /path/to/genome.fa \
  --output /path/to/output.hifi.pbmm2.call_mods.modbam.freq \
  --threads 10 --sort --bed \
  --call_mode aggregate \
  --aggre_model /path/to/ccsmeth/models/model_aggregate.ckpt
```

And here is the code for count mode:

```bash
ccsmeth call_freqb \
  --input_bam /path/to/output.hifi.pbmm2.call_mods.modbam.bam \
  --ref /path/to/genome.fa \
  --output /path/to/output.hifi.pbmm2.call_mods.modbam.freq \
  --threads 10 --sort --bed
```

The output for this step should be a .bed file containing location-specific information about methylation across the genome. Specifically, each column has the following information:   
1. chrom: the chromosome name   
2. pos: 0-based position of the targeted base in the chromosome   
3. pos_end: pos + 1   
4. strand: +/-, the aligned strand of the read to the reference
5. prob_0_sum: sum of the probabilities of the targeted base predicted as 0 (unmethylated) [DEPRECATED, ONLY meaningful in call_freqt module]    
6. prob_1_sum: sum of the probabilities of the targeted base predicted as 1 (methylated) [DEPRECATED, ONLY meaningful in call_freqt module]    
7. count_modified: number of reads in which the targeted base counted as modified   
8. count_unmodified: number of reads in which the targeted base counted as unmodified   
9. coverage: number of reads aligned to the targeted base   
10. modification_frequency: modification frequency    
11. k_mer: the kmer around the targeted base [DEPRECATED, ONLY meaningful in call_freqt module]   

This file can then be used for visualization and quantitative analyses. For an example of how to visualize methylation data, see the visualization section of the "PacBio Primrose in Ubuntu" markdown file and Figure 1 in the repository. 
