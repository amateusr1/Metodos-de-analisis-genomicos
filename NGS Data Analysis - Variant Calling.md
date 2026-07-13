# NGS Data Analysis - Variant Calling

> Una guía para el procesamiento de BAMs y llamada de variantes y un flujo de trabajo completo con GATK
---
# Introducción

El objetivo del llamado de variantes consiste en identificar todas las posiciones del genoma donde las muestras presentan diferencias respecto al genoma de referencia, estas diferencias reciben el nombre de **variantes genéticas**. Las variantes más comunes son: **SNP**, cambio de un único nucleótido; **INDEL** inserción o deleción de uno o varios nucleótidos; **MNP**, sustitución simultánea de varios nucleótidos consecutivos; **Variantes estructurales** incluyen, inversiones, translocaciones, duplicaciones, grandes inserciones, grandes deleciones, estas ultimas generalmente requieren algoritmos especializados distintos de los llamadores convencionales de SNPs.

El llamado de variantes constituye un proceso de inferencia estadística cuyo objetivo es distinguir diferencias genéticas reales de los errores introducidos durante la obtención y procesamiento de los datos de secuenciación. Aunque las plataformas modernas de NGS presentan tasas de error relativamente bajas, ningún experimento de secuenciación está completamente libre de errores. En consecuencia, no toda discrepancia observada entre una lectura y el genoma de referencia corresponde necesariamente a una variante biológica.

Los primeros algoritmos de llamado de variantes utilizaban reglas simples basadas en el conteo de nucleótidos observados en cada posición. Sin embargo, este enfoque resultó insuficiente para analizar regiones con baja cobertura, errores de secuenciación o variantes complejas. Actualmente, prácticamente todos los llamadores modernos emplean modelos probabilísticos basados en inferencia bayesiana, estos modelos integran simultáneamente la información proveniente de: calidad de las bases; calidad del alineamiento; profundidad de cobertura; frecuencia esperada de variantes; distribución de alelos.

Adionalmente el llamado de variantes no solamente identifica la presencia de una mutación, también determina el **genotipo** del individuo, en organismos diploides existen tres posibilidades básicas:

| Genotipo | Significado |
|-----------|-------------|
| 0/0 | Homocigoto referencia |
| 0/1 | Heterocigoto |
| 1/1 | Homocigoto alternativo |

La determinación correcta del genotipo depende de algunos de estos mismos factores: profundidad de cobertura; calidad Phred; calidad del alineamiento; distribución de alelos; probabilidad estadística.

El resultado final del llamado de variantes corresponde al genotipo con mayor probabilidad posterior.

# Manejo de fuentes de error en el llamado de variantes 

Las principales fuentes de error durante el llamado de variantes pueden clasificarse en errores experimentales, errores computacionales y errores derivados del propio genoma de referencia. El reconocimiento y control de estos factores constituye uno de los aspectos más importantes del análisis bioinformático. 

1. Durante la síntesis y detección de nucleótidos pueden producirse errores en la identificación de las bases. Estos errores aumentan generalmente hacia los extremos de las lecturas debido a la disminución progresiva de la intensidad de fluorescencia y al incremento del ruido instrumental. Cada base recibe una puntuación de calidad Phred que representa la probabilidad de que dicha base haya sido identificada incorrectamente. Una baja calidad de secuenciación incrementa la probabilidad de detectar falsos SNPs o falsos indels. Estos errores se pueden controlar durante el trimming de lecturas de baja calidad con herramientas como Trimmomatic, fastp, o el altamente recomendado módulo clean de Captus (https://edgardomortiz.github.io/captus.docs/) y filtrando sitios con puntuación QUAL baja en el VCF final. Es posible que sea necesario cortar los extremos de las lecturas.

La escala Phred representa logarítmicamente la probabilidad de error durante la identificación de una base. Su definición matemática es:

```


Q=-10\log_{10}(P_{error})

```

donde:

- **Q** corresponde a la calidad Phred.
- **P_error** representa la probabilidad de error.

Por ejemplo:

| Calidad | Probabilidad de error |
|---------|-----------------------|
| Q20 | 1 % |
| Q30 | 0.1 % |
| Q40 | 0.01 % |

Debido a esta relación logarítmica, pequeñas diferencias en la puntuación Phred representan cambios importantes en la confiabilidad de las bases.

2. La amplificación mediante PCR puede introducir mutaciones artificiales que posteriormente son interpretadas como variantes reales. Además, la sobreamplificación puede originar múltiples copias idénticas del mismo fragmento de ADN, conocidas como duplicados de PCR. Estos duplicados generan una representación artificialmente elevada de determinados alelos y pueden sesgar la estimación del genotipo. En el módulo anterior explico como realizar el marcado de duplicados mediante Picard MarkDuplicates. 

3. Los errores de mapeo constituyen una de las principales causas de falsos positivos durante el llamado de variantes. Este problema ocurre principalmente cuando una lectura puede alinearse con alta similitud en múltiples regiones del genoma, situación frecuente en secuencias repetitivas, familias multigénicas o regiones altamente conservadas. Los alineadores asignan un valor de Mapping Quality (MAPQ) para estimar la probabilidad de que una lectura haya sido ubicada correctamente. Se filtran aplicando un umbral mínimo de MAPQ (típicamente MAPQ ≥ 20) durante el llamado de variantes, y excluyendo lecturas que mapean en múltiples posiciones del genoma (multimappers).

4. El genoma de referencia representa únicamente un individuo o ensamblaje específico de la especie y no necesariamente refleja toda la diversidad genética existente. Errores de ensamblaje, regiones faltantes, inversiones o secuencias incorrectamente ensambladas pueden provocar discrepancias sistemáticas entre las lecturas y la referencia. Los errores asociados a la referencia se mitigan usando genomas de alta calidad, y en poblaciones muy divergentes de la referencia se recomienda el uso de grafos de pangenoma (pangenome graphs) como alternativa al mapeo lineal.

5. La profundidad de secuenciación determina el número de lecturas que cubren una determinada posición del genoma. Una cobertura baja disminuye considerablemente la confianza en la estimación del genotipo debido a que un pequeño número de errores experimentales puede confundirse con variantes verdaderas. Por el contrario, coberturas excesivamente altas pueden indicar la presencia de duplicados de PCR o regiones repetitivas. Esto se puede controlar filtrando sitios por profundidad mínima (DP ≥ 10) y máxima (DP ≤ media + 2×SD) en el VCF. 

---






---




# 4.12.1 Calidad Phred

La calidad de cada nucleótido se expresa mediante la escala Phred.

Esta escala representa logarítmicamente la probabilidad de error durante la identificación de una base.

Su definición matemática es:

\[
Q=-10\log_{10}(P_{error})
\]

donde:

- **Q** corresponde a la calidad Phred.
- **P_error** representa la probabilidad de error.

Por ejemplo:

| Calidad | Probabilidad de error |
|---------|-----------------------|
| Q20 | 1 % |
| Q30 | 0.1 % |
| Q40 | 0.01 % |

Debido a esta relación logarítmica, pequeñas diferencias en la puntuación Phred representan cambios importantes en la confiabilidad de las bases.





---

# 4.12.2 Genotype Likelihood

El primer paso del llamado de variantes consiste en calcular la probabilidad de observar las lecturas obtenidas suponiendo que el individuo posee un determinado genotipo.

Esta probabilidad recibe el nombre de **Genotype Likelihood**.

Por ejemplo, para un organismo diploide se evalúan generalmente tres posibilidades:

- Homocigoto referencia (AA)
- Heterocigoto (AB)
- Homocigoto alternativo (BB)

Cada lectura aporta evidencia a favor o en contra de cada uno de estos genotipos.

La probabilidad conjunta de todas las lecturas permite estimar cuál es el genotipo más compatible con los datos experimentales.

---

# 4.12.3 Inferencia bayesiana

La inferencia bayesiana constituye el fundamento estadístico de herramientas como GATK, FreeBayes y bcftools.

El objetivo consiste en calcular la probabilidad del genotipo considerando toda la evidencia disponible.

El modelo general puede expresarse como:

\[
P(Genotipo|Datos)\propto P(Datos|Genotipo)\times P(Genotipo)
\]

donde:

- **P(Datos|Genotipo)** corresponde a la verosimilitud (Likelihood).
- **P(Genotipo)** representa la probabilidad previa (Prior).
- **P(Genotipo|Datos)** corresponde a la probabilidad posterior.

El genotipo seleccionado será aquel con mayor probabilidad posterior.

---

# 4.12.4 Priors

Los priors representan el conocimiento previo disponible antes de analizar las lecturas.

Dependiendo del algoritmo pueden construirse utilizando:

- tasa esperada de heterocigosidad;
- frecuencia conocida de SNPs;
- bases de datos como dbSNP;
- equilibrio de Hardy-Weinberg;
- información proveniente de múltiples individuos.

La utilización de priors mejora considerablemente la precisión del llamado de variantes, especialmente en regiones con baja cobertura.

---

# 4.12.5 Inferencia en múltiples muestras

Cuando varias muestras son analizadas simultáneamente, la estimación de los genotipos incorpora información poblacional.

El supuesto más utilizado corresponde al equilibrio de Hardy-Weinberg, según el cual, si la frecuencia del alelo A es **p** y la del alelo B es **q**, las frecuencias esperadas de los genotipos son:

\[
P(AA)=p^2
\]

\[
P(AB)=2pq
\]

\[
P(BB)=q^2
\]

Este enfoque incrementa la sensibilidad para detectar variantes raras presentes únicamente en algunos individuos de la población.

---

# 4.13 Genome Analysis Toolkit (GATK)

El **Genome Analysis Toolkit (GATK)** es un conjunto de herramientas bioinformáticas desarrollado por el Broad Institute para el descubrimiento y genotipado de variantes genómicas.

Desde su publicación en 2010, GATK se ha convertido en uno de los estándares internacionales para el análisis de datos de resecuenciación debido a su elevada precisión y a la incorporación de modelos estadísticos avanzados.

Actualmente es ampliamente utilizado en proyectos de secuenciación del genoma completo (WGS), exomas, paneles dirigidos y estudios de genética de poblaciones.

---

# 4.13.1 Flujo Best Practices

El flujo de trabajo recomendado por GATK se organiza en cuatro etapas principales:

1. Preprocesamiento de los alineamientos.
2. Descubrimiento de variantes.
3. Genotipado conjunto.
4. Filtrado de variantes.

Cada una de estas etapas busca reducir progresivamente la incertidumbre de los datos hasta obtener un conjunto de variantes de alta confianza.

---

# 4.14 Preprocesamiento del archivo BAM

Antes del llamado de variantes es necesario optimizar los alineamientos mediante varias etapas de procesamiento.

## Ordenamiento

Los alineamientos se ordenan según su posición cromosómica utilizando **samtools sort**.

Este procedimiento facilita el acceso eficiente a regiones específicas del genoma.

---

## Indexación

Posteriormente se genera un índice (.bai) mediante **samtools index**, permitiendo el acceso aleatorio a cualquier posición del archivo BAM.

---

## Marcado de duplicados

Durante la amplificación por PCR pueden generarse múltiples copias del mismo fragmento de ADN.

La herramienta **MarkDuplicates**, incluida en Picard, identifica estas copias utilizando la posición de inicio y orientación de las lecturas.

Los duplicados no se eliminan físicamente, sino que son marcados para que puedan ser ignorados durante el llamado de variantes.

---

## Recalibración de calidad de bases (BQSR)

Los secuenciadores presentan errores sistemáticos asociados al ciclo de secuenciación, contexto nucleotídico y química utilizada.

La herramienta **BaseRecalibrator** estima dichos errores utilizando variantes conocidas y genera un modelo estadístico que posteriormente corrige las puntuaciones Phred mediante **ApplyBQSR**.

Este procedimiento mejora considerablemente la precisión del llamado de variantes.

---

# 4.15 HaplotypeCaller

HaplotypeCaller constituye el algoritmo central del pipeline moderno de GATK.

A diferencia de los llamadores clásicos, que evalúan cada posición del genoma de forma independiente, HaplotypeCaller analiza regiones completas del genoma reconstruyendo los posibles haplotipos presentes en cada individuo.

Este enfoque permite detectar con mayor precisión regiones que contienen múltiples SNPs e inserciones o deleciones cercanas.

## 4.15.1 Regiones activas

Inicialmente, HaplotypeCaller identifica regiones del genoma donde existe evidencia de variación.

Estas regiones reciben el nombre de **Active Regions**.

Únicamente estas regiones son sometidas al análisis intensivo, reduciendo considerablemente el tiempo computacional.

---

## 4.15.2 Reensamblaje local

Cada región activa es reensamblada localmente utilizando grafos de De Bruijn.

Este procedimiento reconstruye los posibles haplotipos presentes sin depender completamente del alineamiento original.

El reensamblaje local incrementa notablemente la precisión en regiones con múltiples indels.

---

## 4.15.3 Pair Hidden Markov Model (PairHMM)

Posteriormente, cada lectura es comparada contra todos los haplotipos candidatos mediante un modelo probabilístico denominado **Pair Hidden Markov Model (PairHMM)**.

Este algoritmo calcula la probabilidad de que cada lectura provenga de cada haplotipo considerando:

- errores de secuenciación;
- inserciones;
- deleciones;
- mismatches.

Las probabilidades obtenidas sirven posteriormente para calcular la verosimilitud de cada genotipo.

---

## 4.15.4 Generación de archivos GVCF

En lugar de producir directamente un archivo VCF tradicional, HaplotypeCaller puede ejecutarse en modo **GVCF (Genomic Variant Call Format)**.

El archivo GVCF almacena información tanto de posiciones variantes como no variantes, permitiendo posteriormente combinar múltiples individuos sin necesidad de repetir el llamado de variantes.

Este enfoque constituye la base del genotipado conjunto utilizado por GATK.

# 4.16 Genotipado conjunto (Joint Genotyping)

En estudios genómicos modernos es frecuente analizar simultáneamente decenas, cientos o incluso miles de individuos. Si cada muestra fuera analizada de manera independiente, una variante presente únicamente en pocos individuos podría no ser detectada debido a la baja evidencia estadística disponible en cada muestra por separado. Para superar esta limitación, GATK implementa una estrategia denominada **Joint Genotyping** o **genotipado conjunto**, mediante la cual todas las muestras son evaluadas simultáneamente utilizando un único modelo probabilístico.

Este enfoque incrementa la sensibilidad para detectar variantes raras, mejora la consistencia de los genotipos entre individuos y permite diferenciar con mayor precisión entre posiciones verdaderamente invariantes y regiones con cobertura insuficiente.

---

## 4.16.1 Formato GVCF

El primer paso del genotipado conjunto consiste en ejecutar **HaplotypeCaller** en modo **GVCF (Genomic Variant Call Format)**.

A diferencia del formato VCF tradicional, un archivo GVCF almacena información tanto de posiciones variantes como de regiones donde no se detectaron variantes, indicando el nivel de confianza asociado a cada sitio del genoma.

Esta estrategia ofrece varias ventajas:

- evita repetir el llamado de variantes cuando se incorporan nuevas muestras;
- permite analizar cohortes de gran tamaño;
- mejora la estimación de genotipos poco frecuentes;
- facilita el procesamiento distribuido en sistemas de alto rendimiento.

Cada individuo genera inicialmente un archivo GVCF independiente que posteriormente será integrado con el resto de las muestras.

---

## 4.16.2 GenomicsDBImport y CombineGVCFs

Una vez obtenidos todos los archivos GVCF, GATK proporciona dos herramientas para consolidar la información.

### GenomicsDBImport

Es el método recomendado para estudios con un gran número de muestras. Esta herramienta organiza la información de todos los GVCF en una base de datos optimizada para consultas rápidas y procesamiento paralelo.

Sus principales ventajas son:

- menor consumo de memoria;
- mayor velocidad;
- escalabilidad para cientos o miles de individuos.

### CombineGVCFs

Esta herramienta combina varios archivos GVCF en un único archivo. Aunque resulta adecuada para proyectos pequeños, presenta limitaciones de rendimiento cuando el número de muestras aumenta considerablemente.

---

## 4.16.3 GenotypeGVCFs

Una vez consolidados los archivos GVCF, la herramienta **GenotypeGVCFs** realiza el genotipado conjunto.

Durante este procedimiento, GATK:

1. analiza simultáneamente todas las muestras;
2. estima la frecuencia de cada alelo;
3. calcula nuevamente las probabilidades de cada genotipo;
4. genera un único archivo VCF multimuesta.

El análisis conjunto permite aumentar la potencia estadística del estudio y reducir la probabilidad de clasificar incorrectamente una variante como ausencia de variación debido a una cobertura insuficiente.

---

# 4.17 Filtrado de variantes

El llamado inicial de variantes está diseñado para maximizar la sensibilidad del análisis. En consecuencia, el conjunto de variantes obtenido suele contener tanto variantes verdaderas como falsos positivos derivados de errores de secuenciación, alineamiento o preparación de bibliotecas.

Por esta razón, el filtrado constituye una etapa indispensable del análisis bioinformático.

El objetivo consiste en eliminar variantes poco confiables conservando únicamente aquellas respaldadas por suficiente evidencia experimental y estadística.

Las estrategias más utilizadas son **Hard Filtering** y **Variant Quality Score Recalibration (VQSR)**.

---

## 4.17.1 Hard Filtering

El Hard Filtering aplica umbrales fijos sobre diferentes métricas de calidad calculadas durante el llamado de variantes.

Es el método recomendado cuando:

- se dispone de pocas muestras;
- no existen bases de datos de variantes conocidas;
- se trabaja con organismos no modelo;
- no es posible entrenar modelos estadísticos complejos.

Las principales métricas utilizadas incluyen:

### Quality by Depth (QD)

Representa la calidad de la variante normalizada por la profundidad de cobertura.

Valores bajos indican que la aparente calidad de la variante depende únicamente de una cobertura muy elevada.

---

### Fisher Strand Bias (FS)

Evalúa si existe un sesgo significativo entre las lecturas provenientes de ambas hebras del ADN.

Un valor elevado puede indicar errores sistemáticos de secuenciación o alineamiento.

---

### Strand Odds Ratio (SOR)

Constituye una medida alternativa del sesgo de hebra menos sensible a diferencias extremas en la cobertura.

---

### Mapping Quality (MQ)

Corresponde a la calidad promedio del alineamiento de todas las lecturas que soportan una variante.

Valores bajos suelen indicar regiones repetitivas o alineamientos ambiguos.

---

### Mapping Quality Rank Sum Test (MQRankSum)

Compara la calidad del alineamiento entre las lecturas que contienen el alelo de referencia y aquellas que contienen el alelo alternativo.

Grandes diferencias pueden sugerir errores de mapeo.

---

### Read Position Rank Sum Test

Evalúa si el alelo alternativo aparece preferentemente en los extremos de las lecturas.

Las variantes verdaderas suelen distribuirse uniformemente a lo largo de toda la longitud de la lectura.

---

### Depth (DP)

Representa el número total de lecturas que cubren una posición determinada.

Coberturas extremadamente bajas disminuyen la confianza en el genotipo, mientras que coberturas excesivamente altas pueden indicar duplicados o regiones repetitivas.

---

## 4.17.2 Variant Quality Score Recalibration (VQSR)

Cuando se dispone de grandes cohortes y bases de datos de variantes conocidas, GATK recomienda utilizar **Variant Quality Score Recalibration (VQSR)**.

Este procedimiento utiliza modelos de aprendizaje automático basados en mezclas de gaussianas para aprender las características de variantes verdaderas a partir de conjuntos de entrenamiento como dbSNP, HapMap o 1000 Genomes.

Posteriormente, cada variante recibe una puntuación que representa la probabilidad de corresponder a una variante real.

Las principales ventajas de VQSR son:

- mayor sensibilidad;
- menor tasa de falsos positivos;
- filtrado adaptativo;
- mejor desempeño en estudios poblacionales.

No obstante, este método requiere un gran número de variantes conocidas y un tamaño de muestra considerable, por lo que generalmente no resulta adecuado para organismos no modelo.

---

# 4.18 Formato Variant Call Format (VCF)

Una vez finalizado el llamado y filtrado de variantes, los resultados se almacenan en archivos **VCF (Variant Call Format)**, considerados el estándar internacional para representar variantes genómicas.

El formato VCF fue desarrollado para facilitar el intercambio de información entre diferentes herramientas bioinformáticas y contiene tanto información de cada variante como los genotipos de todos los individuos analizados.

---

## 4.18.1 Estructura general

Un archivo VCF está compuesto por dos secciones principales.

### Cabecera

Incluye:

- versión del formato;
- programa utilizado;
- filtros aplicados;
- definición de todos los campos INFO y FORMAT;
- información del genoma de referencia.

---

### Registros

Cada línea representa una variante individual.

Los campos obligatorios son:

| Campo | Descripción |
|--------|-------------|
| CHROM | Cromosoma |
| POS | Posición |
| ID | Identificador |
| REF | Alelo de referencia |
| ALT | Alelo alternativo |
| QUAL | Calidad |
| FILTER | Estado del filtrado |
| INFO | Información adicional |
| FORMAT | Campos del genotipo |
| SAMPLE | Información por individuo |

---

## 4.18.2 Campo INFO

El campo INFO almacena información global de la variante.

Algunos parámetros frecuentes son:

- AC: número de alelos alternativos;
- AF: frecuencia del alelo alternativo;
- AN: número total de alelos;
- DP: profundidad total;
- MQ: calidad promedio del alineamiento;
- QD: calidad normalizada por profundidad.

---

## 4.18.3 Campo FORMAT

Describe la información reportada para cada individuo.

Entre los campos más importantes se encuentran:

### GT

Genotipo.

Ejemplos:

```
0/0
```

Homocigoto referencia.

```
0/1
```

Heterocigoto.

```
1/1
```

Homocigoto alternativo.

---

### AD

Número de lecturas que soportan cada alelo.

Ejemplo:

```
18,17
```

18 lecturas apoyan el alelo de referencia y 17 el alternativo.

---

### DP

Profundidad total de cobertura para ese individuo.

---

### GQ

Genotype Quality.

Representa la confianza estadística del genotipo asignado.

---

### PL

Likelihoods del genotipo expresadas en escala Phred.

Estos valores permiten comparar probabilísticamente los diferentes genotipos posibles.

---

# 4.19 Herramientas alternativas para el llamado de variantes

Aunque GATK constituye uno de los estándares actuales, existen otros llamadores ampliamente utilizados dependiendo del tipo de datos y los objetivos del estudio.

## bcftools

bcftools utiliza modelos probabilísticos derivados de SAMtools y destaca por:

- rapidez;
- bajo consumo de memoria;
- facilidad de uso;
- excelente integración con SAMtools.

Es ampliamente utilizado para proyectos pequeños y análisis exploratorios.

---

## FreeBayes

FreeBayes emplea un modelo basado en haplotipos que permite detectar variantes complejas y analizar organismos con diferentes niveles de ploidía.

Resulta particularmente útil en estudios de genética de poblaciones y organismos no modelo.

---

## DeepVariant

DeepVariant, desarrollado por Google, incorpora técnicas de aprendizaje profundo para el llamado de variantes.

El algoritmo transforma los alineamientos en representaciones similares a imágenes y utiliza redes neuronales convolucionales para clasificar variantes.

Diversos estudios han demostrado que DeepVariant alcanza precisiones comparables o superiores a los métodos tradicionales, especialmente en regiones genómicas complejas.

---

## Comparación general

| Herramienta | Modelo | Ventajas | Limitaciones |
|-------------|---------|----------|--------------|
| GATK | Bayesiano + PairHMM | Muy alta precisión | Mayor demanda computacional |
| bcftools | Bayesiano | Muy rápido | Menor sensibilidad en variantes complejas |
| FreeBayes | Haplotipos | Flexible para distintas ploidías | Más lento en cohortes grandes |
| DeepVariant | Aprendizaje profundo | Muy alta precisión | Requiere GPU y mayor capacidad computacional |

---


