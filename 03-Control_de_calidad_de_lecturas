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

- Incrementar la calidad del ensamblaje.
- Mejorar el porcentaje de alineamiento.
- Reducir falsos SNPs e INDELs.
- Disminuir el tiempo de ejecución de los análisis posteriores.
- Aumentar la reproducibilidad del pipeline.

---

# Próximo módulo

Una vez obtenidas lecturas filtradas, aprenderemos a utilizarlas para **ensamblar genomas plastidiales mediante estrategias *de novo*** o para **alinearlas contra un genoma de referencia**, dependiendo de los objetivos del estudio.
