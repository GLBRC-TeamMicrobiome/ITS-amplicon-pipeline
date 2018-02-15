# Proposed workflow for ITS amplicon analysis

*1) raw reads quality check and prefiltering;
--------------------------------------------
- FastQC
- Bowtie2 (removing Phix reads)
- python scripts

*2) Demultiplexing
- QIIME or USEARCH 

*3) read merging;
----------------
- USEARCH/VSEARCH
- FastQC

*4) quality filtering and trimming;
----------------------------------
- USEARCH
- FastQC

*5) Clustering OTUs and Generating ESV (Exact Sequence Variants);
-----------------------------------------------------------------
- USEARCH (UPARSE and UNOISE3 algorithms)

*6) taxonomy assignments;
-------------------------
- CONSTAX (we developed this tool that generates a consensus taxonomy for ITS amplicon reads, please have a look https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-017-1952-x
It is completely integrated in the HPCC and people can run it in there

- Alternatively, RDP or SINTAX? I am happier with RDP and makes more sense to me since we have the RDP folks on campus.

# Folders
- python_scrips
- phix_index
- Reference_DB
