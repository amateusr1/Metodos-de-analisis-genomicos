# Bienvenid@ a Métodos de Análisis Genómicos

Bienvenid@ a este repositorio, una bitácora de trabajo dedicada al análisis de datos genómicos utilizando herramientas de bioinformática y secuenciación de nueva generación (NGS).

Este proyecto fue creado por **Andrey Mateus-Ruiz** como parte del curso de la Maestría en Ciencias - Biología **Métodos de Análisis de Datos Genómicos (2018846-6)** de la **Universidad Nacional de Colombia**, dictado por el **Ph.D. Gustavo Silva-Arias**.

Más que un conjunto de apuntes, este repositorio busca documentar paso a paso los análisis realizados durante el curso: desde la descarga de datos y el manejo de Linux hasta el ensamblaje de genomas, el llamado de variantes y los análisis de diversidad genética. Cada módulo reúne comandos, explicaciones, resultados, recomendaciones y notas obtenidas durante el proceso de aprendizaje.

> **Este repositorio se encuentra en constante construcción.**
>
> La idea fue llevar un diario técnico del curso, registrando cada flujo de trabajo (SCRIPT) conforme era aprendido y ejecutado, acompañado de un fuerte componente teorico para sentar las bases conceptuales inherentes a cada módulo. Por ello, el contenido continuará creciendo, mejorando y actualizándose con nuevas herramientas, explicaciones y ejemplos.

---

- 🐧 **[01. Introducción a Linux](01-Introducción-a-Linux.md)**  
  Introducción al uso de Linux y de clústeres HPC: comandos básicos, manejo de archivos, SLURM, ambientes, transferencia de archivos, permisos, recomendaciones y herramientas esenciales para el trabajo en bioinformática.

- 📥 **[02. Descarga de datos SRA NCBI](02-Descarga-datos-SRA-NCBI.md)**  
  Descarga de datos públicos de secuenciación desde el Sequence Read Archive (SRA) utilizando SRA Toolkit y organización de los archivos FASTQ.

- 🧹 **[03. Exploración de calidad, filtrado y recorte de lecturas](03-Exploración_de_calidad_filtrado_recorte_de_lecturas.md)**  
  Evaluación de la calidad de las lecturas con FastQC y MultiQC, limpieza de datos utilizando Trimmomatic, Fastp y Captus Clean.

- 🌿 **[04. Ensamblaje de Novo Plastoma](04-Ensamblaje-de-Novo-Plastoma.md)**  
  Ensamblaje *de novo* de genomas plastidiales a partir de lecturas Illumina.

- 🧬 **[05. NGS Data Analysis - Mapping](05-NGS_Data_Analysis_Mapping.md)**  
  Alineamiento de lecturas contra un genoma de referencia, procesamiento de archivos BAM y evaluación de la calidad del mapeo.

- 🔬 **[06. NGS Data Analysis - Variant Calling](06-NGS_Data_Analysis_Variant_Calling.md)**  
  Identificación y filtrado de variantes genómicas mediante GATK, generación de archivos GVCF y genotipificación conjunta.

- 📊 **[07. Estadísticas de diversidad genética - Pixy](07-Estadisticas_diversidad_genética_Pixy)**  
  Cálculo de estadísticas de diversidad genética como π, dXY y FST utilizando Pixy a partir de archivos VCF filtrados.

- 🌎 **[08. NGS Data Analysis - RADseq](08-NGS_Data_Analysis-RADseq.md)**  
  Introducción a los datos de representación reducida (RADseq/GBS), procesamiento de variantes y aplicaciones en genética de poblaciones.

- 💬 **[09. Foro de discusión](09-Foro_discusión.md)**  
  Espacio destinado a preguntas, ejercicios y discusión de conceptos abordados durante el curso.
