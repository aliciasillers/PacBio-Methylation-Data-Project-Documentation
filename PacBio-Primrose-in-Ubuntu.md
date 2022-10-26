---
title: "PacBio Primrose Workflow in Ubuntu"
author: "Alicia Sillers"
date: "2022-10-10"
output: 
  html_document:
    keep_md: yes
---



# PacBio Primrose Workflow in Ubuntu

Official PacBio Primrose documentation: https://github.com/PacificBiosciences/primrose    
While the official Primrose documentation only covers use of Primrose itself, this documentation covers the whole workflow, starting with obtaining HiFi reads with kinetics from existing PacBio HiFi sequence data and ending with visualization of DNA methylation data along the genome. 

## Initial Installations

### If using FARM

Primrose, ccs, and pbmm2 are already installed in FARM under the module bio3. This module will need to be loaded and activated every time you open FARM using the following code.

```bash
module load bio3
source activate pbio-2022-09-29 
```

### If not using FARM

Step 1: conda

ccs and primrose can be installed using conda. conda can be installed on Ubuntu as part of anaconda or miniconda. Here we will be using miniconda, but the process for installing anaconda is similar. This example is for miniconda3 for Linux 64-bit with Python 3.8. For other options visit this website: https://docs.conda.io/en/latest/miniconda.html#linux-installers


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

Official ccs documentation: https://ccs.how/     

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

## Alignment

After obtaining the hifi reads with methylation tags, the next step is to align these reads to a reference genome using pbmm2. Minimap2 usually works better for aligning the strawberry genome, but it requires input files to be in FASTA format, and we will need to use .bam input files in order to retain the methylation data.    

Official PBMM2 documentation: https://github.com/PacificBiosciences/pbmm2

### Installation

If using FARM, you do not need to install pbmm2. Simply load the bio3 module and activate pbio-2022-09-29.     

If not using FARM, install PBMM2 using conda.

```bash
conda install -c bioconda pbmm2
```

### Get Reference Genome

Before you can get started using pbmm2, you will need to download the reference genome to which to align your data. For this, you can use the wget command with the url to the online download in order to download the files from the internet into your directory. The result may be a zipped file, in which case you can use the unzip command in order to access the files inside.

```bash
wget http://url.org/ref.zip
```

```bash
unzip ref.zip
```

### Run pbmm2

There are two steps to running pbmm2. The first is to index your reference genome. The second is to align your data with a --sort argument at the end. The --sort argument is necessary in order for the CpG tools step to work.

```bash
pbmm2 index ref.fa ref.mmi
pbmm2 align ref.mmi movie.5mc.hifi_reads.bam movie_aligned.bam --sort
```
Keep in mind that if a file is ever in a different directory than your code, you will need to specify the path to the file. This might be necessary for your reference genome. For example ref.fa might be replaced with /home/user/refdir/ref.fa. 

## PacBio CpG Tools

Official documentation: https://github.com/PacificBiosciences/pb-CpG-tools    

The purpose of PacBio CpG tools is to quantify CpG methylation in particular. Other DNA methylation contexts will be ignored. The output will have a row of data for each CpG site in the read.   

In order to start using PacBio CpG tools, there are some files you will need to obtain. You can use the code below to download these files, which will show up in a directory called pb-CpG-tools.

```bash
git clone https://github.com/PacificBiosciences/pb-CpG-tools.git
```

The first script to run will contain the following code, which will use the .yaml file to create an environment that will have all of the packages required to run the next code. Since the .yaml file is in a directory, you might need to include the path to the file. This goes for later steps as well.

```bash
# create conda environment
conda env create -f conda_env_cpg.yaml
  # or conda env create -f ./pb-CpG-tools/conda_env_cpg.yaml

# activate environment
conda activate cpg
```

The next code takes the .py file and your aligned .bam file as input and should produce .bed and .bg files. Keep in mind that if you close and reopen Ubuntu between running the environment code and the following code, you will need to activate the environment again using "conda activate cpg"

```bash
python aligned_bam_to_cpg_scores.py -b input.bam -f ref.fasta -o label -d /path/to/model

# label is a string which results in [label].bed/bw
#additional optional arguments detailed in official documentation
```
 
The resulting .bed files will have 9 columns of information detailing the likelihood that a CpG site is methylated at each location. Specifically, each column has the following information:    
1. reference name   
2. start coordinate   
3. end coordinate   
4. modification probability   
5. haplotype    
6. coverage   
7. estimated modified site count (extrapolated from model modification probability)   
8. estimated unmodified site count (extrapolated from model modification probability)   
9. discretized modification probability (calculated from estimated mod/unmod site counts)
 
## Visualization   

The .bed file(s) obtained from this process contain the final information this workflow is designed to produce. From here there are many avenues for analyzing the data. For this project, we will be using R to visualize density of methylation across the genome. Since we will no longer be working in Ubuntu at this point, FARM users will need to export their .bed files to their own machines. 

### Exporting data from FARM

To copy data from FARM to your machine, it will need to be pulled from your machine rather than pushed from FARM. Start by listing the directory contents and printing the path to the directory so you will be able to reference this information if you do not have it memorized.

```bash
ls
pwd
```

Next, disconnect from FARM by holding the control key and then pressing the D key. After doing this, Ubuntu should be in your machine's home directory rather than a FARM directory, but your command history from FARM should still be visible for you to reference. Then use the below command to activate your FARM SSH key, map the path to the file you are getting, and map the path to the directory to which you want the file exported. 

```bash
rsync -axz --progress -e "ssh -CY " user@farm.ucdavis.edu:/path/to/farm/directory/filename.bed /path/to/your/directory
```

### Opening your .bed file in R

The following code creates a function called LoadBED, which can be used to load your .bed file into R as a data frame. It is a modified version of code from this source: https://gist.github.com/zerodel/3a901cc5c63e5c2e2bc95c937862a468

```r
LoadBED <- function(path.to.bed) {
    if (require(data.table)) {
        this.bed <-
            data.table::fread(
                path.to.bed,
                header = F,
                sep = "\t",
                stringsAsFactors = F
            )
    } else {
        this.bed <-
            read.table(
                path.to.bed,
                header = F,
                sep = "\t",
                stringsAsFactors = F
            )
    }
    
    names.bed <-
        c(
            "chrom",
            "start",
            "end",
            "modprob",
            "haplo",
            "coverage",
            "modcount",
            "unmodcount",
            "dismodprob"
        )
    num.col.this.bed <- dim(this.bed)[2]
    names(this.bed) <- names.bed[1:num.col.this.bed]
    return(this.bed)
}
```

The LoadBED function will take the path to your .bed file as its argument. If using a Windows computer, the path to this file on your machine will be the same as it is in Ubuntu with the following prefix: //wsl.localhost/Ubuntu/. You will want to bind the output of the function to a name, which you will then be able to access as your data frame.

```r
S18 <- LoadBED("//wsl.localhost/Ubuntu/home/user/movie.combined.denovo.bed")
```
