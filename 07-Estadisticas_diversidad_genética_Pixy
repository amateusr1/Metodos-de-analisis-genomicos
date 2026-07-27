# Mapeo de secuencias WGS contra genoma de cloroplasto de referencia.

BBMap es un alineador de lecturas cortas rápido y preciso para datos de secuenciación de ADN y ARN, capaz de manejar grandes volúmenes de datos. Escrito en Java, es versátil para plataformas como Illumina, PacBio y Nanopore, destacando por su alta tolerancia a errores, inserciones y deleciones largas

Se descargaron lecturas pareadas de ILLUMINA WGS de genoma completo de solanum pimpinellifolium. Estas se mapearon contra el genoma completo de cloroplastop de solanum lycopersicum var lycopersicum como referencia para hacer unicamente el llamdo de las secuencias de cloroplasto de pimpinellifolium. El comando para mapear y extraer solo las lecturas plastidiales de lecturas de genoma completo WGS es:
```
#!/bin/bash
#SBATCH --job-name=bbmap
#SBATCH --nodelist=hercules4
#SBATCH --clusters=biocomputo
#SBATCH --partition=cpu.cecc
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=4
#SBATCH --output=bbmap_%j.out
#SBATCH --error=bbmap_%j.err

module load envs/anaconda3
source $(conda info --base)/etc/profile.d/conda.sh
conda activate captus_1.6.5

bbmap.sh ref=/scratchsan/amateusr/Slycopersicum_chloroplast.fasta \
    in1=/scratchsan/amateusr/SRR38359005_1.fastq.gz \
    in2=/scratchsan/amateusr/SRR38359005_2.fastq.gz \
    outm1=/scratchsan/amateusr/chloroplast_1.fastq.gz \
    outm2=/scratchsan/amateusr/chloroplast_2.fastq.gz \
    fast=t \
    threads=4
```
