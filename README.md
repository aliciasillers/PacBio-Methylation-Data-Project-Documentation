# CCS Methylation Data Project Documentation

This repository is meant to be an unofficial set of documentation for everything related to obtaining DNA methylation data from PacBio circular consensus sequence data, particularly on the UC Davis FARM computing cluster. There are many steps involved in this process, all with their own documentation in different places, some of which is not very comprehensive. The purpose of this repository is to have all of the necessary information in one place with detailed instructions and solutions to problems. This documentation is split into three markdown files: one for using SLURM, a workflow manager used on the UC Davis FARM computing cluster; one for using PacBio Primrose as a method for obtaining methylation data, along with all of the other steps related to using Primrose; and one for using ccsmeth as an alternative method for obtaining methylation data.

All of the steps involved in this documentation are being used to obtain Strawberry methylation data for the Knapp (soon to be Feldmann) Lab at UC Davis. However, code provided in the documentation will be generalized for reproducibility. 

The prerequisite to using this documentation is to obtain PacBio HiFi circular consensus sequence data. While this sequencing method was not designed as a way to obtain methylation data, it results in unique kinetic signatures associated with CpG context DNA methylation. In other words, there is a brief delay in sequencing when a methylated CpG is encountered. Due to the repetitive nature of this sequencing technique, kinetic signatures from multiple passes can be analyzed to provide a probability of methylation. Both Primrose and ccsmeth have been shown to have similar accuracy to bisulfite sequencing when tested with human genome data. Primrose has also been shown to have similar accuracy with plant data, making it the more ideal option for obtaining Strawberry methylation data. Using either method is more cost effective than bisulfite sequencing, and DNA samples used in HiFi circular consensus sequencing are not corrupted as they are in bisulfite sequencing, making both Primrose and ccsmeth advantageous and accessible in comparison to use of bisulfite sequencing for obtaining DNA methylation data. 