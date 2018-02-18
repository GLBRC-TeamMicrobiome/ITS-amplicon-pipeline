# Proposed workflow for ITS amplicon analysis

## 1) raw reads quality check and prefiltering
--------------------------------------------
- FastQC
- Bowtie2 (removing Phix reads)
- python scripts

## 2) read assembly
----------------
- USEARCH/VSEARCH
- FastQC

## 3) Demultiplexing
------------------
- QIIME or USEARCH 

## 4) stripping primers and adapters
-----------------------------------
- USEARCH

## 5) stats results
------------------
- USEARCH/VSEARCH

## 6 quality filtering and trimming
----------------------------------
- USEARCH/VSEARCH
- FastQC

## 7 Clustering OTUs and Generating ESV (Exact Sequence Variants)
-----------------------------------------------------------------
- USEARCH (UPARSE and UNOISE3 algorithms)

## 8 taxonomy assignments
-------------------------
- CONSTAX (we developed this tool that generates a consensus taxonomy for ITS amplicon reads, please have a look https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-017-1952-x
It is completely integrated in the HPCC and people can run it in there

- Alternatively, RDP or SINTAX? I am happier with RDP and makes more sense to me since we have the RDP folks on campus

# Sub-folders
- python_scrips
- phix_index
- Reference_DB
