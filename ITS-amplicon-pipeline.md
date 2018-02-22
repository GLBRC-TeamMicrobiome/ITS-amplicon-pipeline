# *ITS amplicon pipeline*
Please note, necessary folders should be created before running the scripts.

## 1) quality checking and pre-filtering 

### a) count read numbers
```
for fastq in *.fastq.gz
do gunzip -c "$fastq" | paste - - - - | wc -l && echo $fastq 
done > reads_raw.counts
```
### b) produce reads quality graphs using [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)  
```
cat *R1_001.fastq.gz > raw_reads_R1.fastq.gz; cat *R2_001.fastq.gz > raw_reads_R2.fastq.gz
fastqc ./raw_reads_R1.fastq.gz raw_reads_R2.fastq.gz -o stats && rm -rf raw_reads_R1.fastq.gz raw_reads_R2.fastq.gz
```

### c) filtering out Phyax reads using [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml)
```
mkdir no_phix
bowtie2 -x phix_index/my_phix -U $R1 -t -p 20 --un no_phix/$R1.nophix.fastq -S /no_phix/$R1.contaminated_align.sam 2> no_phix/$R1.nophix.log
```

### d) re-sync fwd and rew reads
`cd no_phix/`

```
ls *gz.nophix.fastq > fastq_no_phix.list
while read R1
do read R2
python phyton_scripts/fastqCombinePairedEnd.py $R1 $R2
done < fastq_no_phix.list
```

## 2) Read Assembly [USEARCH](https://www.drive5.com/usearch/manual/cmd_fastq_mergepairs.html)
```
cd re_sync/
ls *pairs*.fastq > pairs.list
```
`mkdir assembled`

```
while read R1
do read R2
/mnt/research/rdp/public/thirdParty/usearch10.0.240_i86linux64 -fastq_mergepairs $R1 -reverse $R2 -fastq_minmergelen 50 -relabel @ -fastqout assembled/$R1
done < pairs.list
```

## 3) removing primers and adapters with [cutadapt](http://cutadapt.readthedocs.io/en/stable/index.html)
`fwd=GAACCWGCGGARGGATCA #ITS1FI2` 
`rev=GCATCGATGAAGAACGCAGC #ITS3`
`revcomp=GCTGCGTTCTTCATCGATGC`

```
for fastq in *.fastq
do cutadapt -g ^$fwd -a $revcomp$ -f fastq -n 2 --discard-untrimmed --match-read-wildcards -o ${fastq//.fastq/.stripped.fastq} $fastq
done
```

## 4) getting stats using USEARCH and [VSEARCH](https://github.com/torognes/vsearch)

```
vsearch -fastq_stats assembled.fastq -log stats_results_VSEARCH.txt
/mnt/research/rdp/public/thirdParty/usearch10.0.240_i86linux64 -fastq_eestats2 assembled.fastq -output stats_eestats2_USEARCH.txt -length_cutoffs 100,500,1
/mnt/research/rdp/public/thirdParty/usearch10.0.240_i86linux64 -fastx_info assembled.fastq -secs 5 -output stats_fastxinfo_USEARCH.txt
```
## 5) filtering, trimming, and quality check

```
for fastq in *_R1.fastq
do /mnt/research/rdp/public/thirdParty/usearch10.0.240_i86linux64 -fastq_filter $fastq -fastq_maxee 1 -fastq_trunclen 250 -fastq_maxns 0 -fastqout ${fastq//_R1.fastq/.filtered.fastq} -fastaout ${fastq//_R1.fastq/.filtered.fasta} 
done

cat *filterd.fastq > filtered.fastq
fastqc ./filtered.fastq
```
## 6) clustering and denoising (OTUs vs. ESV)

### a) generating [Exact Sequence Variants](https://www.drive5.com/usearch/manual/faq_uparse_or_unoise.html) (ESV) also called 0-radius OTUs

```
mnt/research/rdp/public/thirdParty/usearch10.0.240_i86linux64 -fastx_uniques filtered.fasta -fastaout uniques.fasta -sizeout
/mnt/research/rdp/public/thirdParty/usearch10.0.240_i86linux64 -unoise3 uniques.fasta -tabbedout unoise_zotus.txt -zotus zotus.fasta
python python_scripts/fasta_number.py zotus.fasta OTU_ > zotus_numbered.fasta
/mnt/research/rdp/public/thirdParty/usearch10.0.240_i86linux64 -usearch_global assembled.fasta -db zotus_numbered.fasta -strand plus -id 0.97 -otutabout otu_table_ITS_UNOISE.txt
```
### b) [clustering](https://www.drive5.com/usearch/manual/cmd_cluster_otus.html) OTUs at 97% sequence similarity 

```
/mnt/research/rdp/public/thirdParty/usearch10.0.240_i86linux64 -cluster_otus uniques.fasta -minsize 2 -otus otus.fasta -uparseout uparse_otus.txt -relabel OTU_ --threads 20
/mnt/research/rdp/public/thirdParty/usearch10.0.240_i86linux64 -usearch_global assembled.fasta -db otus.fasta -strand plus -id 0.97 -otutabout otu_table_ITS_UPARSE.txt
```

## 7) Taxonomic classification using [CONSTAX](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-017-1952-x) (or eventually RDP)

### a) for using CONSTAX refer to [this](https://github.com/natalie-vandepol/compare_taxonomy) link  
We will incorporate SILVA database in CONSTAX (soon), so if you like we can definitely use it for all our datasets. More than creating a consensus taxonomy, it does some "cosmetic" work on the taxonomy table so that you will not need to do much after you have it imported in R.

### b) for using [RDP](https://github.com/rdpstaff/classifier) Classifier
Download the most recent version of the [UNITE](https://unite.ut.ee/repository.php) database - general fasta format
Generate separated database.fasta and taxonomy.txt files using the custom script below, extracted form our the [CONSTAX tool](https://github.com/Gian77/COSTAX) 

```
generaldb_to_RDPdb.py sh_general_release_dynamic_s_01.12.2017.fasta```

```
Train the taxonomy using RDP scripts and run the java `classifier.jar`
```
python lineage2taxTrain.py sh_general_release_dynamic_s_01.12.2017__RDP_taxonomy.txt > sh_general_release_dynamic_s_01.12.2017__RDP_taxonomy_trained.txt

python addFullLineage.py sh_general_release_dynamic_s_01.12.2017__RDP_taxonomy.txt sh_general_release_dynamic_s_01.12.2017__RDP.fasta > sh_general_release_dynamic_s_01.12.2017__RDP_trained.fasta

java -Xmx32000m -jar /mnt/research/rdp/public/RDPTools/classifier.jar train -o training_files -s sh_general_release_dynamic_s_01.12.2017__RDP_trained.fasta -t sh_general_release_dynamic_s_01.12.2017__RDP_taxonomy_trained.txt

```
Copy the properties file in your `training_files/` folder
```
cd /mnt/research/rdp/public/RDPTools/classifier/samplefiles/
cp rRNAClassifier.properties /path-to-files/training_files/.
```

Assign taxonmy IDs to your otus.fasta file
```
java -Xmx32000m -jar /mnt/research/rdp/public/RDPTools/classifier.jar classify --conf 0.8 --format allrank --train_propfile training_files/rRNAClassifier.properties -o otus_taxonomy.txt ../path-to-files/otus.fasta
```





