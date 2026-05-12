# Descargar datos desde el Sequence Read Archive (SRA)
El SRA Toolkit (SRA Tools) es un conjunto de herramientas de software de línea de comandos desarrollado por el NCBI para descargar, manipular y convertir datos de secuenciación de ADN/ARN (lecturas cortas) desde el repositorio público Sequence Read Archive (SRA) a formatos utilizables como FASTQ.

## Crear un ambiente conda de trabajo
Los ambientes conda permiten instalar software de forma aislada sin afectar el sistema global del clúster, y sin solicitar permisos al administrador. Cada ambiente es independiente y reproducible — cualquier persona puede recrearlo con el mismo comando.
El ambiente "sratools" se creó con:
```
conda create -n sratools -c bioconda sra-tools -y
#para activarlo
conda activate sratools
```
Una vez creado, se activa con conda activate sratools antes de cada sesión de trabajo.
###Activación en scripts SLURM
Los scripts SLURM no cargan el .bashrc automáticamente, por lo que se requiere cargar el módulo de anaconda explícitamente al inicio de cada script:
```
module load envs/anaconda3
source $(conda info --base)/etc/profile.d/conda.sh
conda activate sratools
```
## Instalar sra-tools via mamba que resuelve dependencias mejor:
Si la instalación directa con conda falla por conflicto de dependencias, se recomienda usar mamba, que resuelve dependencias de forma más eficiente:
```
conda install -c conda-forge mamba -y
mamba install -c bioconda -c conda-forge sra-tools -y
```
Verificar la instalación:
```
fastq-dump --version
fasterq-dump --version
```
## Descargar datos desde el Sequence Read Archive (SRA)
La descarga se automatizó con un loop que lee los accesiones desde un archivo Sra.txt (uno por línea), descarga cada uno con fasterq-dump y comprime los archivos resultantes:
```
#!/bin/bash
#SBATCH --job-name=job1        # Job name
#SBATCH --nodelist=hercules3          # Node select
#SBATCH --clusters=biocomputo           # Cluster name
#SBATCH --partition=cpu.cecc           # partition Name, is ~ -q normal.q in SGE
#SBATCH --nodes=1                      # Number of compute nodes for the job.
#SBATCH --ntasks-per-node=1           # Corresponds to number of task/works on the compute node.
#SBATCH --cpus-per-task=10              # Corresponds to number of cores on the compute node.
#SBATCH --output=job1_%j.out     # where to store the standart output,  (%j is the JOBID)
#SBATCH --error=job1_%j.err      # where to store the standart error output,  (%j is the JOBID)

# Your script goes here

module load envs/anaconda3
source $(conda info --base)/etc/profile.d/conda.sh

conda activate sratools

while read run; do
    fasterq-dump $run -e 10
    gzip ${run}_*.fastq
done < Sra.txt
```
El script separa las lecturas pareadas en dos archivos (_1.fastq.gz y _2.fastq.gz). El flag --gzip comprime los archivos durante la descarga, reduciendo el espacio en disco requerido. Donde -e 10 especifica el número de hilos para la descarga. 

