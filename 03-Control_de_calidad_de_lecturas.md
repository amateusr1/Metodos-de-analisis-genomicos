# 🧹 Control de calidad y limpieza de lecturas NGS

> **Autor:** Andrey Mateus-Ruiz  
> **Curso:** Métodos de Análisis de Datos Genómicos (2018846-6)  
> **Profesor:** Ph.D. Gustavo Silva-Arias  
> **Universidad Nacional de Colombia**

---

# Descripción

Una vez descargadas las lecturas desde el **Sequence Read Archive (SRA)**, el siguiente paso consiste en evaluar y mejorar su calidad antes de realizar cualquier análisis bioinformático. Aunque las plataformas modernas de secuenciación generan millones de lecturas con alta precisión, es común encontrar problemas como adaptadores residuales, bases de baja calidad, secuencias demasiado cortas, contaminación y otros artefactos que pueden afectar el ensamblaje, el alineamiento contra un genoma de referencia y la identificación de variantes.

Por esta razón, el **control de calidad (Quality Control, QC)** y la **limpieza de lecturas (Read Cleaning)** constituyen una etapa fundamental en prácticamente todos los flujos de trabajo de secuenciación masiva.

En este módmulo aprendereos a inspeccionar la calidad de los datos crudos mediante **FastQC**, eliminar adaptadores y regiones de baja calidad utilizando **Trimmomatic** y el módulo **Clean** de **Captus**, además de conocer algunas de las herramientas disponibles en **BBTools** para la evaluación y procesamiento de lecturas.

> **Nota**
>
> Este documento hace parte de una bitácora técnica en constante construcción. Los procedimientos, recomendaciones y parámetros podrán ampliarse conforme avance el desarrollo del curso.

---

# Objetivos

Al finalizar este módulo podrás:

- Comprender la importancia del control de calidad en datos NGS.
- Interpretar los principales indicadores reportados por FastQC.
- Detectar adaptadores y regiones de baja calidad.
- Eliminar secuencias de baja calidad utilizando Trimmomatic.
- Utilizar el módulo **Clean** de Captus para automatizar el preprocesamiento de lecturas.
- Conocer herramientas complementarias de BBTools para evaluar y limpiar datos de secuenciación.
- Comparar la calidad de las lecturas antes y después del proceso de limpieza.
- Generar archivos FASTQ listos para ensamblaje o alineamiento contra un genoma de referencia.

---


# ¿Por qué es importante esta etapa?

Una limpieza adecuada de las lecturas puede:

- Eliminar adaptadores remanentes.
- Incrementar la calidad del ensamblaje.
- Mejorar el porcentaje de alineamiento.
- Reducir falsos SNPs e INDELs.
- Disminuir el tiempo de ejecución de los análisis posteriores.
- Aumentar la reproducibilidad del pipeline.

---

# 🌿 Trimmomatic

## ¿Qué es Trimmomatic?

**Trimmomatic** es una de las herramientas más utilizadas para el preprocesamiento de lecturas Illumina. Su función principal es eliminar adaptadores, recortar regiones de baja calidad y descartar lecturas demasiado cortas antes del ensamblaje o el alineamiento contra un genoma de referencia.

Realizar esta limpieza mejora significativamente la calidad de los análisis posteriores, disminuyendo errores de ensamblaje y reduciendo la aparición de falsos positivos durante la llamada de variantes.

---

## Cargar el módulo

En el clúster utilizado durante el curso, Trimmomatic se encuentra instalado como un módulo del sistema.

```bash
module load apps/trimmomatic/0.39
```

---

## Script SLURM

```

module load apps/trimmomatic/0.39

cd /scratchsan/amateusr/secuences/

for sample in SRR31477438 SRR38359005
do

java -jar /local64/usr_local/Trimmomatic-0.39/trimmomatic-0.39.jar PE \
    -threads 16 \
    ${sample}_R1.fastq.gz \
    ${sample}_R2.fastq.gz \
    ../clean/${sample}_R1_clean.fastq.gz \
    ../clean/${sample}_R1_unpaired.fastq.gz \
    ../clean/${sample}_R2_clean.fastq.gz \
    ../clean/${sample}_R2_unpaired.fastq.gz \
    ILLUMINACLIP:/local64/usr_local/Trimmomatic-0.39/adapters/TruSeq3-PE-2.fa:2:30:10 \
    LEADING:3 \
    TRAILING:3 \
    SLIDINGWINDOW:4:15 \
    MINLEN:21

done
```

---

# Explicación del script

El script realiza el procesamiento de todas las muestras especificadas en el ciclo `for`.

Para cada muestra:

1. Lee las dos lecturas pareadas (`R1` y `R2`).
2. Elimina adaptadores Illumina.
3. Recorta bases de baja calidad al inicio y final de cada lectura.
4. Evalúa la calidad mediante una ventana deslizante (*sliding window*).
5. Descarta lecturas demasiado cortas.
6. Guarda por separado las lecturas pareadas y las lecturas que perdieron su pareja durante el proceso.

---

# Archivos de entrada

```text
SRR31477438_R1.fastq.gz
SRR31477438_R2.fastq.gz
```

---

# Archivos de salida

```text
clean/

├── SRR31477438_R1_clean.fastq.gz
├── SRR31477438_R1_unpaired.fastq.gz
├── SRR31477438_R2_clean.fastq.gz
└── SRR31477438_R2_unpaired.fastq.gz
```

Las lecturas **_clean.fastq.gz** conservan únicamente los pares que sobrevivieron al filtrado y serán las utilizadas en los análisis posteriores.

Las lecturas **_unpaired.fastq.gz** corresponden a secuencias cuya pareja fue descartada durante la limpieza.

---

# Explicación de los parámetros

| Parámetro | Descripción |
|-----------|-------------|
| `PE` | Procesamiento de lecturas pareadas (*paired-end*). |
| `-threads 16` | Utiliza 16 núcleos del procesador. |
| `ILLUMINACLIP` | Elimina adaptadores Illumina utilizando la secuencia de referencia indicada. |
| `2` | Máximo de dos errores permitidos al buscar adaptadores (*seed mismatches*). |
| `30` | Umbral para eliminar adaptadores mediante alineamiento tipo palíndromo (*palindrome clip threshold*). |
| `10` | Umbral para eliminar adaptadores mediante coincidencia simple (*simple clip threshold*). |
| `LEADING:3` | Elimina bases del inicio con calidad inferior a 3. |
| `TRAILING:3` | Elimina bases del final con calidad inferior a 3. |
| `SLIDINGWINDOW:4:15` | Evalúa ventanas de 4 bases y corta cuando la calidad promedio es menor a 15. |
| `MINLEN:21` | Descarta lecturas cuya longitud final sea menor de 21 pb. |

---

# ¿Por qué se utilizan estos parámetros?

Estos parámetros corresponden a una configuración conservadora ampliamente utilizada para datos de secuenciación Illumina. Su objetivo es eliminar adaptadores y regiones de muy baja calidad, preservando la mayor cantidad posible de información útil sin afectar significativamente la cobertura del experimento.

Los valores óptimos pueden variar dependiendo de la plataforma de secuenciación, la calidad de las lecturas y los objetivos del análisis.

---

# Verificación

Una vez finalizado el proceso es recomendable ejecutar nuevamente **FastQC** sobre los archivos `*_clean.fastq.gz` para comprobar que:

- Los adaptadores fueron eliminados.
- La calidad promedio de las bases aumentó.
- Se redujo la cantidad de secuencias sobre representadas.
- Las lecturas conservadas mantienen una longitud adecuada para los análisis posteriores.

# Próximo módulo

Una vez obtenidas lecturas filtradas, aprenderemos a utilizarlas para **ensamblar genomas plastidiales mediante estrategias *de novo*** o para **alinearlas contra un genoma de referencia**, dependiendo de los objetivos del estudio.
