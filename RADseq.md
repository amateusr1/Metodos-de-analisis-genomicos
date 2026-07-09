# Restriction-site Associated DNA Sequencing (RADseq)

> Una guía sobre los métodos de secuenciación de representación reducida del genoma basados en enzimas de restricción.

---

# ¿Qué es RADseq?

La **Restriction-site Associated DNA Sequencing (RADseq)** es una familia de técnicas de secuenciación de representación reducida (*Reduced Representation Sequencing, RRS*) diseñadas para obtener información genética de una fracción del genoma mediante el uso de **enzimas de restricción**.

A diferencia de la secuenciación del genoma completo (**Whole Genome Sequencing, WGS**), RADseq no pretende secuenciar cada nucleótido del ADN. En su lugar, selecciona únicamente un subconjunto reproducible de regiones distribuidas a lo largo del genoma. Esto reduce drásticamente los costos de secuenciación y análisis, permitiendo estudiar cientos o miles de individuos simultáneamente.

Estas metodologías son especialmente útiles en organismos no modelo, donde no existe un genoma de referencia o el costo de un WGS resulta prohibitivo.

---

# ¿Cómo funciona RADseq?

Aunque existen diferentes variantes, todas comparten el mismo principio:

1. Extracción del ADN genómico.
2. Digestión del ADN con una o varias enzimas de restricción.
3. Ligación de adaptadores a los extremos generados por las enzimas.
4. Selección de una fracción del genoma (mediante tamaño o combinaciones de enzimas).
5. Amplificación mediante PCR.
6. Secuenciación en plataformas Illumina.
7. Identificación de SNPs y otros polimorfismos.
   
---
<p align="center">
  <img width="720" height="1280" alt="image" src="https://github.com/user-attachments/assets/b3c91d55-c2f1-414f-a477-d907fc421683" />

</p>

<p align="center">
<b>Figura 1.</b> Esquema general de la preparación de bibliotecas basadas en sitios de restricción (RADseq). El diagrama resume las etapas comunes a la familia de métodos RADseq. Dependiendo del protocolo empleado, la reducción adicional de la complejidad del genoma puede realizarse mediante fragmentación mecánica (RADseq original) o mediante una segunda digestión con enzimas de restricción (ddRADseq).
</p>

---

# ¿Por qué utilizar RADseq?

Las técnicas RADseq surgieron para resolver un problema común en genómica: secuenciar genomas completos de muchos individuos puede ser innecesario y muy costoso.

En muchos estudios de genética de poblaciones, evolución o filogenómica, basta con analizar miles de marcadores distribuidos por todo el genoma para responder preguntas biológicas.

RADseq permite obtener esos marcadores de manera eficiente y reproducible, multiplexar numerosos individuos en una misma corrida de secuenciación y reducir considerablemente el costo respecto al WGS.

---

# Los métodos RADseq

Desde la publicación del protocolo original en 2008, numerosos investigadores han desarrollado variantes que buscan simplificar el protocolo, disminuir costos o mejorar la reproducibilidad. Cada variante modifica alguna etapa del procedimiento, como el número de enzimas utilizadas, la forma de seleccionar los fragmentos o el método de preparación de bibliotecas.

| Método | Año | Descripción |
|---------|:---:|-------------|
| **RRL (Reduced Representation Libraries)** | 2006 | Primer método de representación reducida del genoma. Utiliza una enzima de restricción seguida de una selección directa de fragmentos por tamaño antes de la secuenciación. |
| **RADseq** | 2008 | Introduce el uso de adaptadores específicos para sitios de restricción y una fragmentación mecánica (sonicación) del ADN para obtener miles de loci distribuidos por el genoma. |
| **GBS (Genotyping-by-Sequencing)** | 2010 | Simplifica el protocolo de RADseq eliminando la sonicación y la selección física de tamaño, reduciendo costos y permitiendo el procesamiento de un gran número de muestras. |
| **CRoPS (Complexity Reduction of Polymorphic Sequences)** | 2011 | Emplea dos enzimas de restricción para reducir la complejidad del genoma y generar bibliotecas dirigidas al descubrimiento de polimorfismos. |
| **2bRAD** | 2011 | Utiliza enzimas de restricción tipo IIB, las cuales producen fragmentos de longitud uniforme (≈33–36 pb), simplificando el análisis bioinformático. |
| **ddRADseq (Double Digest RADseq)** | 2012 | Sustituye la fragmentación mecánica por una segunda enzima de restricción, aumentando la reproducibilidad y permitiendo controlar el número de loci mediante la selección de tamaño. |
| **MSG (Multiplexed Shotgun Genotyping)** | 2012 | Diseñado para el genotipado masivo mediante multiplexación, optimizando la construcción de mapas genéticos y estudios de genética de poblaciones. |
| **SBG (Sequence-Based Genotyping)** | 2012 | Combina enzimas de corte frecuente y raro para seleccionar un subconjunto específico de loci, disminuyendo la complejidad del genoma de forma controlada. |
| **ezRAD** | 2013 | Adapta RADseq para utilizar kits comerciales estándar de preparación de bibliotecas Illumina, facilitando su implementación en organismos no modelo. |

---

# Principales variantes

Aunque todas pertenecen a la familia RADseq, cada una posee características particulares.

## RRL (Reduced Representation Libraries)

Fue el precursor conceptual de RADseq. Consiste en cortar el ADN con una enzima de restricción y seleccionar únicamente fragmentos dentro de un rango específico de tamaño antes de la secuenciación. No utiliza adaptadores específicos para sitios de restricción.

---

## RADseq original

Publicado por **Baird et al. (2008)**.

Utiliza una única enzima de restricción para generar sitios de corte, seguida de una fragmentación mecánica del ADN mediante sonicación. Solo los fragmentos que contienen un sitio de restricción reciben adaptadores y son secuenciados.

Fue el primer método que permitió obtener miles de SNP distribuidos por todo el genoma de forma reproducible.

---

## ddRADseq

Publicado por **Peterson et al. (2012)**.

Reemplaza la fragmentación mecánica por una segunda enzima de restricción. Solo se secuencian los fragmentos delimitados por ambas enzimas y que cumplen un rango de tamaño determinado. Actualmente es una de las variantes más utilizadas debido a su alta reproducibilidad y bajo costo.

---

## GBS (Genotyping-by-Sequencing)

Desarrollado como una versión más simple y económica de RADseq.

Utiliza una única enzima de restricción y elimina la selección física del tamaño. La PCR favorece naturalmente los fragmentos más cortos, reduciendo la complejidad del genoma sin pasos adicionales.

Es ampliamente utilizado en programas de mejoramiento genético.

---

## 2bRAD

Emplea enzimas de restricción tipo IIB, las cuales cortan a ambos lados de su sitio de reconocimiento, generando fragmentos de longitud uniforme (33–36 pb).

Esta uniformidad simplifica considerablemente el análisis bioinformático.

---

## ezRAD

Diseñado para facilitar la implementación de RADseq en organismos no modelo.

Utiliza una o varias enzimas de restricción, pero emplea kits comerciales estándar de preparación de bibliotecas Illumina, evitando la necesidad de adaptadores especializados.

---

## MSG (Multiplexed Shotgun Genotyping)

Método orientado al genotipado masivo y la construcción de mapas genéticos. Simplifica varios pasos del protocolo original y permite procesar numerosos individuos simultáneamente.

---

## CRoPS (Complexity Reduction of Polymorphic Sequences)

Utiliza dos enzimas de restricción para reducir la complejidad del genoma y fue desarrollado inicialmente para la plataforma de secuenciación 454. Representó uno de los primeros enfoques de doble digestión.

---

## SBG (Sequence-Based Genotyping)

Emplea una combinación de enzimas de corte frecuente y corte raro para controlar con mayor precisión el número de loci obtenidos, disminuyendo aún más la complejidad del genoma.

---

# Módulo 3 – Datos de Representación Reducida y Diversidad Genética

**Autor:** Andrey Sánchez  
**Programa:** Maestría en Ciencias – Biología, Universidad Nacional de Colombia  
**Afiliación:** Herbario Nacional Colombiano (COL)  
**Fecha:** Julio 2026

---

# Introducción

La secuenciación de ADN asociada a sitios de restricción (**RAD-seq**) fue el primer método que permitió obtener miles de SNPs distribuidos por todo el genoma de forma reproducible y a bajo costo (Baird et al., 2008). Su principio se basa en el corte del ADN genómico con enzimas de restricción, seguido de la secuenciación de los fragmentos adyacentes a los sitios de corte, generando un muestreo sistemático del genoma.

La variante **ddRADseq** (*double digest* RADseq; Peterson et al., 2012) reemplaza la fragmentación mecánica por una segunda enzima de restricción. Solo se secuencian los fragmentos delimitados por ambas enzimas que cumplen un rango de tamaño determinado, lo que aumenta la reproducibilidad entre muestras y reduce el costo. Esta variante es actualmente una de las más utilizadas en estudios de genómica de poblaciones.

En este módulo se simularon datos ddRADseq a partir de un fragmento del genoma de referencia de *Solanum lycopersicum* Micro-Tom (cromosoma 6, primeros 10 Mb; AP028940.1; GCF_036512215.1 SLM_r2.1), usando RADinitio. Las lecturas simuladas fueron procesadas con ipyrad para el ensamblaje de loci y la identificación de variantes. Finalmente, se estimaron estadísticas de diversidad genética —diversidad nucleotídica (π), heterocigosidad observada y diferenciación genética (F_ST)— con VCFtools.

---

# Métodos

## Entorno computacional

Todos los análisis se realizaron en Ubuntu 24 bajo WSL2 (Windows Subsystem for Linux) en un equipo local con procesador Intel Core i5 y 8 GB de RAM, usando el gestor de paquetes conda (Miniforge3).

```bash
# Creación del ambiente conda
conda create -n radseq2 python=3.10 -y
conda activate radseq2

# Instalación de herramientas
conda install -c bioconda -c conda-forge ipyrad samtools vcftools bbtools -y
pip install setuptools==69.0.0
pip install radinitio
```

| Herramienta | Versión |
|-------------|---------|
| RADinitio | 1.2.3 |
| msprime | 1.4.0 |
| ipyrad | 0.9.108 |
| BBtools | — |
| VCFtools | 0.1.17 |
| samtools | — |
| Python | 3.10 |
| Sistema operativo | Ubuntu 24 (WSL2) |

---

## 1. Obtención y preparación del genoma de referencia

Se descargó la secuencia del cromosoma 6 de *Solanum lycopersicum* Micro-Tom (AP028940.1; 52,172,941 pb) desde NCBI en formato FASTA. Para reducir el tiempo de cómputo, se extrajo un fragmento de los primeros 10 Mb con `samtools faidx`:

```bash
# Indexar el genoma
samtools faidx chr6_SLM.fasta

# Extraer los primeros 10 Mb
samtools faidx chr6_SLM.fasta "AP028940.1:1-10000000" > chr6_10Mb.fasta

# Verificar el fragmento
ls -lh chr6_10Mb.fasta
# -rw-r--r-- 1 user 9.7M chr6_10Mb.fasta
```

El fragmento resultante tiene un tamaño de **9.7 Mb**.

---

## 2. Simulación de datos ddRADseq con RADinitio

RADinitio v1.2.3 simula el proceso completo de preparación de una biblioteca ddRADseq a partir de un genoma de referencia. Internamente utiliza `msprime` para simular la historia demográfica y la variación genética bajo un modelo coalescente, y luego simula la digestión enzimática, la selección de fragmentos por tamaño, la amplificación por PCR y la secuenciación.

### 2.1 Preparación de archivos de entrada

```bash
# Crear lista de cromosomas/scaffolds del fragmento
awk 'sub(/^>/, "")' chr6_10Mb.fasta | head -1 > chrom_list.txt
cat chrom_list.txt
# AP028940.1:1-10000000

# Crear directorio de salida
mkdir sim_reads
```

### 2.2 Ejecución de la simulación

```bash
radinitio --simulate-all \
    --genome chr6_10Mb.fasta \          # Genoma de referencia
    --chromosomes chrom_list.txt \       # Lista de cromosomas a simular
    --out-dir sim_reads \                # Directorio de salida
    --n-pops 3 \                         # Número de poblaciones
    --pop-eff-size 1000 \                # Tamaño efectivo poblacional (Ne)
    --n-seq-indv 5 \                     # Individuos por población
    --library-type ddRAD \               # Tipo de biblioteca
    --enz ecoRI \                        # Enzima de restricción 1
    --enz2 mseI \                        # Enzima de restricción 2
    --insert-mean 350 \                  # Tamaño medio de inserto (pb)
    --insert-stdev 35 \                  # Desviación estándar del inserto
    --pcr-cycles 9 \                     # Ciclos de PCR
    --coverage 10 \                      # Cobertura objetivo (X)
    --read-length 150                    # Longitud de lectura (pb)
```

**Parámetros del modelo demográfico (msprime):**

| Parámetro | Valor |
|-----------|-------|
| Tasa de mutación | 7 × 10⁻⁸ |
| Número de poblaciones | 3 |
| Ne por población | 1,000 |
| Individuos por población | 5 |
| Tasa de migración | 5 × 10⁻⁴ (simétrica entre todas las poblaciones) |
| Modelo de sustitución | equal (Jukes-Cantor) |
| Probabilidad de INDEL | 0.01 |

**Parámetros de la biblioteca ddRAD:**

| Parámetro | Valor |
|-----------|-------|
| Enzima 1 | EcoRI (G/AATTC) |
| Enzima 2 | MseI (T/TAA) |
| Tamaño de inserto (media ± SD) | 350 ± 35 pb |
| Rango de tamaño seleccionado | 280–420 pb |
| Longitud de lectura | 150 pb |
| Cobertura | 10X |
| Longitud base del locus | 1,000 pb |
| Distancia mínima entre loci | 1,000 pb |

### 2.3 Conversión de FASTA a FASTQ con BBtools

ipyrad requiere lecturas en formato FASTQ. Los archivos simulados por RADinitio están en formato FASTA comprimido y se convierten con `reformat.sh` de BBtools:

```bash
mkdir ddrad_fq
cd sim_reads/rad_reads/

# Loop de conversión para todas las muestras (R1 y R2)
for i in msp_*.1.fa.gz; do
    reformat.sh \
        in=${i//.1./.#.} \
        out=../../ddrad_fq/${i//.1.fa.gz/_R#_.fastq.gz}
done
```

Este loop genera pares de archivos FASTQ (R1 y R2) para cada uno de los 15 individuos simulados, resultando en **30 archivos FASTQ**.

---

## 3. Procesamiento de lecturas con ipyrad

ipyrad v0.9.108 es una plataforma para el ensamblaje de loci RAD y la identificación de variantes. El flujo de trabajo consta de 7 pasos: (1) carga de datos, (2) filtrado y trimming, (3) clustering/mapeo intramuestral, (4) estimación de tasa de error y heterocigosidad, (5) llamado de bases, (6) clustering intermuestral, y (7) filtrado y formateo de salida.

### 3.1 Creación del archivo de parámetros

```bash
cd ~/radseq_practica
ipyrad -n sim_data
# Genera: params-sim_data.txt
```

### 3.2 Configuración de parámetros

El archivo `params-sim_data.txt` fue editado con los siguientes valores:

```
sim_data                                              ## [0]  assembly_name
/home/user/radseq_practica/proc_reads                 ## [1]  project_dir
                                                      ## [2]  raw_fastq_path
                                                      ## [3]  barcodes_path
/home/user/radseq_practica/ddrad_fq/*.fastq.gz        ## [4]  sorted_fastq_path
denovo                                                ## [5]  assembly_method
                                                      ## [6]  reference_sequence
pairddrad                                             ## [7]  datatype
AATTC,TAA                                             ## [8]  restriction_overhang
5                                                     ## [9]  max_low_qual_bases
33                                                    ## [10] phred_Qscore_offset
6                                                     ## [11] mindepth_statistical
6                                                     ## [12] mindepth_majrule
10000                                                 ## [13] maxdepth
0.85                                                  ## [14] clust_threshold
0                                                     ## [15] max_barcode_mismatch
2                                                     ## [16] filter_adapters
35                                                    ## [17] filter_min_trim_len
2                                                     ## [18] max_alleles_consens
0.05                                                  ## [19] max_Ns_consens
0.05                                                  ## [20] max_Hs_consens
4                                                     ## [21] min_samples_locus
0.2                                                   ## [22] max_SNPs_locus
8                                                     ## [23] max_Indels_locus
0.5                                                   ## [24] max_shared_Hs_locus
0, 0, 0, 0                                            ## [25] trim_reads
0, 0, 0, 0                                            ## [26] trim_loci
v                                                     ## [27] output_formats (VCF)
```

### 3.3 Ejecución del pipeline completo

```bash
# Correr los 7 pasos con 4 núcleos
ipyrad -p params-sim_data.txt -s 1234567 -c 4
```

---

## 4. Filtrado de SNPs con VCFtools

El VCF generado por ipyrad fue filtrado para eliminar INDELs y aplicar umbrales de calidad y profundidad de cobertura:

```bash
vcftools --vcf ~/radseq_practica/proc_reads/sim_data_outfiles/sim_data.vcf \
    --remove-indels \        # Eliminar inserciones y deleciones
    --max-missing 0.8 \      # Máximo 20% de datos faltantes por sitio
    --min-meanDP 6 \         # Profundidad media mínima de 6X
    --max-meanDP 100 \       # Profundidad media máxima de 100X
    --recode \               # Generar VCF filtrado
    --out ~/radseq_practica/sim_data_filtered
```

---

## 5. Estimación de estadísticas de diversidad genética

Se definieron tres grupos poblacionales a partir del archivo `popmap.tsv` generado por RADinitio:

```bash
# Crear archivos de población para VCFtools
grep "pop0" sim_reads/popmap.tsv | cut -f1 > pop0.txt  # msp_00 a msp_04
grep "pop1" sim_reads/popmap.tsv | cut -f1 > pop1.txt  # msp_05 a msp_09
grep "pop2" sim_reads/popmap.tsv | cut -f1 > pop2.txt  # msp_10 a msp_14
```

### 5.1 Diversidad nucleotídica (π) por sitio

```bash
# π por sitio para cada población
for pop in pop0 pop1 pop2; do
    vcftools --vcf sim_data_filtered.recode.vcf \
        --keep ${pop}.txt \
        --site-pi \
        --out pi_${pop}
done

# Calcular π promedio por población
awk 'NR>1 {sum+=$3; n++} END {print "pi_pop0 =", sum/n}' pi_pop0.sites.pi
awk 'NR>1 {sum+=$3; n++} END {print "pi_pop1 =", sum/n}' pi_pop1.sites.pi
awk 'NR>1 {sum+=$3; n++} END {print "pi_pop2 =", sum/n}' pi_pop2.sites.pi
```

### 5.2 Heterocigosidad observada

```bash
vcftools --vcf sim_data_filtered.recode.vcf \
    --het \
    --out heterocigosidad
# Salida: heterocigosidad.het (columnas: INDV, O(HOM), E(HOM), N_SITES, F)
```

### 5.3 Diferenciación genética (F_ST de Weir y Cockerham)

```bash
# Comparación pop0 vs pop1
vcftools --vcf sim_data_filtered.recode.vcf \
    --weir-fst-pop pop0.txt \
    --weir-fst-pop pop1.txt \
    --out fst_pop0_pop1

# Comparación pop0 vs pop2
vcftools --vcf sim_data_filtered.recode.vcf \
    --weir-fst-pop pop0.txt \
    --weir-fst-pop pop2.txt \
    --out fst_pop0_pop2

# Comparación pop1 vs pop2
vcftools --vcf sim_data_filtered.recode.vcf \
    --weir-fst-pop pop1.txt \
    --weir-fst-pop pop2.txt \
    --out fst_pop1_pop2
```

---

# Resultados

## Simulación de datos RADseq

RADinitio identificó **4,056 loci EcoRI-MseI** en el fragmento de 10 Mb dentro del rango de tamaño objetivo (280–420 pb). Tras eliminar loci en proximidad (<1,000 pb entre sí), se retuvieron **2,788 loci** para la simulación. El modelo coalescente generó **36,684 variantes** distribuidas en las 15 muestras.

## Ensamblaje de loci con ipyrad

| Filtro | Loci eliminados | Loci retenidos |
|--------|----------------|----------------|
| Total prefiltered | — | 2,807 |
| Duplicados eliminados | 48 | 2,759 |
| Max INDELs | 400 | 2,359 |
| Max SNPs por locus | 1 | 2,359 |
| Max heterocigotos compartidos | 67 | 2,306 |
| Min muestras por locus (≥4) | 469 | **1,837** |

La cobertura por muestra fue de ~1,330–1,340 loci retenidos por individuo.

## Filtrado de SNPs

| Parámetro | Valor |
|-----------|-------|
| SNPs antes del filtrado | 1,006 |
| SNPs después del filtrado | **561** |
| Individuos retenidos | 15/15 |
| Criterios aplicados | Sin INDELs, max-missing 0.8, DP 6–100X |

## Diversidad nucleotídica (π)

| Población | Individuos | π promedio |
|-----------|-----------|-----------|
| pop0 | 5 | 0.1964 |
| pop1 | 5 | 0.2386 |
| pop2 | 5 | 0.2281 |

## Heterocigosidad observada

| Individuo | Población | O(HOM) | E(HOM) | N_sitios | F |
|-----------|-----------|--------|--------|----------|---|
| msp_00 | pop0 | 427 | 382.6 | 516 | 0.333 |
| msp_01 | pop0 | 386 | 361.9 | 486 | 0.194 |
| msp_02 | pop0 | 441 | 384.0 | 516 | 0.432 |
| msp_03 | pop0 | 404 | 364.0 | 490 | 0.318 |
| msp_04 | pop0 | 439 | 393.7 | 531 | 0.330 |
| msp_05 | pop1 | 388 | 368.2 | 496 | 0.155 |
| msp_06 | pop1 | 426 | 391.9 | 528 | 0.250 |
| msp_07 | pop1 | 393 | 370.0 | 498 | 0.180 |
| msp_08 | pop1 | 400 | 366.6 | 492 | 0.267 |
| msp_09 | pop1 | 402 | 375.2 | 503 | 0.210 |
| msp_10 | pop2 | 404 | 377.2 | 503 | 0.213 |
| msp_11 | pop2 | 424 | 384.4 | 517 | 0.299 |
| msp_12 | pop2 | 423 | 376.9 | 509 | 0.349 |
| msp_13 | pop2 | 458 | 397.9 | 536 | 0.435 |
| msp_14 | pop2 | 433 | 391.2 | 525 | 0.312 |

> **F** = coeficiente de endogamia de Wright. Valores positivos indican exceso de homocigotos respecto a lo esperado bajo equilibrio Hardy-Weinberg.

## Diferenciación genética (F_ST de Weir y Cockerham)

| Comparación | F_ST medio | F_ST ponderado |
|-------------|-----------|----------------|
| pop0 vs pop1 | 0.084 | 0.152 |
| pop0 vs pop2 | 0.118 | 0.220 |
| pop1 vs pop2 | 0.080 | 0.143 |

> El **F_ST medio** promedia todos los sitios incluyendo los no informativos; el **F_ST ponderado** da mayor peso a los sitios con mayor variación y es el estimador recomendado para comparaciones entre poblaciones.

---

# Discusión

## Loci RAD y filtrado

El pipeline recuperó 1,837 loci de los 2,788 simulados. La mayor fuente de pérdida fue el filtro de mínimo de muestras por locus (`min_samples_locus = 4`), que eliminó 469 loci. Esto es esperable: en datos simulados con un modelo de migración simétrica moderada, algunos loci solo son polimórficos en un subconjunto de poblaciones y no alcanzan el umbral de cobertura mínima en todas las muestras. Este patrón es consistente con lo reportado por Eaton & Overcast (2020) para datasets ddRAD simulados.

Tras el filtrado con VCFtools, se retuvieron 561 SNPs de 1,006 sitios variables (55.8%), principalmente por el criterio de datos faltantes (`max-missing 0.8`). Este porcentaje de retención es razonable para datos RAD de baja cobertura (10X).

## Diversidad nucleotídica

Los valores de π oscilaron entre 0.196 (pop0) y 0.239 (pop1). Bajo el modelo neutral de Jukes-Cantor con Ne = 1,000 y tasa de mutación μ = 7 × 10⁻⁸, la diversidad nucleotídica esperada es:

> θ = 4 × Ne × μ = 4 × 1,000 × 7×10⁻⁸ = **2.8 × 10⁻⁴**

Los valores observados son más altos porque se calcularon solo sobre los sitios polimórficos identificados por el protocolo RAD (561 SNPs), no sobre la totalidad de sitios del genoma. El π calculado sobre sitios variables sobreestima el π genómico real. En un análisis real con pixy sobre un VCF allsites (con sitios invariantes incluidos), el denominador incluiría todos los sitios y el estimador sería insesgado.

## Diferenciación genética

Los valores de F_ST ponderado (0.143–0.220) indican diferenciación moderada entre las tres poblaciones, coherente con el modelo de migración simétrica utilizado (m = 5 × 10⁻⁴). Bajo el modelo de isla de Wright:

> F_ST ≈ 1 / (1 + 4×Ne×m) = 1 / (1 + 4×1000×5×10⁻⁴) = 1 / 3 ≈ **0.333**

Los valores observados (0.143–0.220) son menores que la predicción teórica, lo que puede deberse al tamaño pequeño de muestra (n=5 por población) y al uso de un fragmento reducido del genoma. La comparación pop0 vs pop2 muestra la mayor diferenciación (F_ST = 0.220), mientras que pop1 vs pop2 muestra la menor (F_ST = 0.143), lo que podría reflejar diferencias estocásticas en la historia genealógica de los loci muestreados dado el Ne reducido.

## Limitaciones

- El análisis se realizó sobre un fragmento de 10 Mb del cromosoma 6, lo que limita el número de loci RAD y reduce la precisión de los estimadores poblacionales.
- El tamaño muestral reducido (n=5 por población) introduce alta varianza en los estimadores de π y F_ST.
- El cálculo de π con VCFtools sobre sitios variables únicamente produce estimaciones sesgadas hacia arriba. Para estimaciones insesgadas en datos WGS se recomienda usar pixy con un VCF allsites.

---

# Referencias

- Andrews, K. R., Good, J. M., Miller, M. R., Luikart, G., & Hohenlohe, P. A. (2016). Harnessing the power of RADseq for ecological and evolutionary genomics. *Nature Reviews Genetics*, 17(2), 81–92.
- Baird, N. A., et al. (2008). Rapid SNP discovery and genetic mapping using sequenced RAD markers. *PLOS ONE*, 3(10), e3376.
- Eaton, D. A. R., & Overcast, I. (2020). ipyrad: Interactive assembly and analysis of RADseq datasets. *Bioinformatics*, 36(8), 2592–2594.
- Kelleher, J., Etheridge, A. M., & McVean, G. (2016). Efficient coalescent simulation and genealogical analysis for large sample sizes. *PLOS Computational Biology*, 12(5), e1004842.
- Peterson, B. K., Weber, J. N., Kay, E. H., Fisher, H. S., & Hoekstra, H. E. (2012). Double digest RADseq: An inexpensive method for de novo SNP discovery and genotyping in model and non-model species. *PLOS ONE*, 7(5), e37135.
- Takei, H., et al. (2021). De novo genome assembly of two tomato ancestors, *Solanum pimpinellifolium* and *Solanum lycopersicum* var. *cerasiforme*, by long-read sequencing. *DNA Research*, 28(1), dsaa029.
- Weir, B. S., & Cockerham, C. C. (1984). Estimating F-statistics for the analysis of population structure. *Evolution*, 38(6), 1358–1370.


# Referencias

- Baird NA et al. (2008). *Rapid SNP Discovery and Genetic Mapping Using Sequenced RAD Markers.*
- Elshire RJ et al. (2011). *A Robust, Simple Genotyping-by-Sequencing (GBS) Approach.*
- Peterson BK et al. (2012). *Double Digest RADseq.*
- Andrews KR et al. (2016). *Harnessing the power of RADseq for ecological and evolutionary genomics.*
