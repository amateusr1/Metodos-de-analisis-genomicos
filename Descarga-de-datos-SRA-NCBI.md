# Descargar datos desde el Sequence Read Archive (SRA)
El SRA Toolkit (SRA Tools) es un conjunto de herramientas de software de línea de comandos desarrollado por el NCBI para descargar, manipular y convertir datos de secuenciación de ADN/ARN (lecturas cortas) desde el repositorio público Sequence Read Archive (SRA) a formatos utilizables como FASTQ.

# Instalar sra-tools via mamba que resuelve dependencias mejor:
```
conda install -c conda-forge mamba -y
mamba install -c bioconda -c conda-forge sra-tools -y
```

