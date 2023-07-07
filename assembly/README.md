# Assembly

# Trim adapters only from passed reads using Porechop
for folder in ../Ri*;
    do foldershort=$(basename $folder)
    sbatch /mnt/shared/scratch/jnprice/private/scripts/genome_assembly/porechop.sh $folder/high_acc/pass/ /mnt/shared/scratch/jnprice/R_idaeus/new_calling/porechop/$foldershort.fastq
    done

# Run NanoPlot on new read sets
for file in /mnt/shared/scratch/jnprice/R_idaeus/new_calling/porechop/*.fastq.gz;
    do fileshort=$(basename $file | sed s/".fastq.gz"//g)
    sbatch /mnt/shared/scratch/jnprice/private/scripts/genome_assembly/nanoplot_fastq.sh $file /mnt/shared/scratch/jnprice/R_idaeus/new_calling/nanoplot/trim/$fileshort/
    done

# Filter
for file in /mnt/shared/scratch/jnprice/R_idaeus/new_calling/porechop/*.fastq.gz;
    do fileshort=$(basename $file | sed s/".fastq.gz"//g)
    sbatch /mnt/shared/scratch/jnprice/private/scripts/genome_assembly/filtlong_1kb+10kb.sh $file $fileshort
    done

## NECAT
# config files
for file in /mnt/shared/scratch/jnprice/R_idaeus/new_calling/filtlong/*_1kb.fastq.gz; do bash /mnt/shared/scratch/jnprice/private/scripts/genome_assembly/necat_config.sh $file; done

# ^^^ Edit config files as below
PROJECT=<strain>
ONT_READ_LIST=read_list.txt
GENOME_SIZE=300000000
THREADS=16

# Run assembly
sbatch /mnt/shared/scratch/jnprice/private/scripts/genome_assembly/necat.sh {config}

# QC
for file in *.fasta; do sbatch /mnt/shared/scratch/jnprice/private/scripts/ortholog_analysis/busco_eudicots_genome.sh $file; done
for file in *.fasta; do python /mnt/shared/scratch/jnprice/private/scripts/genome_assembly/assembly_stats.py $file; done && python /mnt/shared/scratch/jnprice/private/scripts/genome_assembly/assembly_stats_concat.py *stats
cp BUSCO_*/short* . && for file in *txt; do python /mnt/shared/scratch/jnprice/private/scripts/genome_assembly/busco_extract.py $file; done && python /mnt/shared/scratch/jnprice/private/scripts/genome_assembly/busco_concat.py *busco

# Run purge_dups on NECAT assemblies
for file in /mnt/shared/scratch/jnprice/R_idaeus/new_calling/filtlong/*_1kb.fastq.gz;
    do fileshort=$(basename $file | sed s/"_1kb.fastq.gz"//g)
    sbatch ~/scratch/private/scripts/genome_assembly/purge_dups.sh /mnt/shared/scratch/jnprice/R_idaeus/new_calling/necat/"$fileshort"_1kb_necat.fasta $file
    done

## Racon (x1) & Medaka
for file in /mnt/shared/scratch/jnprice/R_idaeus/new_calling/purge_dups/*.fasta; 
    do fileshort=$(basename $file | sed s/"_1kb_necat.purged.fasta"//g)
    sbatch ~/scratch/private/scripts/genome_assembly/long_polish.sh /mnt/shared/scratch/jnprice/R_idaeus/new_calling/filtlong/"$fileshort"_1kb.fastq.gz $file
    done

## QC of short reads
fastqc /projects/oldhome/groups/harrisonlab/raw_data/raw_seq/mallingjewel/TGAC_ENQ_1176/*.fastq.gz -o /projects/ensa/plants/Ridaeus/fastqc


## Trim reads using Trimmomatic
idaeus_trim.sh

#!/usr/bin/env bash
#SBATCH --partition=long
#SBATCH --time=0-48:00:00
#SBATCH --mem=64G
#SBATCH --cpus-per-task=16

for file in /projects/oldhome/groups/harrisonlab/raw_data/raw_seq/mallingjewel/TGAC_ENQ_1176/*.fastq.gz;
    do fileshort=$(basename $file | sed s/".fastq.gz"//g)
    trimmomatic SE -phred33 $file /projects/ensa/plants/Ridaeus/$fileshort.trimmed.fq.gz ILLUMINACLIP:TruSeq3-SE:2:30:10 SLIDINGWINDOW:4:15 MINLEN:50
    fastqc /projects/ensa/plants/Ridaeus/$fileshort.trimmed.fq.gz -o /projects/ensa/plants/Ridaeus/fastqc_trim
    done

## Pilon
sbatch ~/scratch/private/scripts/genome_assembly/pilon_1lib-aa.sh \
    /mnt/shared/scratch/jnprice/R_idaeus/long_polish/RiAB_1kb_necat.purged_medaka.fasta \
    /mnt/shared/scratch/jnprice/R_idaeus/illumina_reads/trimmed/AutumnBliss_S1_L001_R1_001_F_paired.fastq.gz \
    /mnt/shared/scratch/jnprice/R_idaeus/illumina_reads/trimmed/AutumnBliss_S1_L001_R1_001_R_paired.fastq.gz \
    ./RiAB \
    3

sbatch ~/scratch/private/scripts/genome_assembly/pilon_4lib-aa.sh \
    /mnt/shared/scratch/jnprice/R_idaeus/long_polish/RiMJ_1kb_necat.purged_medaka.fasta \
    /mnt/shared/scratch/jnprice/R_idaeus/illumina_reads/trimmed/1628_LIB19366_LDI16701_CTTGTA_L001_R1_F_paired.fastq.gz \
    /mnt/shared/scratch/jnprice/R_idaeus/illumina_reads/trimmed/1628_LIB19366_LDI16701_CTTGTA_L001_R1_R_paired.fastq.gz \
    /mnt/shared/scratch/jnprice/R_idaeus/illumina_reads/trimmed/1650_LIB19366_LDI16701_CTTGTA_L001_R1_F_paired.fastq.gz \
    /mnt/shared/scratch/jnprice/R_idaeus/illumina_reads/trimmed/1650_LIB19366_LDI16701_CTTGTA_L001_R1_R_paired.fastq.gz \
    /mnt/shared/scratch/jnprice/R_idaeus/illumina_reads/trimmed/1650_LIB19366_LDI16701_CTTGTA_L002_R1_F_paired.fastq.gz \
    /mnt/shared/scratch/jnprice/R_idaeus/illumina_reads/trimmed/1650_LIB19366_LDI16701_CTTGTA_L002_R1_R_paired.fastq.gz \
    /mnt/shared/scratch/jnprice/R_idaeus/illumina_reads/trimmed/1670_LIB19366_LDI16701_CTTGTA_L001_R1_F_paired.fastq.gz \
    /mnt/shared/scratch/jnprice/R_idaeus/illumina_reads/trimmed/1670_LIB19366_LDI16701_CTTGTA_L001_R1_R_paired.fastq.gz \
    ./RiMJ \
    3

KAT

# Reference based assembly with Anitra genome
sbatch ~/scratch/private/scripts/genome_assembly/ragtag.sh Anitra7LGPLOS.fasta RiAB_pilon.fasta
sbatch ~/scratch/private/scripts/genome_assembly/ragtag.sh Anitra7LGPLOS.fasta RiMJ_pilon.fasta
