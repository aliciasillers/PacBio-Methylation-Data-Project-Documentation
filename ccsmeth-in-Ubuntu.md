---
title: "ccsmeth in Ubuntu"
author: "Alicia Sillers"
date: "2022-10-10"
output: 
  html_document:
    keep_md: yes
---



# ccsmeth in Ubuntu

ccsmeth can be used as an alternative to Primrose. Both detect DNA methylation using PacBio circular consensus sequencing. They also were both trained using human genome data. However, Primrose has been officially tested on plant genome data and has shown to have similar results to bisulfite sequencing, while ccsmeth has not been officially tested on plant genome data. That said, the current documentation on ccsmeth is much more complete that that on Primrose. Most of the documentation here comes from the official documentation, which is very comprehensive and can be found at https://github.com/PengNi/ccsmeth 

## Installation

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


```bash
conda create -n ccsmethenv python=3.8
# activate
conda activate ccsmethenv
# deactivate this environment
conda deactivate

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


