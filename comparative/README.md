# Comparative Analysis


D-genies

# QC proteomes
for file in *.fasta; do sbatch /mnt/shared/scratch/jnprice/private/scripts/ortholog_analysis/busco_eudicots_proteome.sh $file; done

OrthoFinder

# Circos (Anitra vs RagTag)
sbatch ~/scratch/private/scripts/circos/satsuma_synteny.sh Anitra7LGPLOS.fasta RiAB_ragtag_HiC.fasta circos/Anitra_vs_RiAB
sbatch ~/scratch/private/scripts/circos/satsuma_synteny.sh Anitra7LGPLOS.fasta RiMJ_ragtag_HiC.fasta circos/Anitra_vs_RiMJ

pyCircos