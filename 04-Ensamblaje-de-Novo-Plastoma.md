# Ensamblaje de Novo - Plastoma

> Una guía sobre los fundamentos del ensamblaje de Novo y un flujo de trabajo completo con datos de NGS utilizando GetOrganell

---
## Introducción

El ensamblado de novo es un proceso bioinformático que permite reconstruir una secuencia genómica a partir de lecturas obtenidas mediante tecnologías de secuenciación, sin utilizar un genoma de referencia. Este enfoque es especialmente útil cuando se trabaja con especies cuyos genomas no han sido previamente secuenciados o cuando se busca identificar variaciones estructurales que podrían perderse en un ensamblado basado en referencia.

El proceso inicia con el control de calidad de las lecturas, en el que se eliminan adaptadores, bases de baja calidad y secuencias contaminantes. Posteriormente, el software de ensamblado identifica regiones de solapamiento entre las lecturas para construir fragmentos continuos de secuencia denominados contigs. En etapas posteriores, estos contigs pueden organizarse en estructuras de mayor longitud llamadas scaffolds, utilizando información adicional como lecturas pareadas o de larga longitud.

Los algoritmos de ensamblado de novo se basan principalmente en dos estrategias. La primera corresponde al método Overlap-Layout-Consensus (OLC), que compara directamente las lecturas para identificar sus solapamientos, siendo adecuado para lecturas largas. La segunda estrategia utiliza grafos de De Bruijn, en los que las lecturas se dividen en fragmentos más pequeños llamados k-mers. Cada k-mer constituye un nodo del grafo y las conexiones entre ellos permiten reconstruir la secuencia original. Este último enfoque es ampliamente utilizado para datos provenientes de plataformas de secuenciación de lectura corta, como Illumina.

La calidad del ensamblado depende de factores como la profundidad de secuenciación, la longitud de las lecturas, la presencia de regiones repetitivas, el contenido de errores de secuenciación y la complejidad del genoma. Para evaluar el resultado se emplean métricas como el número de contigs, la longitud total ensamblada, el valor N50, el porcentaje de cobertura y la completitud del genoma obtenido.

Los plastidios son orgánulos presentes en plantas y algas que contienen un genoma propio, generalmente de estructura circular, altamente conservado y con un tamaño aproximado entre 120 y 170 kilobases en plantas terrestres. El genoma plastidial suele presentar una organización característica formada por una región larga de copia única (LSC, Large Single Copy), una región corta de copia única (SSC, Small Single Copy) y dos regiones repetidas invertidas (IR, Inverted Repeats).

El ensamblado de genomas plastidiales puede realizarse a partir de datos de secuenciación del ADN total debido a que las células vegetales contienen numerosas copias del genoma del cloroplasto, lo que genera una alta cobertura de estas secuencias. En consecuencia, es posible reconstruir el genoma plastidial sin necesidad de aislar previamente los cloroplastos.

Existen diferentes estrategias para el ensamblado de genomas plastidiales. Una consiste en extraer previamente las lecturas correspondientes al plastidio mediante alineamiento contra un genoma de referencia cercano y posteriormente ensamblarlas. Otra alternativa es realizar un ensamblado de novo utilizando únicamente la información contenida en las lecturas y, posteriormente, identificar los contigs pertenecientes al genoma plastidial. Herramientas especializadas como GetOrganelle, NOVOPlasty y Fast-Plast han sido desarrolladas para facilitar este proceso mediante algoritmos optimizados para genomas organelares.

Una vez obtenido el ensamblado, es necesario verificar que el genoma sea completo y presente la estructura cuadripartita esperada. Posteriormente, se realiza la anotación de genes codificantes de proteínas, ARN ribosomales (rRNA) y ARN de transferencia (tRNA), así como la identificación de las regiones LSC, SSC e IR. La calidad del ensamblado también puede confirmarse mediante el análisis de la cobertura de secuenciación y la comparación con genomas plastidiales de especies relacionadas.

Los genomas plastidiales constituyen una fuente importante de información para estudios de evolución, filogenia, sistemática, biogeografía y genética de poblaciones, debido a su relativa conservación estructural, su herencia predominantemente materna en la mayoría de las angiospermas y su baja tasa de recombinación.

Mapeo de secuencias WGS contra genoma de cloroplasto de referencia.
BBMap es un alineador de lecturas cortas rápido y preciso para datos de secuenciación de ADN y ARN, capaz de manejar grandes volúmenes de datos. Escrito en Java, es versátil para plataformas como Illumina, PacBio y Nanopore, destacando por su alta tolerancia a errores, inserciones y deleciones largas

Se descargaron lecturas pareadas de ILLUMINA WGS de genoma completo de solanum pimpinellifolium. Estas se mapearon contra el genoma completo de cloroplastop de solanum lycopersicum var lycopersicum como referencia para hacer unicamente el llamdo de las secuencias de cloroplasto de pimpinellifolium. El comando para mapear y extraer solo las lecturas plastidiales de lecturas de genoma completo WGS es:

# Un flujo de trabajo completo para el ensamblaje del genoma del cloroplasto

En este módulo se procesaron tres muestras de *Solanum sección lycopersicum* secuenciadas por WGS con Illumina: una accesión de *S. lycopersicum var. cerasiforme* (SLC; SRR31477438), una accesión de *S. lycopersicum var. lycopersicum* (LA1924; SRR38359005) y una accesión de *S. pimpinellifolium* (SRR37254991), mapeadas contra el genoma de referencia del tomate cultivado (*S. lycopersicum var. lycopersicum* Micro-Tom, ensamblaje SLM_r2.1 (GCF_036512215.1; 832.8 Mb). Las lecturas WGS de las muestras fueron descargadas desde la base de datos SRA del NCBI usando fasterq-dump, en formato FASTA.

---

Todos los análisis se ejecutaron en el clúster de cómputo HPC perteneciente a la Facultad de Ciencias de la Universidad Nacional de Colombia mediante scripts SLURM, utilizando el gestor de paquetes Conda y entornos virtuales asociados. La instalación y resolución de dependencias se ejecuto a través de Anaconda, Miniconda o Mamba.

```
# Carga del módulo de conda
module load envs/anaconda3
source /scratchsan1/anaconda3/etc/profile.d/conda.sh
```
---
## Enriquecimineto con ADN plastidial 

Las lecturas obtenidas mediante secuenciación Illumina fueron sometidas inicialmente a un proceso de enriquecimiento para recuperar únicamente aquellas correspondientes al genoma del cloroplasto. Para ello, se alinearon las lecturas pareadas contra un genoma de referencia plastidial Solanum lycopersicum chloroplast, complete genome-NC_007898.3 utilizando BBMap. Como resultado de este paso, se extrajeron exclusivamente los pares de lecturas que presentaron alineamientos con el genoma plastidial de referencia, reduciendo considerablemente la presencia de otras secuencias y el tamaño de los archivos.

```
bbmap.sh ref=Slycopersicum_chloroplast.fasta \
in1=reads_1.fastq.gz \
in2=reads_2.fastq.gz \
in2=reads_3.fastq.gz \
outm1=chloroplast_1.fastq.gz \
outm2=chloroplast_1.fastq.gz \
outm3=chloroplast_2.fastq.gz
```

Aquí BBMap:

Indexó el genoma del cloroplasto;
Comparó cada par de lecturas contra esa referencia;
Conservó únicamente los pares que alineaban;
Descartó las lecturas que provenían del genoma nuclear o mitocondrial.

---
# Ensamblaje del cloroplasto con GetOrganelle

Las lecturas enriquecidas fueron utilizadas como entrada para GetOrganelle, software especializado en el ensamblaje de genomas de organelos. Se empleó el modo embplant_pt, diseñado específicamente para genomas de cloroplastos de plantas terrestres. GetOrganelle realiza un reclutamiento iterativo de lecturas plastidiales y construye el ensamblaje mediante el algoritmo de gráficos de De Bruijn implementado en SPAdes, eliminando ramas espurias y resolviendo repeticiones para obtener un ensamblaje plastidial de alta calidad.

```
for sample in SRR38359005 SRR31477438 SRR37254991; do
get_organelle_from_reads.py \
-1 chloro_R1.fastq.gz \
-2 chloro_R2.fastq.gz \
-3 chloro_R3.fastq.gz \
-o /scratchsan/amateusr/outs/getorganelle/${sample}_chloro \
-F embplant_pt
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

    # Your script goes here

module load envs/anaconda3
conda activate sratools

for sample in SRR38359005 SRR31477438; do
    get_organelle_from_reads.py \
        -1 /scratchsan/amateusr/clean/${sample}_chloro_R1.fastq.gz \
        -2 /scratchsan/amateusr/clean/${sample}_chloro_R2.fastq.gz \
        -o /scratchsan/amateusr/clean/getorganelle/${sample}_chloro \
        -F embplant_pt
done

# ── Rutas ──────────────────────────────────────────────────
IN1=/scratchsan/amateusr/secuences/chloroplast_1.fastq.gz
IN2=/scratchsan/amateusr/secuences/chloroplast_2.fastq.gz
SUB1=/scratchsan/amateusr/secuences/chloro_sub_1.fastq.gz
SUB2=/scratchsan/amateusr/secuences/chloro_sub_2.fastq.gz
OUTDIR=/scratchsan/amateusr/outs

# ── Módulos ────────────────────────────────────────────────
module load apps/spades/3.15.4
module load apps/bbmap/38.34        # ajusta el nombre exacto si es diferente

# ── Paso 1: Submuestreo a ~200x (200,000 reads) ────────────
echo "MESSAGE: Submuestreando reads..."

reformat.sh \
  in1=$IN1 \
  in2=$IN2 \
  out1=$SUB1 \
  out2=$SUB2 \
  samplereadstarget=200000 \
  sampleseed=42 \
  threads=16

echo "MESSAGE: Reads en SUB1: $(zcat $SUB1 | awk 'NR%4==1' | wc -l)"
echo "MESSAGE: Reads en SUB2: $(zcat $SUB2 | awk 'NR%4==1' | wc -l)"

# ── Paso 2: Limpiar output anterior ────────────────────────

# ── Paso 3: Correr SPAdes ──────────────────────────────────
echo "MESSAGE: Iniciando SPAdes..."

spades.py \
  --careful \
  -1 $SUB1 \
  -2 $SUB2 \
  -o $OUTDIR \
  -t 16 \
  -m 60

# ── Paso 4: Verificar resultado ────────────────────────────
if [ -f "$OUTDIR/scaffolds.fasta" ]; then
    echo "MESSAGE: ¡Ensamble exitoso!"
    echo "MESSAGE: Scaffolds generados:"
    grep "^>" $OUTDIR/scaffolds.fasta | head -10
    echo "MESSAGE: Total de scaffolds: $(grep -c '^>' $OUTDIR/scaffolds.fasta)"
else
    echo "ERROR: No se generó scaffolds.fasta — revisa $OUTDIR/spades.log"
fi

