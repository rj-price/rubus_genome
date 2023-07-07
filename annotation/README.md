# Annotation
# run softmasking using Red
for file in ../ragtag/*fasta;
    do fileshort=$(basename $file | sed s/".fasta"//g)
    mkdir -p repeat_masked/genome/"$fileshort"
    cp $file repeat_masked/genome/"$fileshort"/"$fileshort".fa
    mkdir -p repeat_masked/red/"$fileshort"
    done

for file in ../../../ragtag/*fasta;
    do fileshort=$(basename $file | sed s/".fasta"//g)
    sbatch red.sh $fileshort
    done


# QC reads
for file in *fq; do sbatch ~/scratch/private/scripts/genome_assembly/fastqc.sh $file; done

# Trim Jahn's data (ILLUMINACLIP, HEADCROP:10, MINLEN:100)
for file in ../*val_1.fq; 
    do file2=$(ls $file | sed s/"R1_val_1.fq"//g)
    sbatch ~/scratch/private/scripts/genome_assembly/trimmomatic_pe.sh $file "$file2"R2_val_2.fq
    done

# Create index for genomes
for file in *HiC.fasta;
    do sbatch ~/scratch/private/scripts/genome_assembly/hisat2_index.sh $file;
    done

for file in /mnt/shared/scratch/jnprice/R_idaeus/ragtag/Anitra7LGPLOS.fasta;
    do sbatch ~/scratch/private/scripts/genome_assembly/hisat2_index.sh $file;
    done

# Align Jahn's RNA-seq
for file in /mnt/shared/scratch/jnprice/R_idaeus/jahn_data/trimmed/*F_paired.fastq.gz; 
    do file2=$(ls $file | sed s/"F_paired.fastq.gz"//g)
    sbatch ~/scratch/private/scripts/rna-seq/hisat2_pe-map.sh /mnt/shared/scratch/jnprice/R_idaeus/annotation/genome_index/RiAB_ragtag_HiC $file "$file2"R_paired.fastq.gz
    done

for file in /mnt/shared/scratch/jnprice/R_idaeus/jahn_data/trimmed/*F_paired.fastq.gz; 
    do file2=$(ls $file | sed s/"F_paired.fastq.gz"//g)
    sbatch ~/scratch/private/scripts/rna-seq/hisat2_pe-map.sh /mnt/shared/scratch/jnprice/R_idaeus/annotation/genome_index/RiMJ_ragtag_HiC $file "$file2"R_paired.fastq.gz
    done

for file in /mnt/shared/scratch/jnprice/R_idaeus/jahn_data/trimmed/*F_paired.fastq.gz; 
    do file2=$(ls $file | sed s/"F_paired.fastq.gz"//g)
    sbatch ~/scratch/private/scripts/rna-seq/hisat2_pe-map.sh /mnt/shared/scratch/jnprice/R_idaeus/annotation/genome_index/Anitra7LGPLOS.fasta $file "$file2"R_paired.fastq.gz
    done

## BRAKER annotation
for folder in /mnt/shared/scratch/jnprice/R_idaeus/annotation/repeat_masked/red/*;
    do foldershort=$(basename $folder)
    sbatch BRAKER_rna-prot.sh /mnt/shared/scratch/jnprice/R_idaeus/annotation/repeat_masked/red/$foldershort/"$foldershort".msk
    done

# Only keep longest isoforms
python ~/scratch/private/scripts/genome_assembly/longest_isoform_filter.py input.fa > out_longest.fa
