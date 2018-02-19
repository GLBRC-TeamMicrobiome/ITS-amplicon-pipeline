# ITS amplcon pipeline

## 1) quality checking and pre-filtering 

### a) count read numbers
`for fastq in *.fastq.gz`
`do gunzip -c "$fastq" | paste - - - - | wc -l && echo $fastq` 
`done > reads_raw.counts`

### b) produce redas quality graphs using FastQC 
`mkdir stats`
`cat *R1_001.fastq.gz > raw_reads_R1.fastq.gz; cat *R2_001.fastq.gz > raw_reads_R2.fastq.gz`
`fastqc ./raw_reads_R1.fastq.gz raw_reads_R2.fastq.gz -o stats && rm -rf raw_reads_R1.fastq.gz raw_reads_R2.fastq.gz`

### c) filtering out Phyax reads
`mkdir no_phux`
`bowtie2 -x /mnt/home/benucci/phix_index/my_phix -U $R1 -t -p 20 --un no_phix/$R1.nophix.fastq -S ../no_phix/$R1.contaminated_align.sam 2> ../no_phix/$R1.nophix.log`

### d) re-sync fwd and rew reads
`cd no_phix/`
`ls *gz.nophix.fastq > fastq_no_phix.list`
`while read R1`
`do read R2`
`python ../Phyton_scripts/fastqCombinePairedEnd.py $R1 $R2`
`done < fastq_no_phix.list`

## 2) 




[link to Google!](http://google.com)

