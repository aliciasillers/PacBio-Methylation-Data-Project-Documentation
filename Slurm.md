---
title: "Slurm"
author: "Alicia Sillers"
date: "2022-10-10"
output:
  html_document: 
    keep_md: yes
---



# SLURM

## Background Info

SLURM is a workload manager for computer clusters, and it is used for the FARM computer cluster used by UC Davis College of Agriculture and Environmental Sciences and College of Biological Sciences. A more comprehensive documentation for using SLURM on FARM can be found at https://github.com/RILAB/lab-docs/wiki/Using-Farm     
Despite there already being documentation, this SLURM documentation exists in order to have all the documentation needed for this DNA methylation project in one place. The same goes for other files in the repository. In general, all the documentation is written for FARM users, but it also includes alternative code for those not using FARM, as there will be different processes for installation, etc. 

## How SLURM works

SLURM aids in users' interaction with computer clusters in order for computing resources to be used efficiently. On the FARM cluster, there is one head node and several compute nodes. Large computing tasks should not be run on the head node because everyone has to share this node. Instead, these tasks must be submitted to run on the compute nodes which are designated for this purpose. This is where SLURM comes in. SLURM commands are used to submit computing jobs in the form of scripts. SLURM then determines what resources you have requested for your job and assesses what resources are available so that the appropriate resources can be allocated on the available node(s). If resources are not available right away, SLURM puts your job in the queue to run once other jobs finish.

## Writing scripts to submit as jobs

To submit code to run, the code needs to be written in a batch script. Open a window to write a script using the following command, substituting any name for [script name]:

```bash
nano [script name].sh
# you also can change the file type if you want to code in a different language. for example, you could write [script name].py for python
```

At the top of the script will be a header, which contains information about the job. The following code has header sections that are yet to be filled in, with the exception of nodes, mail type, and time, which are filled in but can be changed. For output and error, provide a file name with type .out and .err respectively. These files will be created when the job is run. If you want your script to be written in a different language, such as python or R, you can change the top line and replace bash with the language you want to use. Then, below the header you can write your code in the language you've chosen.

```bash
#!/bin/bash
#
#SBATCH --job-name=
#SBATCH --ntasks= # Number of cores
#SBATCH --nodes=1 # Ensure that all cores are on one machine
#SBATCH --mem=G # Memory pool for all cores (see also --mem-per-cpu)
#SBATCH --partition= # Partition to submit to
#SBATCH --output= # File to which STDOUT will be written
#SBATCH --error= # File to which STDERR will be written
#SBATCH --mail-type=ALL # Type of email notification- BEGIN,END,FAIL,ALL
#SBATCH --mail-user= # Email to which notifications will be$
#SBATCH --time=1-00:00:00 # how much time your job will be given to run
```

One thing to keep in mind while filling out the header is that you want to make sure to request enough memory and time for your job to run, but you also don't want to request too much more than you need, or else it might take longer for SLURM to allocate the resources you need and start running your job. 

## Submitting a job

sbatch is used to submit a job to run on a specified partition of the cluster. Just specify the partition and the script you are submitting. For information about what partitions exist and what nodes they have, you can use the command sinfo. 

```bash
sbatch -p [partition] [script name].sh
```

## Checking on your job

The following command will provide a list of all jobs currently running or waiting to run on the cluster as well as information about these jobs, such as how long they have been running and what node(s) they are running on. The list of jobs is organized by partition, as each partition has its own queue. 

```bash
squeue
```

There will also be two files created when your job starts running: one ending in .out and one ending in .err. Printing these files will give you more information about your job, such as whether it was successfully completed and how long it ran or has been running (.out), or what kinds of problems were encountered while running the code (.err). 
