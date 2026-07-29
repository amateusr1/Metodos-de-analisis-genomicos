# 📥 Descargar datos desde el Sequence Read Archive (SRA)

> **Autor:** Andrey Mateus-Ruiz  
> **Curso:** Métodos de Análisis Genómicos (2018846-6)  
> **Profesor:** Ph.D. Gustavo Silva-Arias  
> **Universidad Nacional de Colombia**

---

# Descripción

El **Sequence Read Archive (SRA)** es el mayor repositorio público de datos de secuenciación de ADN y ARN, administrado por el **National Center for Biotechnology Information (NCBI)**. En él se almacenan millones de experimentos provenientes de proyectos de genómica, transcriptómica, metagenómica y otras aplicaciones de secuenciación masiva.

Para acceder a estos datos desde la línea de comandos se utiliza **SRA Toolkit**, un conjunto de herramientas desarrollado por el NCBI que permite descargar, convertir y gestionar archivos almacenados en el formato SRA.

Durante este curso utilizamos **fasterq-dump**, la herramienta recomendada para convertir archivos SRA en archivos **FASTQ**, el formato estándar empleado por la mayoría de los programas de bioinformática.

> **Nota**
>
> Este documento se encuentra en constante construcción y actualización. Hace parte de la bitácora técnica del curso y continuará incorporando nuevos ejemplos, recomendaciones y buenas prácticas.

---

# Objetivos

Al finalizar este módulo podrás:

- Comprender qué es el Sequence Read Archive (SRA).
- Instalar SRA Toolkit utilizando Conda.
- Crear un ambiente reproducible para el análisis.
- Descargar datos de secuenciación mediante `fasterq-dump`.
- Automatizar descargas de múltiples muestras.
- Comprimir y organizar archivos FASTQ para análisis posteriores.

---

# ¿Qué es el Sequence Read Archive?

El Sequence Read Archive (SRA) almacena datos crudos de secuenciación provenientes de diferentes plataformas como:

- Illumina
- PacBio
- Oxford Nanopore
- Ion Torrent

Cada experimento posee un identificador único denominado **Accession**, por ejemplo

```
SRR31477438
```

Estos identificadores serán utilizados para descargar los datos.

---

# Instalación de SRA Toolkit

La forma más sencilla de instalar SRA Toolkit es mediante Conda.

Crear el ambiente

```bash
conda create -n sratools -c bioconda sra-tools -y
```

Activar el ambiente

```bash
conda activate sratools
```

Verificar la instalación

```bash
fasterq-dump --version
```

---

# Instalación utilizando Mamba

Cuando Conda presenta conflictos de dependencias, **Mamba** suele resolverlas considerablemente más rápido.

Instalar Mamba

```bash
conda install -c conda-forge mamba -y
```

Instalar SRA Toolkit

```bash
mamba install -c bioconda -c conda-forge sra-tools -y
```

Verificar

```bash
fasterq-dump --version
```

---

# Activar el ambiente en un clúster HPC

En muchos clústeres es necesario cargar primero el módulo de Anaconda.

En consola

```bash
module load envs/anaconda3

source $(conda info --base)/etc/profile.d/conda.sh

conda activate sratools
```

En un script SLURM

```bash
module load envs/anaconda3

source $(conda info --base)/etc/profile.d/conda.sh

conda activate sratools
```

---

# Descargar una única muestra

Supongamos que queremos descargar la muestra

```
SRR31477438
```

El comando es

```bash
fasterq-dump SRR31477438
```

Si la muestra es de lecturas pareadas (*paired-end*), se obtendrán dos archivos

```
SRR31477438_1.fastq
SRR31477438_2.fastq
```

---

# Utilizar múltiples hilos

La descarga puede acelerarse indicando el número de núcleos disponibles.

```bash
fasterq-dump SRR31477438 \
-e 10
```

donde

- `-e` especifica el número de hilos utilizados durante la conversión del archivo SRA.

---

# Comprimir los archivos

`fasterq-dump` genera archivos FASTQ sin comprimir.

Una vez finalizada la descarga es recomendable comprimirlos.

```bash
gzip SRR31477438*.fastq
```

o todos los FASTQ del directorio

```bash
gzip *.fastq
```

---

# Descargar múltiples muestras automáticamente

Crear un archivo llamado

```
Sra.txt
```

con un identificador por línea

```
SRR31477438
SRR38359005
SRR38359006
```

Posteriormente utilizar el siguiente script.

```

module load envs/anaconda3

source $(conda info --base)/etc/profile.d/conda.sh

conda activate sratools

while read run
do

    echo "Descargando ${run}"

    fasterq-dump \
        ${run} \
        -e 10

    gzip ${run}_*.fastq

done < Sra.txt
```

---

# Explicación del script

El script realiza automáticamente las siguientes acciones:

1. Carga el ambiente Conda donde se encuentra instalado SRA Toolkit.

2. Lee cada identificador almacenado en `Sra.txt`.

3. Descarga cada experimento utilizando `fasterq-dump`.

4. Convierte el archivo SRA al formato FASTQ.

5. Comprime los archivos FASTQ mediante `gzip`.

6. Continúa con la siguiente muestra hasta finalizar todas las descargas.

---

# Verificar la descarga

Comprobar que los archivos fueron creados

```bash
ls -lh *.fastq.gz
```

Visualizar las primeras secuencias

```bash
zcat SRR31477438_1.fastq.gz | head -12
```

Contar el número de archivos descargados

```bash
ls *.fastq.gz | wc -l
```

---

# Buenas prácticas

- Mantener un archivo `Sra.txt` con todos los accesiones utilizados en el proyecto.
- Comprimir los archivos inmediatamente después de la descarga para ahorrar espacio en disco.
- Verificar que las descargas finalizaron correctamente antes de continuar con el análisis.
- Conservar los archivos `.out` y `.err` generados por SLURM para facilitar la identificación de errores.

---

# Próximo módulo

En el siguiente módulo realizaremos el **control de calidad de las lecturas** utilizando herramientas como **Trimmomatic** **Modúlo clean de Captus** **FastQC** y **BBTools**, preparando los datos para el ensamblaje o el alineamiento contra un genoma de referencia.

