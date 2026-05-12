# Descargar datos desde el Sequence Read Archive (SRA)
El SRA Toolkit (SRA Tools) es un conjunto de herramientas de software de línea de comandos desarrollado por el NCBI para descargar, manipular y convertir datos de secuenciación de ADN/ARN (lecturas cortas) desde el repositorio público Sequence Read Archive (SRA) a formatos utilizables como FASTQ.

## Instalar sra-tools
### Crear un ambiente conda
Los ambientes conda permiten instalar software de forma aislada sin afectar el sistema global del clúster, y sin solicitar permisos al administrador. Cada ambiente es independiente y reproducible — cualquier persona puede recrearlo con el mismo comando.

El ambiente sratools se creó con:
```
conda create -n sratools -c bioconda sra-tools -y
```
Donde -c bioconda especifica el canal de donde se obtienen las herramientas bioinformáticas. Una vez creado, se activa con conda activate sratools antes de cada sesión de trabajo.
### Instalar sra-tools via mamba que resuelve dependencias mejor:
```
conda install -c conda-forge mamba -y
mamba install -c bioconda -c conda-forge sra-tools -y
```
Para garantizar su activación dentro de scripts SLURM — que no cargan el .bashrc automáticamente — se requiere cargar explícitamente el módulo de anaconda al inicio de cada script:
```
module load envs/anaconda3
source $(conda info --base)/etc/profile.d/conda.sh
conda activate sratools
```

