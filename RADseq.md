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

# Referencias

- Baird NA et al. (2008). *Rapid SNP Discovery and Genetic Mapping Using Sequenced RAD Markers.*
- Elshire RJ et al. (2011). *A Robust, Simple Genotyping-by-Sequencing (GBS) Approach.*
- Peterson BK et al. (2012). *Double Digest RADseq.*
- Andrews KR et al. (2016). *Harnessing the power of RADseq for ecological and evolutionary genomics.*
