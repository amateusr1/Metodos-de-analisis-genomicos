# 🧹 Exploración de calidad, filtrado y recorte de lecturas

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

# 🔍 Evaluación de la calidad de las lecturas con FastQC

## El formato FASTQ

Los datos crudos de secuenciación generados por plataformas de secuenciación como Illumina y otras se almacenan en archivos con formato **FASTQ**, el cual contiene tanto la secuencia de nucleótidos como la calidad asociada a cada base.

Cada lectura está representada por **cuatro líneas**:

```text
@SRR31477438.1
GATCGATCGATCGATCGATCG...
+
&&&%%%@@@@@@@@@@@@@@@...
```

| Línea | Contenido |
|--------|-----------|
| **1** | Identificador único de la lectura (comienza con `@`). |
| **2** | Secuencia de nucleótidos (A, T, G, C y N). |
| **3** | Separador (`+`), que puede repetir opcionalmente el identificador. |
| **4** | Calidad Phred de cada base codificada mediante caracteres ASCII. |

La cuarta línea contiene una puntuación de calidad para cada nucleótido de la secuencia. Estas puntuaciones indican la probabilidad de que una base haya sido identificada incorrectamente durante la secuenciación.

---

# Inspeccionar un archivo FASTQ comprimido

Los archivos FASTQ suelen distribuirse comprimidos en formato **gzip** (`.fastq.gz`) para reducir el espacio de almacenamiento. Debido a ello, no pueden visualizarse directamente con comandos como `cat` o `head`.

Para inspeccionar su contenido se utiliza `zcat`, que descomprime el archivo temporalmente y envía la salida a otros comandos.

Visualizar la primera lectura:

```bash
zcat SRR31477438_R1.fastq.gz | head -4
```

---

# Contar el número de lecturas

Cada lectura ocupa exactamente **cuatro líneas**, por lo que el número total de secuencias puede obtenerse dividiendo el número total de líneas entre cuatro.

Contar las líneas del archivo:

```bash
zcat SRR31477438_R1.fastq.gz | wc -l
```

Otra alternativa consiste en contar únicamente las líneas correspondientes a los encabezados de las lecturas.

Por ejemplo:

```bash
zgrep -c "^@" SRR31477438_R1.fastq.gz
```

Sin embargo, este método no siempre es recomendable, ya que el carácter `@` también puede aparecer en la línea de calidad debido a la codificación ASCII utilizada por el formato FASTQ. Por esta razón, el conteo basado únicamente en `@` puede sobreestimar el número real de secuencias.

---

### ¿Qué es ASCII?

ASCII (*American Standard Code for Information Interchange*) es un estándar que asigna un valor numérico a cada carácter utilizado por un computador. Por ejemplo:

| Carácter | Código ASCII |
|----------|-------------:|
| `!` | 33 |
| `"` | 34 |
| `#` | 35 |
| `$` | 36 |
| `%` | 37 |
| `A` | 65 |
| `F` | 70 |
| `I` | 73 |
| `J` | 74 |

En un archivo FASTQ, estos caracteres **no representan letras**, sino valores de calidad para cada nucleótido. La calidad de una base se expresa mediante una **puntuación Phred (Q)**, que estima la probabilidad de que dicha base haya sido identificada incorrectamente durante la secuenciación.

Por ejemplo:

| Calidad (Q) | Probabilidad de error | Precisión |
|------------:|----------------------:|----------:|
| 10 | 1 en 10 | 90 % |
| 20 | 1 en 100 | 99 % |
| 30 | 1 en 1000 | 99.9 % |
| 40 | 1 en 10000 | 99.99 % |

En bioinformática, generalmente se considera que una base con **Q ≥ 30** posee una calidad muy alta.

---

## ¿Por qué usar caracteres y no números?

Guardar millones de puntuaciones de calidad como números ocuparía mucho espacio. Por ello, el formato FASTQ almacena cada valor utilizando un único carácter ASCII. En los archivos FASTQ modernos (Illumina 1.8+), la conversión se realiza mediante la codificación **Phred+33**.

---

## ¿Cómo interpreta FastQC estas puntuaciones?

FastQC convierte automáticamente los caracteres ASCII en puntuaciones Phred y calcula diferentes métricas de calidad, entre ellas:

- Calidad promedio por posición de la lectura.
- Calidad promedio por secuencia.
- Distribución de las puntuaciones Phred.
- Identificación de regiones con baja calidad.

Estos resultados permiten determinar si las lecturas requieren un proceso de limpieza antes del ensamblaje o del alineamiento contra un genoma de referencia.

---

# ¿Qué es FastQC?

**FastQC** es una de las herramientas más utilizadas para realizar el control de calidad de datos de secuenciación masiva (NGS).

Su objetivo es identificar posibles problemas presentes en los archivos FASTQ antes de iniciar cualquier análisis bioinformático. Entre los análisis realizados por FastQC se encuentran:

- Calidad por posición de la lectura.
- Calidad promedio por secuencia.
- Distribución de longitudes.
- Contenido GC.
- Contenido de bases por posición.
- Niveles de duplicación.
- Secuencias sobre representadas.
- Presencia de adaptadores.
- Distribución de puntuaciones Phred.

---

# Consultar la ayuda del programa

Para visualizar todas las opciones disponibles:

```bash
fastqc --help
```

o

```bash
fastqc -h
```

---

# Ejecutar FastQC sobre una muestra

El análisis puede realizarse indicando simplemente el archivo FASTQ como argumento.

```bash
fastqc SRR31477438_R1.fastq.gz
```

El programa acepta directamente archivos comprimidos (`.fastq.gz`), por lo que no es necesario descomprimirlos previamente.

---

# Analizar lecturas pareadas

Cuando se trabaja con datos *paired-end*, es recomendable analizar ambos archivos de forma independiente.

```bash
fastqc \
SRR31477438_R1.fastq.gz \
SRR31477438_R2.fastq.gz
```

---

# Especificar un directorio de salida

Los reportes generados por FastQC pueden almacenarse en una carpeta específica mediante la opción `-o`.

```bash
mkdir FastQC

fastqc \
-o FastQC \
SRR31477438_R1.fastq.gz \
SRR31477438_R2.fastq.gz
```

---

# Analizar múltiples muestras

Cuando un proyecto contiene numerosas muestras, no es necesario ejecutar FastQC individualmente para cada archivo.

Mediante el uso de comodines (`*`) es posible analizar todos los archivos FASTQ de un directorio en un único comando.

```bash
mkdir FastQC

fastqc \
-o FastQC \
*.fastq.gz
```

o indicando la ruta completa

```bash
fastqc \
-o FastQC \
/ruta/al/directorio/*.fastq.gz
```

---

# Ejemplo de ejecución en un clúster HPC

```bash
#!/bin/bash

#SBATCH --job-name=FastQC
#SBATCH --partition=cpu.cecc
#SBATCH --clusters=biocomputo
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --output=FastQC_%j.out
#SBATCH --error=FastQC_%j.err

module load envs/anaconda3

source $(conda info --base)/etc/profile.d/conda.sh

conda activate fastqc

mkdir -p FastQC

fastqc \
*.fastq.gz \
-o FastQC \
-t 8
```

---

# Resultados generados

Por cada archivo analizado, FastQC genera dos archivos:

```
SRR31477438_R1_fastqc.html

SRR31477438_R1_fastqc.zip
```

- **HTML:** reporte interactivo para inspección visual.
- **ZIP:** contiene todos los datos utilizados para generar el reporte.

# Visualizar los reportes

FastQC genera un reporte en formato **HTML**, que puede abrirse directamente desde cualquier navegador web. Cuando el análisis se ejecuta en un clúster HPC, lo más práctico es descargar únicamente los archivos `.html`, ya que contienen toda la información necesaria para interpretar los resultados.

La transferencia puede realizarse mediante herramientas como `scp` o `rsync`. LA DESCARGA SE REALIZA DESDE SU TERMINAL LOCAL NO DESDE EL INTERIOR DEL CLUSTER.

Ejemplo utilizando `scp`:

```bash
 scp -J amateusr@168.176.8.19 "amateusr@hercules2:/scratchsan/amateusr/outs*_fastqc.html" "C:\Users\ASUS\Downloads\"
```

Una vez descargados, basta con abrir los archivos en el navegador de su computador haciendo doble clic sobre ellos.

<img width="838" height="403" alt="image" src="https://github.com/user-attachments/assets/cec61738-5bef-423c-a337-c364e04b0541" />

---

# Resumen de resultados con MultiQC

Cuando se analizan múltiples muestras resulta poco práctico revisar cada reporte individualmente.

**MultiQC** recopila automáticamente todos los resultados generados por FastQC y produce un único informe interactivo que resume la calidad de todas las muestras del proyecto.

Activar el ambiente:

```bash
module load envs/anaconda3

source $(conda info --base)/etc/profile.d/conda.sh

conda activate multiqc
```

Ejecutar MultiQC: El punto al final de la linea le dice al programa lea todos los .html en el directorio y unifiquelos.

```bash
cd FastQC

multiqc .
```

Como resultado se genera un archivo

```
multiqc_report.html
```

que resume todos los reportes individuales de FastQC en una única página.

---

# Interpretación de los resultados

Al inspeccionar los reportes de FastQC es recomendable prestar especial atención a los siguientes indicadores:

- **Per base sequence quality:** calidad de las bases a lo largo de cada lectura.
- **Per sequence quality scores:** distribución de la calidad promedio por lectura.
- **Per base sequence content:** proporción de A, T, G y C en cada posición.
- **Per sequence GC content:** distribución del contenido GC de las secuencias.
- **Adapter content:** detección de adaptadores residuales.
- **Overrepresented sequences:** identificación de secuencias excesivamente abundantes.
- **Sequence duplication levels:** evaluación del nivel de duplicación de las lecturas.

No todos los módulos marcados con **Warning** o **Fail** representan necesariamente un problema. La interpretación de estos indicadores depende del tipo de experimento (WGS, RNA-seq, RADseq, amplicones, metagenómica, entre otros) y del protocolo de preparación de bibliotecas utilizado.

---

# Buenas prácticas

- Ejecutar FastQC inmediatamente después de descargar las lecturas.
- Repetir el análisis una vez finalizada la limpieza de los datos.
- Comparar los reportes antes y después del filtrado para verificar la eliminación de adaptadores y la mejora en la calidad de las lecturas.
- Utilizar MultiQC cuando se analicen múltiples muestras para facilitar la comparación entre ellas.

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

for sample in SRR31477438 SRR38359005 SRR37254991
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

  # 🧹 Limpieza de lecturas con Captus

Además de **Trimmomatic**, **Captus** incorpora un módulo denominado **Clean**, diseñado para automatizar el preprocesamiento de lecturas. Este módulo integra varias etapas de limpieza en un único comando, incluyendo la eliminación de adaptadores, el filtrado de bases de baja calidad, la remoción de contaminantes (como PhiX) y la generación de reportes de calidad.

Una ventaja de Captus es que reconoce automáticamente las lecturas pareadas, siempre que los archivos sigan una convención de nombres adecuada.

## Convención de nombres

Captus utiliza el nombre de los archivos para identificar las muestras y emparejar correctamente las lecturas **R1** y **R2**. Por ello, es recomendable asignar nombres claros y consistentes antes de iniciar cualquier análisis.

### Recomendaciones

- Utilizar únicamente letras, números, guiones (`-`) y guiones bajos (`_`).
- Evitar espacios, caracteres especiales (`!`, `#`, `%`, `@`, etc.) y letras con acentos.
- No utilizar dobles guiones bajos (`__`), ya que Captus los emplea internamente durante el procesamiento.
- Las lecturas pareadas deben contener obligatoriamente los identificadores **`_R1`** y **`_R2`**.
- Para datos *single-end* también debe utilizarse el sufijo **`_R1`**.
- Utilizar extensiones válidas como `.fastq`, `.fastq.gz`, `.fq` o `.fq.gz`.

### Ejemplos de nombres válidos

```text
Brosimum_alicastrum_COL01_R1.fastq.gz
Brosimum_alicastrum_COL01_R2.fastq.gz
```

### Ejemplos de nombres no recomendados

```text
ERR246535_1.fastq.gz
ERR246535_2.fastq.gz
```
## Ejecutar Captus Clean

### Script SLURM

```bash

module load envs/anaconda3
source $(conda info --base)/etc/profile.d/conda.sh

conda activate captus_1.6.5

captus clean \
    -r *.fastq.gz \ 
    -o clean \
    --threads 16
```
El único argumento obligatorio es la ubicación de las lecturas crudas.

```bash
captus clean \
    --reads ./00_raw_reads/*_CAP_R?.fq.gz
```
---

## Explicación de los parámetros

| Parámetro | Descripción |
|-----------|-------------|
| `clean` | Ejecuta el módulo de limpieza de Captus. |
| `-r *.fastq.gz` | Selecciona todas las lecturas FASTQ presentes en el directorio. Captus identifica automáticamente los pares `R1` y `R2` a partir del nombre de los archivos. |
| `-o clean` | Directorio donde se almacenarán las lecturas limpias y los reportes. |
| `--threads 16` | Número de núcleos utilizados durante el procesamiento. |

---

## Estructura de salida

Al finalizar la ejecución, Captus crea un directorio denominado `01_clean_reads`, que contiene tanto las lecturas limpias como los reportes generados durante el proceso.

| Archivo o directorio | Descripción |
|-----------------------|-------------|
| **00_adaptors_trimmed/** | Archivos intermedios tras la eliminación de adaptadores. |
| **Sample_R1.fq.gz / Sample_R2.fq.gz** | Lecturas limpias que serán utilizadas en los análisis posteriores. |
| **Sample.cleaning.log** | Registro detallado de la limpieza realizada para cada muestra. |
| **Sample.cleaning.stats.txt** | Resumen de los contaminantes detectados y eliminados durante el filtrado. |
| **01_qc_stats_before/** | Resultados de FastQC/Falco antes de la limpieza. |
| **02_qc_stats_after/** | Resultados de FastQC/Falco después de la limpieza. |
| **03_qc_extras/** | Tablas con estadísticas detalladas utilizadas para construir el reporte final. |
| **captus-clean_report.html** | Reporte interactivo que resume la calidad de todas las muestras procesadas. |
| **captus-clean.log** | Registro general de la ejecución de Captus. |

---

# Discusión

Para las tres muestras de Solanum lycopersicum utilizadas en este curso se realizó una evaluación de calidad antes y después del proceso de limpieza utilizando FastQC y la integración de resultados mediante MultiQC. Los reportes iniciales permitieron identificar el estado general de las lecturas crudas, mientras que los reportes posteriores confirmaron que la configuración de Trimmomatic empleada (ILLUMINACLIP:TruSeq3-PE-2.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:21) eliminó de forma efectiva los adaptadores y las regiones de baja calidad sin comprometer la cantidad de datos útiles. Tras el filtrado, las muestras presentaron una calidad promedio por base superior a Q30 en prácticamente toda la longitud de las lecturas, ausencia de contaminación significativa por adaptadores, una distribución del contenido GC consistente y un contenido de bases por posición sin sesgos importantes.

<img width="314" height="126" alt="image" src="https://github.com/user-attachments/assets/a5475c9c-ce0a-43f2-a1a1-0c383220dd93" />

Informe para SRR31477438 tras realizada la limpieza.

---

# Próximo módulo

Una vez obtenidas lecturas filtradas, aprenderemos a utilizarlas para **ensamblar genomas plastidiales mediante estrategias *de novo*** o para **alinearlas contra un genoma de referencia**, dependiendo de los objetivos del estudio.
